<!--
  Copyright © 2020-2026 Airbus DS SLC. All rights reserved.
-->
<template>
    <b-accordion free>
        <div class="container-fluid mb-3 d-flex gap-4 justify-content-end align-items-center">
            <b-button size="sm" variant="outline-primary" @click="() => importResultsA.click()">Import Merged
                Results</b-button>
            <b-button :disabled="mergedResults.length === 0" size="sm" variant="outline-primary"
                @click="() => exportJson(mergedResults, 'merged_results')">Save Merged Results</b-button>
        </div>
        <b-accordion-item class="accordion-item" title="Merged results">
            <div class="d-flex justify-content-end align-items-center mb-3 gap-3">
                <b-button :disabled="mergedData.length === 0" size="sm" variant="outline-primary"
                    @click="() => exportMergedResultsTable('csv')">Export Merged Results > CSV</b-button>
                <b-button :disabled="mergedData.length === 0" size="sm" variant="outline-primary"
                    @click="() => exportMergedResultsTable('md')">Export Merged Results > Markdown</b-button>
            </div>
            <b-table hover striped responsive :items="mergedData" :fields="fields"
                style="--bs-table-font-size: 0.875rem; --bs-table-cell-padding-y: 0.25rem; --bs-table-cell-padding-x: 0.2rem">
                <template #head(Test_id)="scope">
                    <div style="width: 20px; min-width: 20px; max-width: 20px; text-align: center">{{ scope.label }}
                    </div>
                </template>

                <template #cell(Test_id)="row">
                    {{ row.item.testId }}
                </template>

                <template #cell(Test_case)="row">
                    <div style="display: flex; width: 100%; gap: 4px">
                        <div style="flex: 1; padding: 4px; border: 0.2px solid #ddd; text-align: center">S_{{
                            row.item.scenario }}</div>
                        <div style="flex: 1; padding: 4px; border: 0.2px solid #ddd; text-align: center">{{
                            row.item.callType }} call</div>
                        <div style="flex: 1; padding: 4px; border: 0.2px solid #ddd; text-align: center">
                            {{ row.item.preset }}
                        </div>
                    </div>
                </template>

                <template #cell(Expected_participants)="row">
                    <span v-if="row.item.expectedParticipants.length > 0">
                        {{ row.item.expectedParticipants.join(", ") }}
                    </span>
                    <span v-else class="text-danger">None</span>
                </template>

                <template #cell(Missed_participants)="row">
                    <span v-if="row.item.missedParticipants.length > 0" class="text-danger">
                        {{ row.item.missedParticipants.join(", ") }}
                    </span>
                    <span v-else class="text-muted">None</span>
                </template>

                <template #cell(Caller_conversation_name)="row">
                    <div v-if="row.item.callerConversationName !== 'None'">"{{ row.item.callerConversationName }}"</div>
                    <span v-else class="text-danger">None</span>
                </template>

                <template #cell(Caller_conversation_name_after_load)="row">
                    <div v-if="row.item.callerConvNameAfterLoad !== 'None'">"{{ row.item.callerConvNameAfterLoad }}"
                    </div>
                    <span v-else class="text-danger">None</span>
                </template>

                <template #cell(Callees_conversation_names)="row">
                    <div v-if="Object.entries(row.item.calleesConversationNames).length > 0">
                        {{
                        Object.entries(row.item.calleesConversationNames)
                        .map(([name, participants]) => `"${name}" for callee${participants.length > 1 ? "s" : ""}
                        ${participants.join(", ")}`)
                        .join("; ")
                        }}
                    </div>
                    <span v-else class="text-danger">None</span>
                </template>

                <template #cell(Callees_conversation_names_after_load)="row">
                    <div
                        v-if="row.item.calleesConvNamesAfterLoad && Object.entries(row.item.calleesConvNamesAfterLoad).length > 0">
                        {{
                        Object.entries(row.item.calleesConvNamesAfterLoad)
                        .map(([name, participants]) => `"${name}" for callee${participants.length > 1 ? "s" : ""}
                        ${participants.join(", ")}`)
                        .join("; ")
                        }}
                    </div>
                    <span v-else class="text-danger">None</span>
                </template>

                <template #cell(Errors)="row">
                    <div v-for="(error, index) in row.item.errors" :key="index">
                        <strong>{{ getIssueString(error).title }}</strong>
                        {{ getIssueString(error).detail }}
                    </div>
                </template>
            </b-table>
        </b-accordion-item>
        <b-accordion-item class="accordion-item" title="Issues inventory">
            <div class="d-flex justify-content-end align-items-center mb-3 gap-3">
                <b-button :disabled="completeIssuesLists.length === 0" size="sm" variant="outline-primary"
                    @click="() => exportTestCasesTable('csv')">Export Test cases > CSV</b-button>
                <b-button :disabled="completeIssuesLists.length === 0" size="sm" variant="outline-primary"
                    @click="() => exportTestCasesTable('md')">Export Test cases > Markdown</b-button>
                <b-button :disabled="completeIssuesLists.length === 0" size="sm" variant="outline-primary"
                    @click="() => exportIssuesTable('csv')">Export All Issues > CSV</b-button>
                <b-button :disabled="completeIssuesLists.length === 0" size="sm" variant="outline-primary"
                    @click="() => exportIssuesTable('md')">Export All Issues > Markdown</b-button>
                <b-button :disabled="completeIssuesLists.length === 0" size="sm" variant="outline-primary"
                    @click="() => exportAllResults()">Export Full Test Data</b-button>
            </div>
            <b-table hover striped responsive :items="completeIssuesLists" :fields="issuesFields"
                style="--bs-table-font-size: 0.875rem; --bs-table-cell-padding-y: 0.25rem; --bs-table-cell-padding-x: 0.2rem">
                <template #head(Test_id)="scope">
                    <div style="width: 20px; min-width: 20px; max-width: 20px; text-align: center">{{ scope.label }}
                    </div>
                </template>
                <template #head(Test_case_severity)="scope">
                    <div style="width: 20px; min-width: 20px; max-width: 20px; text-align: center">{{ scope.label }}
                    </div>
                </template>
                <template #head(Issues_count)="scope">
                    <div style="width: 20px; min-width: 20px; max-width: 20px; text-align: center">{{ scope.label }}
                    </div>
                </template>

                <template #cell(Test_id)="row">
                    {{ row.item.testId }}
                </template>

                <template #cell(Test_case)="row">
                    <div style="display: flex; width: 100%; gap: 4px">
                        <div style="flex: 1; padding: 4px; border: 0.2px solid #ddd; text-align: center">S_{{
                            row.item.scenario }}</div>
                        <div style="flex: 1; padding: 4px; border: 0.2px solid #ddd; text-align: center">{{
                            row.item.callType }} call</div>
                        <div style="flex: 1; padding: 4px; border: 0.2px solid #ddd; text-align: center">
                            {{ row.item.preset }}
                        </div>
                    </div>
                </template>

                <template #cell(Test_case_severity)="row">
                    {{ row.item.testCaseSeverity }}
                </template>

                <template #cell(Issues_count)="row">
                    {{ row.item.issuesCount }}
                </template>

                <template #cell(Issues)="row">
                    <div v-for="(issue, index) in row.item.issues" :key="index">
                        <strong>{{ getIssueString(issue).title }}</strong>
                        {{ getIssueString(issue).detail }}
                    </div>
                </template>

                <template #cell(Export)="row">
                    <div class="d-flex justify-content-center">
                        <b-button size="sm" variant="outline-primary"
                            @click="() => exportTestCaseDocument(row.item)">Export Test Case document</b-button>
                    </div>
                </template>
            </b-table>
        </b-accordion-item>
        <b-accordion-item class="accordion-item" title="Compare With A Second Test Run" visible>
            <div class="d-flex justify-content-start align-items-center mb-4 gap-4">
                Current test run (merged results) :
                <input ref="importResultsA" type="file" accept=".json" @change="handleResultsImport($event, 'A')" />
            </div>
            <div class="d-flex justify-content-start align-items-center mb-4 gap-4">
                Second test run (merged results) :
                <input ref="importResultsB" type="file" accept=".json" @change="handleResultsImport($event, 'B')" />
            </div>

            <h5 v-if="comparedTestCases && comparedTestCases.length > 0">Comparison Summary</h5>
            <div v-if="comparedTestCases && comparedTestCases.length > 0"
                class="d-flex justify-content-around align-items-center mt-2 mb-4 gap-4">
                <p>Test cases compared : {{ comparedTestCases.length }}</p>
                <p>Identical test cases : {{ comparedTestCases.filter((t) => t.identicalTests).length }}</p>
                <p>Test cases with differences : {{ comparedTestCases.filter((t) => !t.identicalTests).length }}</p>
                <p>Missing test cases (only in A) : {{ mergedData.length - comparedTestCases.length }}</p>
                <p>New test cases (only in B): {{ secondMergedResultsData.length - comparedTestCases.length }}</p>
            </div>

            <h5 v-if="comparedTestCases && comparedTestCases.length > 0">Test cases with differences</h5>
            <b-table v-if="comparedTestCases && comparedTestCases.length > 0" hover striped responsive
                :items="comparedTestCases.filter((t) => !t.identicalTests)" :fields="comparisonFields"
                style="--bs-table-font-size: 0.875rem; --bs-table-cell-padding-y: 0.25rem; --bs-table-cell-padding-x: 0.2rem">
                <template #cell(Test_case)="row">
                    {{ row.item.testCase }}
                </template>

                <template #cell(Different_issues_count)="row">
                    {{ row.item.differentIssues.length }}
                </template>

                <template #cell(Issues-A)="row">
                    <div v-for="(issue, index) in row.item.differentIssues.filter((i) => i.resultSide === 'A')"
                        :key="index">
                        <strong>{{ getIssueString(issue).title }}</strong>
                        {{ getIssueString(issue).detail }}
                    </div>
                </template>

                <template #cell(Issues-B)="row">
                    <div v-for="(issue, index) in row.item.differentIssues.filter((i) => i.resultSide === 'B')"
                        :key="index">
                        <strong>{{ getIssueString(issue).title }}</strong>
                        {{ getIssueString(issue).detail }}
                    </div>
                </template>
            </b-table>
        </b-accordion-item>
    </b-accordion>
</template>

<script setup>
    import { computed, ref, useTemplateRef } from "vue";
    import { mergedResults } from "../composables/testResults.js";
    import { mergeData, getTestCaseIssuesList } from "../utils/mergeResults.js";
    import { exportJson, exportTableIntoCSV, exportTableIntoMarkdown, exportAllResultsAsZip } from "../utils/exportUtils.js";

    // Fields for merged results table
    const fields = [
        {
            key: "Test_id",
            label: "Test id",
            sortable: true
        },
        {
            key: "Test_case",
            label: "Test case",
            sortable: true
        },
        {
            key: "Expected_participants",
            label: "Expected participants",
            sortable: true
        },
        {
            key: "Missed_participants",
            label: "Missed participants",
            sortable: true
        },
        {
            key: "Caller_conversation_name",
            label: "Caller conversation name",
            sortable: true
        },
        {
            key: "Caller_conversation_name_after_load",
            label: "Caller conversation name after load",
            sortable: true
        },
        {
            key: "Callees_conversation_names",
            label: "Callees conversation names",
            sortable: true
        },
        {
            key: "Callees_conversation_names_after_load",
            label: "Callees conversation names after load",
            sortable: true
        },
        {
            key: "Errors",
            label: "Errors : User > category (severity) : details",
            sortable: true
        }
    ];

    // Fields for issues table
    const issuesFields = [
        ...fields.filter((f) => ["Test_id", "Test_case"].includes(f.key)),
        {
            key: "Test_case_severity",
            label: "Test case severity",
            sortable: true
        },
        {
            key: "Issues_count",
            label: "Issues count",
            sortable: true
        },
        {
            key: "Issues",
            label: "Issues : User > category (severity) : details",
            sortable: true
        },
        {
            key: "Export",
            label: "Export detailed test case",
            sortable: false
        }
    ];

    /**
     * Formated data extracted from the merged results of all particpants
     * @type {import("vue").ComputedRef<import("../utils/mergeResults.js").MergedTestCaseData[]>}
     */
    const mergedData = computed(() => {
        const actualResults = mergedResults.value || [];
        return mergeData(actualResults);
    });

    /**
     * Format an issue in two strings (title + detail)
     * @param {import("../utils/mergeResults.js").ReportedError} i - The issue to expose
     * @return {title : string, detail: string}
     */
    const getIssueString = (i) => {
        return { title: `${i.participantType}${i.participants.length > 1 ? "s" : ""} ${i.participants.join(", ")} > ${i.category} (${i.severity}) : `, detail: `${i.observation}` };
    };

    /**
     * @typedef {import("../utils/mergeResults.js").CompleteTestCase} CompleteTestCase
     */

    /**
     * Detailed list of all identified issues, including naming ones
     * @type {import("vue").ComputedRef<CompleteTestCase[]>}
     */
    const completeIssuesLists = computed(() => {
        return mergedData.value.map(getTestCaseIssuesList);
    });

    /**
     * Map of export functions according to target file type
     */
    const exportFunctions = {
        csv: exportTableIntoCSV,
        md: exportTableIntoMarkdown
    };

    /**
     * Select and format pertinent data from merged results data
     */
    const extractMergedResultsTable = () => {
        return mergedData.value.map(({ testId, timestamp, testCase, expectedParticipants, missedParticipants, callerConversationName, callerConvNameAfterLoad, calleesConversationNames, calleesConvNamesAfterLoad, errors }) => ({
            testId,
            timestamp: timestamp.toISOString(),
            testCase,
            expectedParticipants: expectedParticipants.join(", "),
            missedParticipants: missedParticipants.length > 0 ? missedParticipants.join(", ") : "None",
            callerConversationName,
            callerConvNameAfterLoad,
            calleesConversationNames:
                calleesConversationNames && Object.entries(calleesConversationNames).length > 0
                    ? Object.entries(calleesConversationNames)
                        .map(([name, participants]) => `"${name}" for callee${participants.length > 1 ? "s" : ""} ${participants.join(", ")}`)
                        .join("\n")
                    : "None",
            calleesConvNamesAfterLoad:
                calleesConvNamesAfterLoad && Object.entries(calleesConvNamesAfterLoad).length > 0
                    ? Object.entries(calleesConvNamesAfterLoad ?? {})
                        .map(([name, participants]) => `"${name}" for callee${participants.length > 1 ? "s" : ""} ${participants.join(", ")}`)
                        .join("\n")
                    : "None",
            errors: errors.map((e) => `${getIssueString(e).title}${getIssueString(e).detail}`).join("\n")
        }));
    };

    /**
     * Export pertinent merged results data in a merged_results file of specified extension
     * @param {"csv" | "md"} [to="csv"] - type of the export
     */
    const exportMergedResultsTable = (to = "csv") => {
        const table = extractMergedResultsTable();
        exportFunctions[to](table, "merged_results");
    };

    /**
     * @typedef {{testId: number,timestamp: Date,testCase: string,scenario: number,callType: string,preset: string,testCaseSeverity: number,issuesCount: number,issues: string}[]} TestCasesTable
     */

    /**
     * Select and format pertinent data from complete issues list
     * @return  {TestCasesTable}
     */
    const extractTestCasesTable = () => {
        return completeIssuesLists.value.map(({ testId, timestamp, testCase, scenario, callType, preset, testCaseSeverity, issuesCount, issues }) => ({
            testId,
            timestamp: timestamp.toISOString(),
            testCase,
            scenario,
            callType,
            preset,
            testCaseSeverity,
            issuesCount,
            issues: issues.map((i) => `${getIssueString(i).title}${getIssueString(i).detail}`).join("\n")
        }));
    };

    /**
     * Export pertinent of complete issues list in a issues_by_test_case file of specified extension
     * @param {"csv" | "md"} [to="csv"] - type of the export
     */
    const exportTestCasesTable = (to = "csv") => {
        const table = extractTestCasesTable();
        exportFunctions[to](table, "issues_by_test_case");
    };

    /**
     * @typedef {{testId: number,timestamp: Date,testCase: string,scenario: number,callType: string,preset: string,category: string,severity: number | "None",participantType: "Caller" | "Callee",participantsMsisdn: string,details: string}[]} IssuesTable
     */

    /**
     * Select and format pertinent data from complete issues list or just specified test case
     * @param {CompleteTestCase} [testCase=undefined] - test case to export
     * @return {IssuesTable}
     */
    const extractIssuesTable = (testCase = undefined) => {
        const table = [];

        for (const item of completeIssuesLists.value.filter(({ testId }) => !testCase || testId === testCase.testId)) {
            const { testId, timestamp, testCase, scenario, callType, preset, issues } = item;
            const testCaseBase = {
                testId,
                timestamp: timestamp.toISOString(),
                testCase,
                scenario,
                callType,
                preset,
                category: "None",
                severity: "None",
                participantType: "None",
                participantsMsisdn: "None",
                details: "None"
            };

            if (issues.length === 0) {
                table.push(testCaseBase);
                continue;
            }

            for (const i of issues) {
                table.push({
                    ...testCaseBase,
                    category: i.category,
                    severity: i.severity,
                    participantType: i.participantType,
                    participantsMsisdn: i.participants.join(", "),
                    details: i.observation
                });
            }
        }
        return table;
    };

    /**
     * Export pertinent data of complete issues list or just specified test case in a all_issues or a issues_of_test_case_{testCase.testId} file of specified extension
     * @param {"csv" | "md"} [to="csv"] - type of the export
     * @param {CompleteTestCase} [testCase=undefined] - test case to export
     */
    const exportIssuesTable = (to = "csv", testCase = undefined) => {
        const table = extractIssuesTable(testCase);
        exportFunctions[to](table, testCase ? `issues_of_test_case_${testCase.testId}` : "all_issues", testCase);
    };

    /**
     * Export a markdown document with details on the specified test case descriptions, steps, expected results and issues
     * @param {CompleteTestCase} testCase - test case to export
     */
    const exportTestCaseDocument = (testCase) => {
        exportIssuesTable("md", testCase);
    };

    /**
     * Export all results and documents in a single zip archive
     */
    const exportAllResults = async () => {
        const issuesByTestCase = extractTestCasesTable();
        const allIssues = extractIssuesTable();

        await exportAllResultsAsZip(issuesByTestCase, allIssues, completeIssuesLists.value);
    };

    /**
     * Template ref for the import file input - Side A
     * @type {import('vue').Ref<HTMLInputElement>}
     */
    const importResultsA = useTemplateRef("importResultsA");

    /**
     * Formated data extracted from the second merged results
     * @type {import("vue").Ref<import("../utils/mergeResults.js").MergedTestCaseData[]>}
     */
    const secondMergedResultsData = ref([]);

    /**
     * Handle file import change event
     * @param {Event} evt - File input change event
     * @param {"A" | "B"} resultSide - Identifier of the results among the paire to compare
     * @returns {Promise<void>}
     * @throws {Error} If file parsing fails
     */
    const handleResultsImport = async (evt, resultSide) => {
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
            const data = JSON.parse(text);
            if (resultSide === "A") {
                mergedResults.value = data.map((r) => ({ ...r, timestamp: new Date(r.timestamp) }));
            }
            if (resultSide === "B") {
                secondMergedResultsData.value = mergeData(data);
            }
        } catch (error) {
            console.error("Failed to process JSON file", error);
        }
    };

    // Fields for comparison table
    const comparisonFields = [
        ...issuesFields.filter((f) => f.key === "Test_case"),
        {
            key: "Different_issues_count",
            label: "Different issues count",
            sortable: true
        },
        ...issuesFields.filter((f) => ["Issues"].includes(f.key)).map((f) => ({ ...f, key: `${f.key}-A`, label: `${f.label} A` })),
        ...issuesFields.filter((f) => ["Issues"].includes(f.key)).map((f) => ({ ...f, key: `${f.key}-B`, label: `${f.label} B` }))
    ];

    /**
     * @type {import("vue").ComputedRef<{ testCase: string, A: CompleteTestCase, B: CompleteTestCase,identicalTests : boolean, differentIssues : Array<import("../utils/mergeResults.js").ReportedError & {resultSide : "A" | "B"}>}[]>}
     */
    const comparedTestCases = computed(() => {
        const resultsA = mergedData.value.map(getTestCaseIssuesList);
        const resultsB = secondMergedResultsData.value.map(getTestCaseIssuesList);

        if (resultsA.length === 0 || resultsB.length === 0) {
            return [];
        }

        const commonTestCases = new Set();
        const testCasesData = [];

        for (const testCaseA of resultsA) {
            const match = resultsB.filter((testCaseB) => testCaseA.testCase === testCaseB.testCase);
            if (match.length === 0) {
                continue;
            }
            commonTestCases.add(testCaseA.testCase);

            testCasesData.push({ testCase: testCaseA.testCase, A: testCaseA, B: match[0] });
        }

        for (const testCaseB of resultsB) {
            if (commonTestCases.has(testCaseB.testCase)) {
                continue;
            }

            const match = resultsA.filter((testCaseA) => testCaseA.testCase === testCaseB.testCase);
            if (match.length === 0) {
                continue;
            }
            commonTestCases.add(testCaseB.testCase);
            testCasesData.push({ testCase: testCaseB.testCase, A: match[0], B: testCaseB });
        }

        for (const testCase of testCasesData) {
            const { identicalTests, differentIssues } = compareResults(testCase);
            testCase["identicalTests"] = identicalTests;
            testCase["differentIssues"] = differentIssues;
        }

        return testCasesData;
    });

    /**
     * Extract different issues between run A and run B for a same test case
     * @param {{testCase : string, A : CompleteTestCase, B : CompleteTestCase}} testCasesData
     * @return {{identicalTests : boolean, differentIssues : Array<import("../utils/mergeResults.js").ReportedError & {resultSide : "A" | "B"}>}}
     */
    const compareResults = (testCasesData) => {
        if (testCasesData.length === 0) {
            return;
        }

        const issuesA = testCasesData.A.issues.map((i) => `${getIssueString(i).title}${getIssueString(i).detail}`); // TODO : special compare for duplicates errors that contain ids (=> same type of error will be accounted as different)
        const issuesB = testCasesData.B.issues.map((i) => `${getIssueString(i).title}${getIssueString(i).detail}`);

        const differences = new Set(issuesA).symmetricDifference(new Set(issuesB));

        if (differences.size === 0) {
            return { identicalTests: true, differentIssues: [] };
        }

        const differentIssues = [];

        for (const i of testCasesData.A.issues) {
            if (differences.has(`${getIssueString(i).title}${getIssueString(i).detail}`)) {
                differentIssues.push({ ...i, resultSide: "A" });
            }
        }

        for (const i of testCasesData.B.issues) {
            if (differences.has(`${getIssueString(i).title}${getIssueString(i).detail}`)) {
                differentIssues.push({ ...i, resultSide: "B" });
            }
        }
        return { identicalTests: false, differentIssues };
    };
</script>

<style scoped>
    .error-item {
        margin-bottom: 4px;
        font-size: 0.875rem;
    }
</style>