<template>
    <div class="container-fluid overflow-auto bg-secondary h-100 p-4">
        <b-accordion free>
            <b-accordion-item class="accordion-item" title="Current user tools" visible>
                <CurrentUserTools />
            </b-accordion-item>
            <b-accordion-item v-if="testMode !== 'callee'" class="accordion-item" title="Select test cases" visible>
                <TestCaseSelection />
            </b-accordion-item>
            <b-accordion-item class="accordion-item" title="Test Execution" visible>
                <TestControls />
                <TestResults />
            </b-accordion-item>
            <b-accordion-item v-if="testMode === 'caller'" class="accordion-item" title="Test Analysis" visible>
                <TestAnalysis />
            </b-accordion-item>
        </b-accordion>
    </div>
</template>