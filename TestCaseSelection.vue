<!--
  Copyright © 2020-2026 Airbus DS SLC. All rights reserved.
-->
<template>
    <div class="d-flex justify-content-end align-items-center mb-3 gap-3">
        <input ref="importConfig" type="file" accept=".json" class="d-none" @change="importTestConfiguration($event)" />
        <b-button size="sm" variant="primary" @click="() => importConfig.click()">Import Test Configuration</b-button>
        <b-button size="sm" variant="primary" @click="exportTestConfiguration(selectedTestCases)">Export Test
            Configuration</b-button>
        <b-button size="sm" variant="outline-primary" @click="resetSelectedTestCases">Clear Configuration</b-button>
    </div>
    <b-card bg-variant="light" class="mb-3" title="Scenarios">
        <div class="d-flex justify-content-end">
            <b-button @click="selectAllScenarios">
                {{ selectedTestCases.scenarios.every((s) => s.selected) ? "Unselect all" : "Select all" }}
            </b-button>
        </div>
        <div class="container-fluid mt-2 d-flex row gap-2">
            <span class="mb-2"><strong>Select scenario(s) to test :</strong></span>
            <div class="d-flex flex-wrap gap-3 justify-content-around align-items-center">
                <div v-for="(scenario, scenarioIndex) of selectedTestCases.scenarios" :key="scenarioIndex">
                    <b-form-checkbox v-model="scenario.selected">{{ scenario.description }}</b-form-checkbox>
                </div>
            </div>
        </div>
    </b-card>
    <b-card bg-variant="light" class="mb-3" title="Calls">
        <div class="d-flex justify-content-end">
            <b-button @click="selectAllCalls">
                {{ selectedTestCases.callTypes.every((c) => c.selected) ? "Unselect all" : "Select all" }}
            </b-button>
        </div>
        <div class="container-fluid mt-2 d-flex row gap-2">
            <span class="mb-2"><strong>Select call type(s) under test :</strong></span>
            <div class="d-flex flex-wrap gap-3 justify-content-around align-items-center">
                <div v-for="(call, callIndex) of selectedTestCases.callTypes" :key="callIndex">
                    <b-form-checkbox v-model="call.selected">{{ call.name }} call</b-form-checkbox>
                </div>
            </div>
        </div>
    </b-card>
    <b-card bg-variant="light" class="mb-3" title="Recipients">
        <div class="d-flex justify-content-between alig-items-center my-1">
            <div class="d-flex gap-3">
                <div v-for="(contact, index) of recipients" :key="index">
                    <b-badge :class="contact._type !== 'group' ? 'common-chip-user' : 'common-chip-group'"
                        class="common-chip" pill>
                        <span class="contact-identifier text-truncate">{{ contact._type === "group" ? contact._name :
                            `${contact._firstName} ${contact._lastName}` }}</span>
                        <icon
                            :class="contact._type !== 'group' ? 'common-chip-user__remove' : 'common-chip-group__remove'"
                            name="cross" role="button" @click="() => recipients.splice(index, 1)" />
                    </b-badge>
                </div>
            </div>
            <div class="d-flex justify-content-end gap-3">
                <b-button @click="selectRecipients">Select contacts</b-button>
                <b-button block variant="primary" :disabled="recipients.length === 0"
                    @click="() => selectedTestCases.recipientSets.push({selected: true, recipients, alias: `P_${selectedTestCases.recipientSets.length + 1}`})">Save
                    as recipients preset</b-button>
            </div>
        </div>
        <br />
        <div class="container-fluid mt-2 d-flex row gap-2">
            <span class="mb-2"><strong>Select preset(s) of recipients to call :</strong></span>
            <div v-for="(set, setIndex) of selectedTestCases.recipientSets" :key="setIndex"
                class="d-flex align-items-center gap-5">
                <div class="d-flex flex-wrap gap-3 justify-content-start align-items-center my-2">
                    <b-form-checkbox v-model="set.selected" class="mt-1" />
                    <div v-for="(contact, index) of set.recipients" :key="index" role="button" class="mt-2"
                        @click="() => (set.selected = !set.selected)">
                        <b-badge :class="contact._type !== 'group' ? 'common-chip-user' : 'common-chip-group'"
                            class="common-chip" style="height: 2rem !important" pill>
                            <span class="contact-identifier text-truncate">{{ contact.fullName }}</span>
                        </b-badge>
                    </div>
                    <icon :class="'common-chip-user__remove'" name="cross" role="button"
                        @click="() => selectedTestCases.recipientSets.splice(setIndex, 1)" />
                </div>
                <div class="d-flex align-items-center gap-3">
                    <label>Preset alias :</label>
                    <b-form-input v-model="set.alias" class="alias-input" placeholder=""
                        :state="set.alias ? null : false" size="sm" style="width: 100px"
                        @focus="set.alias ? null : (set.alias = `P_${setIndex + 1}`)" />
                </div>
            </div>
        </div>
    </b-card>
</template>

<script>
    export default {
        name: "TestCaseSelection"
    };
</script>

<script setup>
    import { useTemplateRef } from "vue";
    import { recipients, selectedTestCases, resetSelectedTestCases, importTestCases, selectRecipients, selectAllCalls, selectAllScenarios } from "../composables/testConfiguration.js";
    import { exportJson } from "../utils/exportUtils.js";

    /**
     * Export test configuration (selected presets, call types and scenarios) as JSON
     * @param {import("../composables/testConfiguration.js").TestCases} testCases - The selected test cases
     */
    const exportTestConfiguration = (testCases) => {
        const config = {
            exportTimestamp: new Date().toISOString(),
            selectedScenarios: testCases.scenarios.filter((s) => s.selected).map((s) => ({ ...s, selected: undefined })),
            selectedCallTypes: testCases.callTypes.filter((c) => c.selected).map((c) => `${c.name} Call`),
            selectedRecipientPresets: testCases.recipientSets
                .filter((p) => p.selected)
                .map((p) => ({
                    alias: p.alias || null,
                    recipients: p.recipients.map((r) => ({
                        id: r.type === "user" ? r.msisdn : r.groupId,
                        type: r.type,
                        fullName: r.name
                    }))
                }))
        };

        exportJson(config, "test-configuration");
    };

    /**
     * Template ref for the import file input - Side A
     * @type {import('vue').Ref<HTMLInputElement>}
     */
    const importConfig = useTemplateRef("importConfig");

    /**
     * Import test configuration (selected presets, call types and scenarios) from a JSON saved with above function
     * @param {Event} evt - File input change event
     * @returns {Promise<void>}
     * @throws {Error} If file parsing fails
     */
    const importTestConfiguration = async (evt) => {
        try {
            const { files } = evt.target;
            if (!files.length) {
                return;
            }
            const file = files[0];
            const isJson = file.name.endsWith(".json");
            if (!isJson) {
                throw new Error("Input is not a JSON file");
            }
            const text = await file.text();
            const { selectedScenarios, selectedCallTypes, selectedRecipientPresets } = JSON.parse(text);

            importTestCases(selectedScenarios, selectedCallTypes, selectedRecipientPresets);
        } catch (error) {
            console.error("Failed to process JSON file", error);
        }
    };
</script>