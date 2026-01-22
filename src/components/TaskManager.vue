<template>
  <section class="card">
    <header class="card-header">
      <h2>Tasks</h2>
      <div class="filters">
        <select v-model="filterStatus">
          <option value="all">All</option>
          <option value="active">Active</option>
          <option value="done">Completed</option>
        </select>
        <input v-model="query" placeholder="Search tasks..." />
      </div>
    </header>

    <form class="task-form" @submit.prevent="addTask">
      <input v-model="newTitle" placeholder="What needs to be done?" />
      <button type="submit">Add TasSK</button>
    </form>

    <ul class="task-list">
      <li v-for="task in filteredTasks" :key="task.id" :class="{ done: task.done }">
        <label>
          <input type="checkbox" v-model="task.done" @change="persist()" />
          <span v-if="!task.editing" @dblclick="startEdit(task)">{{ task.title }}</span>
          <input v-else v-model="task.title" @keyup.enter="stopEdit(task)" @blur="stopEdit(task)" />
        </label>
        <div class="actions">
          <button @click="startEdit(task)" v-if="!task.editing">Edit</button>
          <button @click="stopEdit(task)" v-else>Save</button>
          <button class="danger" @click="removeTask(task.id)">Delete</button>
        </div>
      </li>
    </ul>

    <footer class="card-footer">
      <span>{{ remaining }} items left</span>
      <div class="spacer"></div>
      <button @click="clearCompleted" :disabled="completed===0">Clear completed ({{ completed }})</button>
    </footer>
  </section>
  
</template>

<script>
const STORAGE_KEY = 'focusflow.tasks'

export default {
  name: 'TaskManager',
  data() {
    return {
      tasks: JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'),
      newTitle: '',
      filterStatus: 'all',
      query: ''
    }
  },
  computed: {
    filteredTasks() {
      const q = this.query.trim().toLowerCase()
      return this.tasks
        .filter(t => {
          if (this.filterStatus === 'active') return !t.done
          if (this.filterStatus === 'done') return t.done
          return true
        })
        .filter(t => (q ? t.title.toLowerCase().includes(q) : true))
    },
    remaining() { return this.tasks.filter(t => !t.done).length },
    completed() { return this.tasks.filter(t => t.done).length }
  },
  methods: {
    persist() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(this.tasks))
    },
    addTask() {
      const title = this.newTitle.trim()
      if (!title) return
      this.tasks.unshift({ id: Date.now(), title, done: false, editing: false })
      this.newTitle = ''
      this.persist()
    },
    removeTask(id) {
      this.tasks = this.tasks.filter(t => t.id !== id)
      this.persist()
    },
    clearCompleted() {
      this.tasks = this.tasks.filter(t => !t.done)
      this.persist()
    },
    startEdit(task) {
      this.tasks.forEach(t => (t.editing = false))
      task.editing = true
    },
    stopEdit(task) {
      task.editing = false
      task.title = (task.title || '').trim()
      if (!task.title) this.removeTask(task.id)
      this.persist()
    }
  }
}
</script>

<style scoped>
.card { background: #fff; border: 1px solid #e5e7eb; border-radius: 12px; padding: 16px; }
.card-header { display: flex; align-items: center; justify-content: space-between; gap: 12px; margin-bottom: 12px; }
.filters { display: flex; gap: 8px; }
.task-form { display: flex; gap: 8px; margin-bottom: 12px; }
.task-form input { flex: 1; }
.task-list { list-style: none; padding: 0; margin: 0; display: grid; gap: 8px; }
.task-list li { display: flex; align-items: center; justify-content: space-between; border: 1px solid #eef2f7; padding: 10px 12px; border-radius: 8px; }
.task-list li.done span { text-decoration: line-through; color: #9ca3af; }
.actions { display: flex; gap: 6px; }
.danger { color: #b91c1c; }
.card-footer { display: flex; align-items: center; gap: 8px; margin-top: 12px; }
.spacer { flex: 1; }
input, select, button { border: 1px solid #e5e7eb; border-radius: 8px; padding: 8px 10px; }
button { background: #111827; color: #fff; cursor: pointer; }
button:disabled { background: #9ca3af; cursor: not-allowed; }
</style>


