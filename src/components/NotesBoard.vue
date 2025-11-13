<template>
  <section class="card">
    <header class="card-header">
      <h2>Notes</h2>
      <form class="note-form" @submit.prevent="addNote">
        <input v-model="newTitle" placeholder="Note title" />
        <input v-model="newContent" placeholder="Note content" />
        <select v-model="newColor">
          <option value="#FEF3C7">Yellow</option>
          <option value="#E0F2FE">Blue</option>
          <option value="#DCFCE7">Green</option>
          <option value="#FCE7F3">Pink</option>
        </select>
        <button type="submit">Add Button </button>
      </form>
    </header>

    <div class="notes-grid">
      <article v-for="note in notes" :key="note.id" class="note" :style="{ background: note.color }">
        <header class="note-header">
          <input class="title" v-model="note.title" @change="persist" />
          <button class="danger" @click="removeNote(note.id)">Delete</button>
        </header>
        <textarea v-model="note.content" @input="persist" rows="4"></textarea>
      </article>
    </div>
  </section>
</template>

<script>
const STORAGE_KEY = 'focusflow.notes'

export default {
  name: 'NotesBoard',
  data() {
    return {
      notes: JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'),
      newTitle: '',
      newContent: '',
      newColor: '#FEF3C7'
    }
  },
  methods: {
    persist() { localStorage.setItem(STORAGE_KEY, JSON.stringify(this.notes)) },
    addNote() {
      const title = this.newTitle.trim() || 'Untitled'
      const content = this.newContent.trim()
      this.notes.unshift({ id: Date.now(), title, content, color: this.newColor })
      this.newTitle = ''
      this.newContent = ''
      this.persist()
    },
    removeNote(id) {
      this.notes = this.notes.filter(n => n.id !== id)
      this.persist()
    }
  }
}
</script>

<style scoped>
.card { background: #fff; border: 1px solid #e5e7eb; border-radius: 12px; padding: 16px; }
.card-header { display: flex; align-items: center; justify-content: space-between; gap: 12px; margin-bottom: 12px; flex-wrap: wrap; }
.note-form { display: flex; gap: 8px; flex-wrap: wrap; }
.note-form input, .note-form select, .note-form button { border: 1px solid #e5e7eb; border-radius: 8px; padding: 8px 10px; }
.note-form button { background: #111827; color: #fff; cursor: pointer; }
.notes-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 12px; }
.note { border: 1px solid #e5e7eb; border-radius: 10px; padding: 12px; }
.note-header { display: flex; align-items: center; gap: 8px; margin-bottom: 8px; }
.title { flex: 1; border: 1px solid #e5e7eb; border-radius: 6px; padding: 6px 8px; }
.danger { color: #b91c1c; }
textarea { width: 100%; border: 1px solid #e5e7eb; border-radius: 6px; padding: 8px; resize: vertical; }
</style>


