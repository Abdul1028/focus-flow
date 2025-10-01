<template>
  <section class="card">
    <header class="card-header">
      <h2>Pomodoro</h2>
      <div class="settings">
        <label>Focus <input type="number" min="1" v-model.number="focusMinutes" @change="saveSettings" /> min</label>
        <label>Break <input type="number" min="1" v-model.number="breakMinutes" @change="saveSettings" /> min</label>
        <label>Long Break <input type="number" min="1" v-model.number="longBreakMinutes" @change="saveSettings" /> min</label>
        <label>Every <input type="number" min="2" v-model.number="longBreakEvery" @change="saveSettings" /> cycles</label>
      </div>
    </header>

    <div class="timer">
      <div class="mode" :class="mode">{{ modeLabel }}</div>
      <div class="time">{{ mm }}:{{ ss }}</div>
      <div class="controls">
        <button @click="toggle">{{ running ? 'Pause' : 'Start' }}</button>
        <button @click="reset">Reset</button>
        <button @click="skip" :disabled="!running">Skip</button>
      </div>
    </div>

    <footer class="card-footer">
      <span>Completed cycles: {{ completedCycles }}</span>
      <div class="spacer"></div>
      <label><input type="checkbox" v-model="sound" @change="saveSettings" /> Sound</label>
      <label><input type="checkbox" v-model="notify" @change="saveSettings" /> Notifications</label>
    </footer>
    <audio ref="beep">
      <source src="data:audio/wav;base64,UklGRiQAAABXQVZFZm10IBAAAAABAAEAESsAACJWAAACABAAZGF0YQAAAAA=" type="audio/wav" />
    </audio>
  </section>
</template>

<script>
const STORAGE_KEY = 'focusflow.pomodoro.v1'

export default {
  name: 'PomodoroTimer',
  data() {
    const saved = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}')
    return {
      focusMinutes: saved.focusMinutes || 25,
      breakMinutes: saved.breakMinutes || 5,
      longBreakMinutes: saved.longBreakMinutes || 15,
      longBreakEvery: saved.longBreakEvery || 4,
      notify: saved.notify ?? true,
      sound: saved.sound ?? true,
      running: false,
      mode: 'focus',
      remaining: (saved.focusMinutes || 25) * 60,
      completedCycles: saved.completedCycles || 0,
      intervalId: null
    }
  },
  computed: {
    mm() { return String(Math.floor(this.remaining / 60)).padStart(2, '0') },
    ss() { return String(this.remaining % 60).padStart(2, '0') },
    modeLabel() { return this.mode === 'focus' ? 'Focus' : (this.mode === 'break' ? 'Break' : 'Long Break') }
  },
  methods: {
    saveSettings() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify({
        focusMinutes: this.focusMinutes,
        breakMinutes: this.breakMinutes,
        longBreakMinutes: this.longBreakMinutes,
        longBreakEvery: this.longBreakEvery,
        notify: this.notify,
        sound: this.sound,
        completedCycles: this.completedCycles
      }))
      if (!this.running) this.reset()
    },
    requestNotifyPermission() {
      if (!('Notification' in window)) return
      if (Notification.permission === 'default') Notification.requestPermission()
    },
    toggle() {
      this.running = !this.running
      if (this.running) this.tickStart(); else this.tickStop()
    },
    tickStart() {
      this.requestNotifyPermission()
      this.intervalId = setInterval(() => {
        if (this.remaining > 0) {
          this.remaining--
        } else {
          this.onPhaseEnd()
        }
      }, 1000)
    },
    tickStop() { if (this.intervalId) clearInterval(this.intervalId); this.intervalId = null },
    reset() {
      this.tickStop()
      this.running = false
      this.remaining = (this.mode === 'focus' ? this.focusMinutes : this.mode === 'break' ? this.breakMinutes : this.longBreakMinutes) * 60
    },
    skip() { this.remaining = 0 },
    onPhaseEnd() {
      this.alertUser()
      if (this.mode === 'focus') {
        this.completedCycles++
        if (this.completedCycles % this.longBreakEvery === 0) {
          this.mode = 'long'
          this.remaining = this.longBreakMinutes * 60
        } else {
          this.mode = 'break'
          this.remaining = this.breakMinutes * 60
        }
      } else {
        this.mode = 'focus'
        this.remaining = this.focusMinutes * 60
      }
      this.saveSettings()
    },
    alertUser() {
      if (this.sound && this.$refs.beep) {
        try { this.$refs.beep.play() } catch (e) {}
      }
      if (this.notify && 'Notification' in window && Notification.permission === 'granted') {
        const title = this.mode === 'focus' ? 'Focus complete' : (this.mode === 'break' ? 'Break over' : 'Long break over')
        new Notification(title, { body: 'Next session starting', silent: !this.sound })
      }
    }
  },
  beforeUnmount() { this.tickStop() }
}
</script>

<style scoped>
.card { background: #fff; border: 1px solid #e5e7eb; border-radius: 12px; padding: 16px; }
.card-header { display: flex; align-items: center; justify-content: space-between; gap: 12px; margin-bottom: 12px; flex-wrap: wrap; }
.settings { display: flex; gap: 8px; flex-wrap: wrap; align-items: center; }
.settings input { width: 64px; }
.timer { display: grid; place-items: center; gap: 12px; padding: 16px; }
.mode { font-weight: 600; }
.mode.focus { color: #111827; }
.mode.break { color: #065f46; }
.mode.long { color: #1e40af; }
.time { font-size: 48px; font-weight: 700; letter-spacing: 2px; }
.controls { display: flex; gap: 8px; }
input, button { border: 1px solid #e5e7eb; border-radius: 8px; padding: 8px 10px; }
button { background: #111827; color: #fff; cursor: pointer; }
button:disabled { background: #9ca3af; cursor: not-allowed; }
.card-footer { display: flex; align-items: center; gap: 8px; margin-top: 12px; }
.spacer { flex: 1; }
</style>


