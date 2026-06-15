<!--
  Copyright © 2020-2026 Airbus DS SLC. All rights reserved.
-->
<template>
    <b-card bg-variant="light" class="mb-4" title="Clear data">
        <div class="mt-2 d-flex column gap-3">
            <b-overlay :show="deleting" rounded="sm">
                <b-button block variant="primary" name="clean-conv" :disabled="convNumber <= 0" @click="() => deleteConversations(conversations)">{{ convNumber > 0 ? `Delete ${convNumber} conversation(s)` : "No conversation to delete" }}</b-button>
            </b-overlay>
            <b-overlay :show="hangingUpIncoming" rounded="sm">
                <b-button block variant="primary" name="clean-calls" :disabled="callNumber <= 0" @click="() => hangUpCalls(calls)">{{ callNumber > 0 ? `Hang up ${callNumber} incoming call(s)` : "No incoming call to hang up" }}</b-button>
            </b-overlay>
            <b-overlay :show="hangingUpOutgoing" rounded="sm">
                <b-button block variant="primary" name="clean-out-call" :disabled="outgoingCall === null" @click="hangUpOutgoingCall">{{ outgoingCall !== null ? `Hang up outgoing call` : "No outgoing call to hang up" }}</b-button>
            </b-overlay>
        </div>
    </b-card>
    <b-card bg-variant="light" class="mb-4" title="Current user configuration">
        <div class="my-3 d-flex column gap-3">
            <b-form-group label="Test Mode" class="mx-4 d-flex justify-content-start align-items-center gap-5">
                <label>Automatized configurations :</label>
                <b-form-radio v-model="testMode" name="caller-test-mode" value="caller">Caller</b-form-radio>
                <b-form-radio v-model="testMode" name="callee-test-mode" value="callee">Callee</b-form-radio>
                <label>Manual configuration :</label>
                <b-form-radio v-model="testMode" name="manual-test-mode" value="manual">Manual</b-form-radio>
            </b-form-group>
        </div>
        <span>
            <strong>Notes :</strong>
            <br />
            - To start automated tests, be sure to use Caller/Callee configuration for each participant. Each selected test case will be performed one after the other.
            <br />
            - Data will be automatically collected on each participant side after automatic steps : call acceptation, hangup and conversation deletion.
            <br />
            - To deactivate all automatizations, use the Manual configuration instead of the Caller mode.
            <br />
            - To start a call on Manual configuration, you have to select one and only one test case.
        </span>
    </b-card>
    <div class="container-fluid mb-3 d-flex gap-4 align-items-center">
        <b-form-checkbox v-model="inspect" switch>Inspect conversations</b-form-checkbox>
        <b-form-checkbox v-model="logFlow" switch>Log Flow events</b-form-checkbox>
        <b-form-checkbox v-model="logSocket" switch>Log SIO events</b-form-checkbox>
        <b-form-checkbox v-model="logBus" switch>Log BUS events</b-form-checkbox>
        <div class="form-check d-flex align-items-center gap-2 p-0">
            <b-form-select id="logLevelSelect" v-model="selectedLogLevel" :options="logLevels" size="sm" style="width: min-content; height: min-content" />
            <label for="logLevelSelect" class="form-check-label">Logger level</label>
        </div>
    </div>
    <b-card v-if="conversationsData.length > 0" bg-variant="light" title="Conversations data">
        <div class="d-flex justify-content-end align-items-center mb-3 gap-3">
            <b-button size="sm" variant="outline-primary" @click="() => exportJson(conversationsData)">Export All</b-button>
        </div>
        <div v-for="(result, index) in conversationsData" :key="index" class="mb-3">
            <b-card>
                <b-card-body>
                    <pre>{{ JSON.stringify(result, null, 2) }}</pre>
                </b-card-body>
                <b-card-footer class="d-flex justify-content-start align-items-center gap-2">
                    <b-button size="sm" variant="outline-primary" @click="() => exportJson(result)">Export</b-button>
                </b-card-footer>
            </b-card>
        </div>
    </b-card>
</template>

<script setup>
import {ref, computed, watch, onMounted, onBeforeUnmount} from "vue";
import {Socket} from "@webchat/sdk";
import {dispatcher} from "@dispatcher/global";
import {hangingUpIncoming, hangUpCalls, outgoingCall, hangingUpOutgoing, hangUpOutgoingCall} from "../composables/calls.js";
import {deleting, deleteConversations} from "../composables/conversations.js";
import {testMode, selectedLogLevel, logLevels} from "../composables/testConfiguration.js";
import {inspectConversations} from "../composables/testResults.js";
import {exportJson} from "../utils/exportUtils.js";

/**
 * @type {Set<import("@dispatcher/conversations-flow/conversations.js").ConversationWrapper>}
 * Current user conversations
 */
const conversations = dispatcher.conversations.flow.root.emits;

/**
 * @type {import("vue").ComputedRef<number>}
 * Current user conversations number
 */
const convNumber = computed(() => conversations.size);

/**
 * Current user active calls
 */
const allCalls = dispatcher.calls.flow.root.emits;

/**
 * Current user active calls, excluding talkgroups
 */
const calls = computed(() => {
    return Array.from(allCalls).filter((c) => c.type !== "CHANNEL");
});

/**
 * Current user active calls (except for talkgroups) number
 */
const callNumber = computed(() => calls.value.length);

/**
 * Boolean to store either to show conversations details or not
 */
const inspect = ref(false);

/**
 * Current conversations details to show
 */
const conversationsData = computed(() => (inspect.value ? inspectConversations(Array.from(conversations)) : [])); // TODO : Check that it's computed again when intern data of a conversation changes

/**
 * Either to log SIO events or not.
 */
const logFlow = ref(false);

const convFlowLogger = (e) => console.warn("CONVERSATIONS flow -----------------", e);

const msgFlowLogger = (e) => console.error("MESSAGES flow  -----------------", e);

/**
 * Start or stop to listen to SIO events according to new value of logSocket boolean
 */
watch(logFlow, (newValue) => {
    if (newValue) {
        console.log("Activating Flow create event logs");
        dispatcher.conversations.flow.root.on.create(convFlowLogger);
        dispatcher.conversations.messages.flow.root.on.create(msgFlowLogger);
        dispatcher.conversations.flow.root.on.remove(convFlowLogger);
        dispatcher.conversations.messages.flow.root.on.remove(msgFlowLogger);
    } else {
        console.log("Deactivating Flow create event logs");
        dispatcher.conversations.flow.root.off.create(convFlowLogger);
        dispatcher.conversations.messages.flow.root.off.create(msgFlowLogger);
        dispatcher.conversations.flow.root.off.remove(convFlowLogger);
        dispatcher.conversations.messages.flow.root.off.remove(msgFlowLogger);
    }
});

/**
 * Either to log SIO events or not.
 */
const logSocket = ref(false);

/**
 * SIO events to never log.
 */
const SOCKET_EVENT_EXCLUDES = new Set(["CONVERSATIONS_UNREAD_REQUEST", "CONVERSATIONS_UNREAD_RESPONSE", "MY_RULES_PACKAGE_LIST_REQUEST", "MY_RULES_PACKAGE_LIST"]);

/**
 * Log incoming event as a console warn if its name is not among the log excluded events.
 * @param {string} name - The name of the socket event
 * @param {Object} event - The event data
 */
const handlerIncoming = (name, event) => {
    if (logSocket.value && !SOCKET_EVENT_EXCLUDES.has(name)) {
        console.warn("EVENT < == ", name, event);
    }
};

/**
 * Log outgoing event as a console warn if its name is not among the log excluded events.
 * @param {string} name - The name of the socket event
 * @param {Object} event - The event data
 */
const handlerOutgoing = (name, event) => {
    if (logSocket.value && !SOCKET_EVENT_EXCLUDES.has(name)) {
        console.warn("EVENT == >", name, event);
    }
};

/**
 * Start or stop to listen to SIO events according to new value of logSocket boolean
 */
watch(logSocket, (newValue) => {
    if (newValue) {
        console.log("Activating SIO event logs");
        Socket.socket.onAny(handlerIncoming);
        Socket.socket.onAnyOutgoing(handlerOutgoing);
    } else {
        console.log("Deactivating SIO event logs");
        Socket.socket.offAny(handlerIncoming);
        Socket.socket.offAnyOutgoing(handlerOutgoing);
    }
});

const logBus = ref(false);

const busLogger = (e) => console.error("BUS -----------------", e);

/**
 * Start or stop to listen to SIO events according to new value of logSocket boolean
 */
watch(logBus, (newValue) => {
    if (newValue) {
        console.log("Activating BUS event logs");
        dispatcher.bus.on(busLogger);
    } else {
        console.log("Deactivating BUS event logs");
        dispatcher.bus.off(busLogger);
    }
});

onMounted(() => {
    if (logFlow.value) {
        dispatcher.conversations.flow.root.on.create(convFlowLogger);
        dispatcher.conversations.messages.flow.root.on.create(msgFlowLogger);
    }
    if (logSocket.value) {
        Socket.socket.onAny(handlerIncoming);
        Socket.socket.onAnyOutgoing(handlerOutgoing);
    }
    if (logBus.value) {
        dispatcher.bus.on(busLogger);
    }
});

onBeforeUnmount(() => {
    if (logFlow.value) {
        dispatcher.conversations.flow.root.off.create(convFlowLogger);
        dispatcher.conversations.messages.flow.root.off.create(msgFlowLogger);
    }
    if (logSocket.value) {
        Socket.socket.offAny(handlerIncoming);
        Socket.socket.offAnyOutgoing(handlerOutgoing);
    }
    if (logBus.value) {
        dispatcher.bus.off(busLogger);
    }
});
</script>
