<template>
  <b-accordion class="ai-settings" free>
    <b-accordion-item show>
      <template #title>AI Detection Prompts</template>

      <div class="card">

        <!-- HEADER -->
        <header class="card__header">
          <div class="header">
            <div class="header__icon">
              <img src="./icons/ai-brain.svg" alt="AI" />
            </div>

            <div>
              <h2>AI Detection Prompts</h2>
              <p>Configure prompts and data sources for AI detection.</p>
            </div>
          </div>
        </header>

        <!-- TOGGLE -->
        <section class="section row">
          <div class="label">
            Enable anomaly detection
            <img src="./icons/info.svg" alt="" />
          </div>

          <label class="switch">
            <input type="checkbox" checked />
            <span class="slider"></span>
          </label>
        </section>

        <!-- SAVED PROMPTS -->
        <section class="section">
          <h3 class="section__title">Saved prompts</h3>

          <div class="cards">
            <div class="prompt">
              <div class="prompt__title">Protected Site Intrusion</div>
              <div class="prompt__text">
                Detect unauthorized access or presence in protected sites...
              </div>
            </div>

            <div class="prompt">
              <div class="prompt__title">Restricted Area Presence</div>
              <div class="prompt__text">
                Detect individuals entering restricted operational zones...
              </div>
            </div>

            <div class="prompt">
              <div class="prompt__title">Alarm Escalation</div>
              <div class="prompt__text">
                Detect alarm events requiring immediate escalation...
              </div>
            </div>
          </div>
        </section>

        <!-- PROFILE + PROMPT -->
        <section class="grid">
          <div>
            <label class="label">
              Detection profile
              <img src="./icons/info.svg" alt="" />
            </label>

            <select class="input">
              <option>Protected Site Intrusion</option>
            </select>
          </div>

          <div>
            <label class="label">
              Detection prompt
              <img src="./icons/info.svg" alt="" />
            </label>

            <textarea class="textarea" rows="7">
You are an AI assistant for public safety. Detect potential unauthorized access or intrusion at protected sites.

Consider unauthorized entry, restricted zones, alarms, and escalation signals.

Provide confidence, evidence, uncertainty, and verification steps.
            </textarea>
          </div>
        </section>

        <!-- DATA SOURCES -->
        <section class="section">
          <h3 class="section__title">Data sources</h3>

          <div class="chips">
            <button class="chip chip--active">Radio transcripts</button>
            <button class="chip chip--active">Browser messages</button>
            <button class="chip chip--active">Alarm events</button>
            <button class="chip">Unit status</button>
          </div>
        </section>

        <!-- ACTIONS -->
        <footer class="footer">
          <button class="btn btn--primary">Save</button>
        </footer>

      </div>
    </b-accordion-item>
  </b-accordion>
</template>

<script>
export default {
  name: "SettingsPrompts",
};
</script>

<style scoped>
/* =========================
   BASE
========================= */

.ai-settings {
  background: #fff;
}

.card {
  background: #ffffff;
  border: 1px solid #e6e8ee;
  border-radius: 14px;
  padding: 24px;
  color: #1f2937;
  position: relative;
}

/* subtle depth */
.card::before {
  content: "";
  position: absolute;
  inset: 0;
  border-radius: inherit;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.04);
  pointer-events: none;
}

/* =========================
   HEADER
========================= */

.header {
  display: flex;
  gap: 14px;
  align-items: center;
}

.header__icon {
  width: 42px;
  height: 42px;
  border-radius: 10px;
  background: #eef6ff;
  display: flex;
  align-items: center;
  justify-content: center;
}

.header__icon img {
  width: 20px;
}

h2 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
}

p {
  margin: 2px 0 0;
  font-size: 13px;
  color: #6b7280;
}

/* =========================
   SECTIONS
========================= */

.section {
  margin-top: 22px;
}

.section__title {
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 10px;
  color: #374151;
}

.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 0;
  border-top: 1px solid #eef0f4;
  border-bottom: 1px solid #eef0f4;
}

/* =========================
   LABELS
========================= */

.label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  color: #374151;
}

.label img {
  width: 14px;
  opacity: 0.6;
}

/* =========================
   INPUTS
========================= */

.input,
.textarea {
  width: 100%;
  border: 1px solid #dcdfe6;
  border-radius: 10px;
  padding: 10px 12px;
  font-size: 13px;
  background: #fff;
  outline: none;
}

.textarea {
  resize: none;
}

.input:focus,
.textarea:focus {
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.12);
}

/* =========================
   PROMPT CARDS
========================= */

.cards {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.prompt {
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 12px 14px;
  background: #fafafa;
  transition: 0.2s;
}

.prompt:hover {
  border-color: #3b82f6;
  background: #f8fbff;
}

.prompt__title {
  font-weight: 500;
  font-size: 14px;
}

.prompt__text {
  font-size: 13px;
  color: #6b7280;
  margin-top: 4px;
}

/* =========================
   GRID
========================= */

.grid {
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 18px;
  margin-top: 22px;
}

/* =========================
   CHIPS
========================= */

.chips {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.chip {
  border: 1px solid #dcdfe6;
  background: #fff;
  padding: 8px 12px;
  border-radius: 999px;
  font-size: 13px;
  cursor: pointer;
  transition: 0.2s;
}

.chip--active {
  border-color: #3b82f6;
  background: #eef6ff;
  color: #1d4ed8;
}

/* =========================
   BUTTONS
========================= */

.footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 26px;
}

.btn {
  height: 42px;
  padding: 0 18px;
  border-radius: 10px;
  font-size: 13px;
  border: none;
  cursor: pointer;
}

.btn--primary {
  background: #3b82f6;
  color: white;
}

.btn--primary:hover {
  background: #2563eb;
}

/* =========================
   SWITCH
========================= */

.switch {
  position: relative;
  width: 46px;
  height: 26px;
}

.switch input {
  display: none;
}

.slider {
  position: absolute;
  inset: 0;
  background: #e5e7eb;
  border-radius: 999px;
  transition: 0.2s;
}

.slider::before {
  content: "";
  position: absolute;
  width: 20px;
  height: 20px;
  top: 3px;
  left: 3px;
  background: white;
  border-radius: 50%;
  transition: 0.2s;
  box-shadow: 0 2px 6px rgba(0,0,0,0.15);
}

.switch input:checked + .slider {
  background: #3b82f6;
}

.switch input:checked + .slider::before {
  transform: translateX(20px);
}

/* =========================
   RESPONSIVE
========================= */

@media (max-width: 900px) {
  .grid {
    grid-template-columns: 1fr;
  }
}
</style>
