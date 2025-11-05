<template>
  <div class="app">
    <header class="app-header">
      <img alt="Vue logo" src="./assets/logo.png" class="logo" />
      <h1>FocusFlow</h1>
      <p class="subtitle">Tasks • Pomodoro • Notes</p>
    </header>

    <nav class="tabs">
      <button :class="{ active: activeTab==='tasks' }" @click="setTab('tasks')">Tasks</button>
      <button :class="{ active: activeTab==='pomodoro' }" @click="setTab('pomodoro')">Pomodoro</button>
      <button :class="{ active: activeTab==='notes' }" @click="setTab('notes')">Notes</button>
    </nav>

    <main class="tab-panel">
      <TaskManager v-if="activeTab==='tasks'" />
      <PomodoroTimer v-if="activeTab==='pomodoro'" />
      <NotesBoard v-if="activeTab==='notes'" />
    </main>
  </div>
</template>

<script>
import TaskManager from './components/TaskManager.vue'
import PomodoroTimer from './components/PomodoroTimer.vue'
import NotesBoard from './components/NotesBoard.vue'

const TAB_KEY = 'focusflow.activeTab'

export default {
  name: 'App',
  components: { TaskManager, PomodoroTimer, NotesBoard },
  data() {
    return {
      activeTab: localStorage.getItem(TAB_KEY) || 'tasks'
    }
  },
  methods: {
    setTab(tab) {
      this.activeTab = tab
      localStorage.setItem(TAB_KEY, tab)
    }
  }
}
</script>

<style>
.app {
  max-width: 960px;
  margin: 0 auto;
  padding: 24px 16px;
}
.app-header {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 16px;
}
.logo { width: 64px; height: 64px; }
.subtitle { color: #6b7280; margin: 0; }
.tabs { display: flex; gap: 8px; justify-content: center; margin-bottom: 16px; flex-wrap: wrap; }
.tabs button {
  border: 1px solid #e5e7eb;
  background: #fff;
  padding: 8px 14px;
  border-radius: 8px;
  cursor: pointer;
}
.tabs button.active { background: #111827; color: #fff; border-color: #111827; }
.tab-panel { text-align: left; }
</style>
