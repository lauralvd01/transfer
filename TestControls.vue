<!--
  Copyright © 2020-2026 Airbus DS SLC. All rights reserved.
-->
<template>
    <b-card bg-variant="light" class="mb-3" title="Test Status">
        <b-form-group label="Test Status" class="mx-4 d-flex align-items-center">
            <b-badge v-if="testStatus === 'idle'" variant="success">Idle</b-badge>
            <b-badge v-if="testStatus === 'running'" variant="info">Running</b-badge>
            <b-badge v-if="testStatus === 'waiting'" variant="warning">Waiting</b-badge>
            <b-badge v-if="testStatus === 'error'" variant="danger">Error</b-badge>
        </b-form-group>
        <div class="my-2 d-flex justify-content-around align-items-center">
            <b-overlay :show="calling" rounded="sm">
                <b-button :disabled="!canStartManually" class="mt-3 px-5" variant="primary" @click="startManualTest">Start manual test</b-button>
            </b-overlay>
            <b-overlay :show="testStatus === 'running'" rounded="sm">
                <b-button :disabled="testMode !== 'caller'" class="mt-3 px-5" variant="primary" @click="launchAutomatedTests">Start automated tests</b-button>
            </b-overlay>
        </div>
    </b-card>
</template>

<script>
export default {
    name: "TestControls"
};
</script>

<script setup>
import {computed, onMounted, onBeforeUnmount} from "vue";
import {dispatcher} from "@dispatcher/global";
import {createEventSubscriberController, sourceFilter, syncPointFilter} from "../composables/events.js";
import {testMode, canStartManually} from "../composables/testConfiguration.js";
import {calling, testStatus, startManualTest, startAutomatedTest} from "../composables/testExecutionCaller.js";
import {handleIncomingTestCase} from "../composables/testExecutionCallee.js";

/**
 * @type {Set<import("@dispatcher/conversations-flow/conversations.js").ConversationWrapper>}
 * Current user conversations
 */
const conversations = dispatcher.conversations.flow.root.emits;

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

let /** @type {import("../composables/events.js").EventSubscriberController} */ eventSubscriberController;

const launchAutomatedTests = () => {
    startAutomatedTest(conversations, eventSubscriberController);
};

const handleCalleeTestStart = async (/** @type {import("../composables/events.js").ChronologyEvent[]} */ chronology) => {
    if (testMode.value !== "callee" || testStatus.value !== "idle" || !chronology.length) {
        return;
    }

    const lastEvent = chronology.at(-1);

    if (sourceFilter("test:broadcast")(lastEvent) && syncPointFilter("testStart")(lastEvent)) {
        const currentUserId = dispatcher.resources.currentUser.value?.data?._msisdn;
        const /** @type {import("../composables/events.js").BroadcastEvent} */ event = lastEvent;
        if (event.detail.expectedParticipants.includes(currentUserId)) {
            handleIncomingTestCase(event, calls, conversations, eventSubscriberController);
        }
    }
};

onMounted(() => {
    eventSubscriberController = createEventSubscriberController();
    eventSubscriberController.start();
    eventSubscriberController.on(handleCalleeTestStart, true);
});

onBeforeUnmount(() => {
    eventSubscriberController.stop();
    eventSubscriberController.reset();
});
</script>
