<!--
  Copyright © 2020-2026 Airbus DS SLC. All rights reserved.
-->
<template>
    <b-card bg-variant="light" title="Individual Test Results">
        <div class="d-flex justify-content-end align-items-center mb-3 gap-3">
            <span v-if="testResults.length === 0" class="text-muted">No test results available</span>
            <b-button v-if="testResults.length > 0" size="sm" variant="outline-primary"
                @click="() => exportTestResults(testResults)">Export All Results</b-button>
            <b-button v-if="testResults.length > 0" size="sm" variant="outline-primary"
                @click="() => (testResults = [])">Delete All Results</b-button>
        </div>
        <b-accordion>
            <b-accordion-item v-for="(result, index) in testResults" :key="index" class="mb-3">
                <template #title>
                    {{
                    new Date(result.timestamp).toLocaleString("en-US", {
                    year: "numeric",
                    month: "short",
                    day: "numeric",
                    hour: "2-digit",
                    minute: "2-digit",
                    second: "2-digit",
                    hour12: false
                    })
                    }}
                    --- Scenario {{ result.scenario }} - {{ result.callType }} call - Preset : {{ result.preset }}
                </template>
                <template #default>
                    <pre>{{ JSON.stringify(result, null, 2) }}</pre>
                    <div class="card-footer d-flex justify-content-start align-items-center gap-2">
                        <b-button size="sm" variant="outline-primary"
                            @click="() => testResults.splice(index, 1)">Delete</b-button>
                        <b-button size="sm" variant="outline-primary"
                            @click="() => exportJson(result)">Export</b-button>
                    </div>
                </template>
            </b-accordion-item>
        </b-accordion>
    </b-card>
</template>

<script>
    export default {
        name: "TestResults"
    };
</script>

<script setup>
    import { testResults } from "../composables/testResults.js";
    import { exportJson, exportTestResults } from "../utils/exportUtils.js";
</script>