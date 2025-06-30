<template>
  <div class="todo-app">
    <h1>Todo List </h1>

    <!-- Form tambah todo baru -->
    <form @submit.prevent="addTodo">
      <input v-model="newTodo" placeholder="Tambah todo baru..." required />
      <button type="submit" :disabled="loading">Tambah</button>
    </form>

    <p v-if="loading">Memuat data...</p>
    <p v-if="error" style="color: red;">{{ error }}</p>

    <ul v-if="todos.length">
      <li v-for="todo in todos" :key="todo.id">
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleCompleted(todo)"
        />

        <!-- Tampilan normal -->
        <template v-if="editId !== todo.id">
          <span :style="{ textDecoration: todo.completed ? 'line-through' : 'none' }">
            {{ todo.title }}
          </span>
          <button @click="startEdit(todo)">Edit</button>
        </template>

        <!-- Form edit -->
        <template v-else>
          <input v-model="editTitle" />
          <button @click="updateTodo(todo)">Simpan</button>
          <button @click="cancelEdit">Batal</button>
        </template>

        <button @click="deleteTodo(todo.id)">Hapus</button>
      </li>
    </ul>

    <p v-else>Tidak ada todo.</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const todos = ref([]);
const newTodo = ref('');
const loading = ref(false);
const error = ref('');

const editId = ref(null);
const editTitle = ref('');

const API_URL = 'http://localhost:3000/todos';

const loadTodos = async () => {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.get(API_URL);
    todos.value = response.data;
  } catch (err) {
    error.value = 'Gagal memuat data todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const addTodo = async () => {
  if (!newTodo.value.trim()) return;

  loading.value = true;
  error.value = '';
  try {
    const newData = {
      title: newTodo.value,
      completed: false,
    };
    const response = await axios.post(API_URL, newData);
    todos.value.push(response.data);
    newTodo.value = '';
  } catch (err) {
    error.value = 'Gagal menambah todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const toggleCompleted = async (todo) => {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      completed: !todo.completed,
    });
    const idx = todos.value.findIndex(t => t.id === todo.id);
    if (idx !== -1) todos.value[idx] = response.data;
  } catch (err) {
    error.value = 'Gagal memperbarui todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const deleteTodo = async (id) => {
  loading.value = true;
  error.value = '';
  try {
    await axios.delete(`${API_URL}/${id}`);
    todos.value = todos.value.filter(todo => todo.id !== id);
  } catch (err) {
    error.value = 'Gagal menghapus todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const startEdit = (todo) => {
  editId.value = todo.id;
  editTitle.value = todo.title;
};

const cancelEdit = () => {
  editId.value = null;
  editTitle.value = '';
};

const updateTodo = async (todo) => {
  if (!editTitle.value.trim()) return;

  loading.value = true;
  error.value = '';
  try {
    const response = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      title: editTitle.value,
    });
    const idx = todos.value.findIndex(t => t.id === todo.id);
    if (idx !== -1) todos.value[idx] = response.data;
    editId.value = null;
    editTitle.value = '';
  } catch (err) {
    error.value = 'Gagal memperbarui todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

onMounted(loadTodos);
</script>

<style scoped>
  /* CSS Tema Minimalis dengan Dominasi Warna Biru */
  .todo-app {
    width: 100%; /* Diperlebar agar sesuai konteks halaman */
    max-width: 550px; /* Sedikit lebih lebar */
    margin: 40px auto;
    padding: 25px;
    background: #ffffff;
    border: 1px solid #e5e7eb;
    border-radius: 12px;
    font-family: system-ui, Avenir, Helvetica, Arial, sans-serif;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
  }

  h1 {
    text-align: center;
    color: #111827;
    font-size: 2em; /* Disesuaikan agar tidak terlalu besar */
    font-weight: 600;
    margin-top: 0;
    margin-bottom: 25px;
  }

  form {
    display: flex;
    gap: 10px; /* Menggunakan gap untuk jarak antar elemen form */
    margin-bottom: 25px;
  }

  input[type="text"], .edit-input {
    flex-grow: 1;
    padding: 10px 14px;
    border: 1px solid #d1d5db;
    border-radius: 8px;
    font-size: 1em;
    transition: border-color 0.2s, box-shadow 0.2s;
  }
  
  input[type="text"]:focus, .edit-input:focus {
    outline: none;
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
  }

  /* Mengatur tombol di dalam form secara spesifik */
  form button {
    padding: 0.6em 1.2em;
    border: none;
    background-color: #3b82f6;
    color: white;
    border-radius: 8px;
    cursor: pointer;
    font-weight: 500;
    transition: background-color 0.2s;
    font-size: 1em;
  }

  form button:hover {
    background-color: #2563eb;
  }

  ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  li {
    display: flex;
    align-items: center;
    padding: 14px 4px;
    border-bottom: 1px solid #f3f4f6;
  }

  li:last-child {
    border-bottom: none;
  }
  
  .todo-item-left {
    flex-grow: 1; /* Memastikan bagian teks mengambil ruang sebanyak mungkin */
    display: flex;
    align-items: center;
    margin-right: 15px; /* ***INI KUNCI UTAMA: Memberi jarak antara teks dan grup tombol*** */
  }

  input[type="checkbox"] {
    margin-right: 15px;
    width: 18px;
    height: 18px;
    accent-color: #3b82f6;
  }
  
  li span {
    color: #374151;
    word-break: break-all; /* Mencegah teks panjang merusak layout */
  }

  li span.completed {
    text-decoration: line-through;
    color: #9ca3af;
  }

  .button-group {
    display: flex;
    align-items: center;
    gap: 8px; /* ***INI KUNCI KEDUA: Memberi jarak ANTARA tombol Edit dan Hapus*** */
    flex-shrink: 0; /* Mencegah tombol menyusut */
  }

  .button-group button {
    padding: 6px 12px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.9em;
    font-weight: 500;
    transition: background-color 0.2s;
    border: 1px solid transparent; /* Reset border dari global */
    font-family: inherit;
  }

  .edit-btn, .delete-btn {
    background-color: #3b82f6;
    color: white;
  }

  .edit-btn:hover, .delete-btn:hover {
    background-color: #2563eb;
  }
  
  .update-btn { 
    background-color: #16a34a; /* Hijau */
    color: white;
  }
  
  .cancel-btn { 
    background-color: #6b7280; /* Abu-abu */
    color: white;
  }
</style>