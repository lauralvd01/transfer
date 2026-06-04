<template>
  <b-accordion class="ai-settings" free>
    <b-accordion-item show>
      <template #title>AI Detection Prompts</template>

      <div class="card">

        <!-- HEADER -->
        <header class="header">
          <div class="header__icon">
            <img src="./icons/ai-brain.svg" alt="AI" />
          </div>

          <div>
            <h2>AI Detection Prompts</h2>
            <p>Manage detection prompts and data sources.</p>
          </div>
        </header>

        <!-- ENABLE TOGGLE -->
        <section class="toggle">
          <span>Enable anomaly detection</span>

          <label class="switch">
            <input type="checkbox" v-model="enabled" />
            <span class="slider"></span>
          </label>
        </section>

        <!-- EVERYTHING DISABLED WHEN OFF -->
        <div v-if="enabled" class="layout">

          <!-- LEFT: LIST -->
          <aside class="sidebar">

            <div class="sidebar__title">Saved prompts</div>

            <div
              class="item new"
              :class="{ active: selectedId === 'new' }"
              @click="selectNew"
            >
              + New prompt
            </div>

            <div
              v-for="p in prompts"
              :key="p.id"
              class="item"
              :class="{ active: selectedId === p.id }"
              @click="selectPrompt(p)"
            >
              {{ p.title }}
            </div>

          </aside>

          <!-- RIGHT: EDITOR -->
          <main class="editor">

            <div class="field">
              <label>Prompt title</label>
              <input v-model="activePrompt.title" class="input" />
            </div>

            <div class="field">
              <label>Detection prompt</label>
              <textarea v-model="activePrompt.text" class="textarea" rows="8" />
            </div>

            <div class="field">
              <label>Data sources</label>

              <div class="chips">
                <button class="chip" :class="{ on: true }">Radio</button>
                <button class="chip" :class="{ on: true }">Browser</button>
                <button class="chip" :class="{ on: true }">Alarms</button>
                <button class="chip">Unit status</button>
              </div>
            </div>

            <div class="actions">
              <button class="btn" @click="savePrompt">
                Save
              </button>
            </div>

          </main>

        </div>
      </div>
    </b-accordion-item>
  </b-accordion>
</template>

<script>
export default {
  name: "SettingsPrompts",

  data() {
    return {
      enabled: true,

      prompts: [
        {
          id: 1,
          title: "Protected Site Intrusion",
          text: "Detect unauthorized access at protected sites..."
        },
        {
          id: 2,
          title: "Restricted Area Presence",
          text: "Detect presence in restricted zones..."
        }
      ],

      selectedId: null,

      activePrompt: {
        id: null,
        title: "",
        text: ""
      }
    };
  },

  mounted() {
    this.selectPrompt(this.prompts[0]);
  },

  methods: {
    selectPrompt(p) {
      this.selectedId = p.id;
      this.activePrompt = { ...p };
    },

    selectNew() {
      this.selectedId = "new";
      this.activePrompt = {
        id: Date.now(),
        title: "",
        text: ""
      };
    },

    savePrompt() {
      const existingIndex = this.prompts.findIndex(
        p => p.id === this.activePrompt.id
      );

      if (existingIndex >= 0) {
        this.prompts.splice(existingIndex, 1, { ...this.activePrompt });
      } else {
        this.prompts.unshift({ ...this.activePrompt });
      }

      this.selectedId = this.activePrompt.id;
    }
  }
};
</script>

<style scoped>

/* =========================
   CARD
========================= */

.card {
  background: #fff;
  border: 1px solid #e6e8ee;
  border-radius: 14px;
  padding: 20px;
}

/* =========================
   HEADER
========================= */

.header {
  display: flex;
  gap: 12px;
  align-items: center;
  margin-bottom: 16px;
}

.header__icon {
  width: 40px;
  height: 40px;
  background: #eef6ff;
  border-radius: 10px;
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
}

p {
  margin: 2px 0 0;
  font-size: 13px;
  color: #6b7280;
}

/* =========================
   TOGGLE
========================= */

.toggle {
  display: flex;
  justify-content: space-between;
  align-items: center;

  padding: 14px 0;
  border-top: 1px solid #eef0f4;
  border-bottom: 1px solid #eef0f4;
  margin-bottom: 16px;
}

/* =========================
   LAYOUT
========================= */

.layout {
  display: grid;
  grid-template-columns: 260px 1fr;
  gap: 18px;
}

/* =========================
   SIDEBAR
========================= */

.sidebar {
  border-right: 1px solid #eef0f4;
  padding-right: 12px;
}

.sidebar__title {
  font-size: 12px;
  text-transform: uppercase;
  color: #6b7280;
  margin-bottom: 10px;
}

.item {
  padding: 10px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  color: #374151;
  margin-bottom: 6px;
}

.item:hover {
  background: #f3f6ff;
}

.item.active {
  background: #eaf2ff;
  color: #2563eb;
  font-weight: 500;
}

.item.new {
  border: 1px dashed #cbd5e1;
  color: #2563eb;
}

/* =========================
   EDITOR
========================= */

.editor {
  padding-left: 6px;
}

.field {
  margin-bottom: 14px;
}

label {
  display: block;
  font-size: 12px;
  margin-bottom: 6px;
  color: #374151;
}

.input,
.textarea {
  width: 100%;
  border: 1px solid #dcdfe6;
  border-radius: 10px;
  padding: 10px;
  font-size: 13px;
  outline: none;
}

.textarea:focus,
.input:focus {
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59,130,246,0.1);
}

/* =========================
   CHIPS
========================= */

.chips {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.chip {
  border: 1px solid #dcdfe6;
  padding: 6px 10px;
  border-radius: 999px;
  font-size: 12px;
  background: #fff;
}

.chip.on {
  background: #eaf2ff;
  border-color: #3b82f6;
  color: #2563eb;
}

/* =========================
   ACTIONS
========================= */

.actions {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}

.btn {
  background: #2563eb;
  color: #fff;
  border: none;
  padding: 10px 16px;
  border-radius: 10px;
  cursor: pointer;
}

.btn:hover {
  background: #1d4ed8;
}

/* =========================
   SWITCH
========================= */

.switch {
  position: relative;
  width: 44px;
  height: 24px;
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
  width: 18px;
  height: 18px;
  top: 3px;
  left: 3px;
  background: white;
  border-radius: 50%;
  transition: 0.2s;
}

.switch input:checked + .slider {
  background: #3b82f6;
}

.switch input:checked + .slider::before {
  transform: translateX(20px);
}
</style>
