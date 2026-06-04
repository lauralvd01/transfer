<!--
  Copyright © 2020-2026 Airbus DS SLC. All rights reserved.
-->
<template>
    <b-accordion class="options__option mt-2" free>
        <b-accordion-item show>
            <template #title>AI Detection Prompts</template>

            <div class="ai-card">
                <!-- HEADER -->

                <div class="header-row">
                    <div class="header-left">
                        <div class="header-icon">
                            <img src="./icons/ai-brain.svg" class="ai-icon" alt="" />
                        </div>

                        <div>
                            <h3>AI Detection Prompts</h3>
                            <p>Configure prompts and data sources for AI detection.</p>
                        </div>
                    </div>
                </div>

                <!-- TOGGLE -->

                <div class="section-divider">
                    <div class="label-inline">
                        Enable anomaly detection

                        <img src="./icons/info.svg" class="info-icon" alt="" />
                    </div>

                    <label class="switch">
                        <input type="checkbox" checked />
                        <span class="slider" />
                    </label>
                </div>

                <div class="section saved-prompts-section">
                    <label class="field-label">
                        Saved prompts
                        <img src="./icons/info.svg" class="info-icon" alt="" />
                    </label>

                    <div class="saved-prompts">
                        <div class="saved-prompt-card">
                            <div class="saved-prompt-header">
                                <span>Protected Site Intrusion</span>
                            </div>

                            <div class="saved-prompt-preview">Detect unauthorized access or presence in protected
                                sites...</div>
                        </div>

                        <div class="saved-prompt-card">
                            <div class="saved-prompt-header">
                                <span>Restricted Area Presence</span>
                            </div>

                            <div class="saved-prompt-preview">Detect individuals entering restricted operational
                                zones...</div>
                        </div>

                        <div class="saved-prompt-card">
                            <div class="saved-prompt-header">
                                <span>Alarm Escalation</span>
                            </div>

                            <div class="saved-prompt-preview">Detect alarm events requiring immediate escalation...
                            </div>
                        </div>
                    </div>
                </div>

                <!-- PROFILE + PROMPT -->

                <div class="content-grid">
                    <div>
                        <label class="field-label">
                            Detection profile
                            <img src="./icons/info.svg" class="info-icon" alt="" />
                        </label>

                        <select class="ai-select">
                            <option>Protected Site Intrusion</option>
                        </select>
                    </div>

                    <div>
                        <label class="field-label">
                            Detection prompt
                            <img src="./icons/info.svg" class="info-icon" alt="" />
                        </label>

                        <textarea rows="7" class="ai-textarea">
You are an AI assistant for public safety. Detect potential unauthorized access or intrusion at protected sites.

Consider unauthorized entry or presence in restricted areas, intrusion attempts, urgent radio traffic, and alarm escalations.

Provide confidence (0-100%), evidence (key signals), uncertainty (factors reducing certainty), and recommended verification steps.
                        </textarea>
                    </div>
                </div>

                <!-- DATA SOURCES -->

                <div class="section">
                    <label class="field-label">
                        Data sources
                        <img src="./icons/info.svg" class="info-icon" alt="" />
                    </label>

                    <div class="chips">
                        <button class="chip active">
                            <img src="./icons/check.svg" alt="" />
                            Radio transcripts
                        </button>

                        <button class="chip active">
                            <img src="./icons/check.svg" alt="" />
                            Browser messages
                        </button>

                        <button class="chip active">
                            <img src="./icons/check.svg" alt="" />
                            Alarm events
                        </button>

                        <button class="chip">Unit status</button>
                    </div>
                </div>

                <!-- ACTIONS -->

                <div class="actions">
                    <button class="secondary-btn">
                        <img src="./icons/save.svg" alt="" />
                        Save
                    </button>
                </div>
            </div>
        </b-accordion-item>
    </b-accordion>
</template>

<script>
    export default {
        name: "SettingsPrompts"
    };
</script>

<style scoped>
    .ai-card {
        /* original dark‑gradient background – removed */
        background: #ffffff;
        /* plain white background */
        border: 1px solid #e0e0e0;
        /* light border */
        border-radius: 14px;
        padding: 22px;
        color: #000000;
        /* default text colour for readability */
    }

    /* HEADER */

    .header-row {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;

        margin-bottom: 24px;
    }

    .header-left {
        display: flex;
        gap: 14px;
    }

    .header-icon {
        width: 40px;
        height: 40px;

        border-radius: 10px;

        display: flex;
        align-items: center;
        justify-content: center;

        background: rgba(0, 217, 255, 0.08);
    }

    .ai-icon {
        filter: brightness(0) saturate(100%) invert(84%) sepia(33%) saturate(6434%) hue-rotate(153deg) brightness(102%) contrast(104%);
        filter: drop-shadow(0 0 4px rgba(0, 217, 255, 0.85)) drop-shadow(0 0 12px rgba(0, 217, 255, 0.45));
    }

    .header-icon img {
        width: 150%;
    }

    .header-left h3 {
        margin: 0;
        font-size: 18px;
        font-weight: 600;
        color: #222222;
    }

    .header-left p {
        margin: 2px 0 0;
        font-size: 13px;
        color: #555555;
    }

    .collapse-icon {
        width: 16px;
        opacity: 0.8;
    }

    /* DIVIDER */

    .section-divider {
        border-top: 1px solid rgba(255, 255, 255, 0.06);
        border-bottom: 1px solid rgba(255, 255, 255, 0.06);

        padding: 18px 0;

        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .label-inline {
        display: flex;
        align-items: center;
        gap: 8px;

        color: #333333;
        font-size: 14px;
    }

    .label-inline img {
        width: 14px;
        opacity: 0.6;
    }

    .info-icon img {
        width: 200px;
    }

    /* GRID */

    .content-grid {
        margin-top: 24px;

        display: grid;
        grid-template-columns: 320px 1fr;
        gap: 20px;
    }

    .field-label {
        display: flex;
        align-items: center;
        gap: 6px;

        margin-bottom: 10px;

        color: #333333;

        font-size: 13px;
    }

    .field-label img {
        width: 14px;
        opacity: 0.6;
    }

    .ai-select,
    .ai-textarea {
        width: 100%;

        background: #071220;

        border: 1px solid rgba(80, 120, 255, 0.16);

        border-radius: 10px;

        color: white;

        padding: 12px 14px;

        font-size: 13px;
    }

    .ai-select {
        height: 44px;
    }

    .ai-textarea {
        resize: none;
    }

    .ai-select,
    .ai-textarea {
        background: #fafafa;
        border: 1px solid #d0d0d0;
        color: #000000;
    }

    .ai-select:focus,
    .ai-textarea:focus {
        border-color: #0099ff;
        box-shadow: 0 0 0 2px rgba(0, 153, 255, 0.12);
    }

    /* CHIPS */

    .section {
        margin-top: 28px;
    }

    .chips {
        display: flex;
        flex-wrap: wrap;
        gap: 10px;
    }

    .chip {
        height: 42px;

        padding: 0 16px;

        border-radius: 10px;

        background: #071220;

        border: 1px solid rgba(80, 120, 255, 0.18);

        color: #333333;

        font-size: 13px;

        display: flex;
        align-items: center;
        gap: 8px;
    }

    .chip img {
        width: 14px;
    }

    .chip.active {
        border-color: #0099ff;
        background: linear-gradient(180deg, rgba(0, 153, 255, 0.12), rgba(0, 153, 255, 0.04));
    }

    /* BOTTOM */

    .bottom-grid {
        margin-top: 28px;

        display: grid;
        grid-template-columns: 300px 1fr;
        gap: 20px;
    }

    .segmented {
        display: flex;

        border-radius: 10px;
        overflow: hidden;

        border: 1px solid rgba(80, 120, 255, 0.18);
    }

    .segmented button {
        flex: 1;

        height: 44px;

        background: #071220;
        border: none;

        color: #dbe6ff;

        font-size: 13px;
    }

    .segmented button.active {
        background: linear-gradient(180deg, rgba(126, 73, 255, 0.45), rgba(126, 73, 255, 0.25));

        box-shadow: inset 0 0 0 1px rgba(161, 118, 255, 0.5);
    }

    .status-card {
        display: flex;
        align-items: center;
        gap: 18px;

        padding: 18px 22px;

        border-radius: 12px;

        background: linear-gradient(90deg, rgba(0, 255, 170, 0.12), rgba(0, 255, 170, 0.03));

        border: 1px solid rgba(0, 255, 170, 0.25);
    }

    .status-icon img {
        width: 34px;
    }

    .status-title {
        color: white;
        font-size: 15px;
        font-weight: 600;
    }

    .status-text {
        margin-top: 4px;

        color: #93a3c3;
        font-size: 13px;
    }

    /* ACTIONS */

    .actions {
        margin-top: 28px;

        display: flex;
        justify-content: flex-end;
        gap: 12px;

        flex-wrap: wrap;
    }

    .actions button {
        color: #333333;
    }

    .secondary-btn,
    .primary-btn {
        height: 44px;

        border-radius: 10px;

        padding: 0 18px;

        display: flex;
        align-items: center;
        gap: 8px;

        font-size: 13px;
    }

    .secondary-btn {
        background: #071220;

        border: 1px solid rgba(80, 120, 255, 0.18);

        color: white;
    }

    .primary-btn {
        border: none;

        color: white;

        background: linear-gradient(90deg, #3670ff, #7c4dff);

        min-width: 190px;
    }

    .secondary-btn img,
    .primary-btn img {
        width: 14px;
    }

    /* SWITCH */

    .switch {
        position: relative;
        width: 54px;
        height: 30px;
    }

    .switch input {
        display: none;
    }

    .slider {
        position: absolute;
        inset: 0;

        background: #0d213d;

        border: 1px solid rgba(0, 216, 255, 0.25);

        border-radius: 30px;
    }

    .slider::before {
        content: "";

        position: absolute;

        width: 22px;
        height: 22px;

        right: 4px;
        top: 3px;

        background: white;

        border-radius: 50%;
    }

    /* RESPONSIVE */

    @media (max-width: 992px) {

        .content-grid,
        .bottom-grid {
            grid-template-columns: 1fr;
        }

        .actions {
            justify-content: stretch;
        }

        .actions button {
            width: 100%;
        }
    }

    @media (max-width: 576px) {
        .ai-card {
            padding: 16px;
        }

        .header-row {
            flex-direction: column;
            gap: 12px;
        }

        .header-left {
            width: 100%;
        }

        .chip {
            width: 100%;
            justify-content: flex-start;
        }
    }

    .saved-prompts {
        display: flex;
        flex-direction: column;
        gap: 12px;
    }

    .saved-prompt-card {
        background: linear-gradient(180deg, rgba(17, 30, 50, 0.65), rgba(10, 18, 32, 0.65));

        border: 1px solid rgba(85, 120, 255, 0.12);

        border-radius: 12px;

        transition: all 0.2s ease;

        overflow: hidden;
    }

    .saved-prompt-card:hover {
        border-color: rgba(0, 217, 255, 0.28);

        box-shadow: 0 0 12px rgba(0, 217, 255, 0.08);
    }

    .saved-prompt-header {
        height: 52px;

        padding: 0 18px;

        display: flex;
        align-items: center;
        justify-content: space-between;

        cursor: pointer;
    }

    .saved-prompt-header span {
        color: #dfe8ff;

        font-size: 14px;
        font-weight: 500;
    }

    .saved-prompt-preview {
        border-top: 1px solid rgba(255, 255, 255, 0.04);

        padding: 14px 18px;

        color: #8f9ec1;

        font-size: 13px;

        line-height: 1.5;
    }

    /* ------------------------------------------------ */
    /* ICONS */
    /* ------------------------------------------------ */

    .header-icon img {
        width: 18px;

        filter: drop-shadow(0 0 4px rgba(0, 217, 255, 0.85)) drop-shadow(0 0 12px rgba(0, 217, 255, 0.45));
    }

    .field-label img {
        width: 14px;

        opacity: 0.7;
    }

    .chip img {
        width: 14px;

        filter: drop-shadow(0 0 3px rgba(0, 217, 255, 0.8));
    }

    /* ------------------------------------------------ */
    /* SAVE BUTTON */
    /* ------------------------------------------------ */

    .save-btn {
        height: 44px;

        min-width: 140px;

        border: 1px solid rgba(80, 120, 255, 0.18);

        background: #071220;

        color: white;

        border-radius: 10px;

        display: flex;
        align-items: center;
        justify-content: center;
        gap: 8px;

        transition: 0.2s;
    }

    .save-btn:hover {
        border-color: rgba(0, 217, 255, 0.25);

        box-shadow: 0 0 12px rgba(0, 217, 255, 0.08);

        background: #091728;
    }

    .save-btn img {
        width: 14px;
    }

    /* ------------------------------------------------ */
    /* CARD GLOW */
    /* ------------------------------------------------ */

    .ai-card {
        position: relative;
        overflow: hidden;
    }

    .ai-card::before {
        content: "";

        position: absolute;
        inset: 0;

        border-radius: inherit;

        pointer-events: none;

        box-shadow:
            inset 0 0 0 1px rgba(76, 132, 255, 0.08),
            0 0 40px rgba(31, 97, 255, 0.08);
    }

    .ai-card::after {
        content: "";

        position: absolute;

        top: -120px;
        right: -120px;

        width: 280px;
        height: 280px;

        border-radius: 50%;

        background: radial-gradient(circle, rgba(0, 217, 255, 0.12), transparent 70%);

        pointer-events: none;
    }
</style>