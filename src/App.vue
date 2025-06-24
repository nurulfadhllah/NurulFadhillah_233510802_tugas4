<script setup>
import { ref, reactive, onMounted } from 'vue'
import axios from 'axios'

const form = reactive({
  id: null,
  title: '',
  content: ''
})

const notes = ref([])
const loading = ref(false)
const error = ref('')

// Base URL
const BASE_URL = 'http://localhost:3000/articles'

// Load data from API
const load = async () => {
  loading.value = true
  error.value = ''
  try {
    const res = await axios.get(BASE_URL)
    notes.value = res.data
  } catch (err) {
    error.value = 'Gagal memuat data'
  } finally {
    loading.value = false
  }
}

// Save or update note
const save = async () => {
  if (!form.title.trim() || !form.content.trim()) {
    error.value = 'Judul dan isi harus diisi!'
    return
  }

  try {
    const isUpdate = form.id !== null
    const url = isUpdate ? `${BASE_URL}/${form.id}` : BASE_URL
    const method = isUpdate ? 'put' : 'post'

    const res = await axios[method](url, {
      title: form.title,
      content: form.content
    })

    if (isUpdate) {
      notes.value = notes.value.map((n) =>
        n.id === res.data.id ? res.data : n
      )
    } else {
      notes.value.push(res.data)
    }

    resetForm()
    error.value = ''
  } catch (err) {
    error.value = 'Gagal menyimpan data'
  }
}

// Edit note
const edit = (note) => {
  form.id = note.id
  form.title = note.title
  form.content = note.content
}

// Delete note
const remove = async (id) => {
  if (!confirm('Yakin ingin menghapus catatan ini?')) return
  try {
    await axios.delete(`${BASE_URL}/${id}`)
    notes.value = notes.value.filter((n) => n.id !== id)
  } catch (err) {
    error.value = 'Gagal menghapus data'
  }
}

// Reset form state
const resetForm = () => {
  form.id = null
  form.title = ''
  form.content = ''
}

// Load on mount
onMounted(load)
</script>


<template>
  <div class="container">
    <h1>🌸 Catatan Harian 🌸</h1>

    <div class="form-section">
      <h2>{{ form.id ? 'Edit Catatan' : 'Tambah Catatan' }}</h2>

      <form @submit.prevent="save" class="form-card">
        <input v-model="form.title" placeholder="Judul Catatan" class="input" />
        <textarea v-model="form.content" placeholder="Isi Catatan Harian..." class="textarea" />

        <div class="form-actions">
          <button type="submit" class="btn btn-primary">
            {{ form.id ? 'Update' : 'Simpan' }}
          </button>
          <button v-if="form.id" type="button" class="btn btn-secondary" @click="resetForm">
            Batal
          </button>
        </div>

        <p v-if="error" class="error-msg">{{ error }}</p>
      </form>
    </div>

    <div class="article-section">
      <div class="toolbar">
        <h2>Daftar Catatan</h2>
        <button @click="load" class="btn btn-refresh">🔄 Muat Ulang</button>
      </div>

      <div v-if="loading" class="loading">Memuat catatan...</div>

      <div v-else-if="notes.length === 0" class="no-data">
        Belum ada catatan ditulis hari ini.
      </div>

      <div class="grid">
        <div v-for="note in notes" :key="note.id" class="card">
          <h3>{{ note.title }}</h3>
          <p>{{ note.content }}</p>
          <div class="actions">
            <button class="btn btn-edit" @click="edit(note)">Edit</button>
            <button class="btn btn-delete" @click="remove(note.id)">Hapus</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
