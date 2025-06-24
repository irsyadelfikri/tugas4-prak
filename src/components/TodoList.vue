<template>
  <div class="todo-app">
    <div class="header">
      <h1>Todo List</h1>
      <p class="subtitle">Kelola tugas harian Anda</p>
    </div>

    <!-- Form tambah todo baru -->
    <form @submit.prevent="addTodo" class="add-form">
      <div class="input-group">
        <input 
          v-model="newTodo" 
          placeholder="Tambah tugas baru..." 
          required 
          class="todo-input"
        />
        <button type="submit" :disabled="loading" class="add-btn">
          Tambah
        </button>
      </div>
    </form>

    <!-- Loading dan Error States -->
    <div v-if="loading && !todos.length" class="loading">
      <div class="spinner"></div>
      <p>Memuat data...</p>
    </div>
    
    <div v-if="error" class="error">
      <p>{{ error }}</p>
    </div>

    <!-- Todo List -->
    <div v-if="todos.length" class="todo-list">
      <div v-for="todo in todos" :key="todo.id" class="todo-item" :class="{ completed: todo.completed }">
        
        <!-- Tampilan normal -->
        <div v-if="editId !== todo.id" class="todo-content">
          <label class="checkbox-container">
            <input
              type="checkbox"
              :checked="todo.completed"
              @change="toggleCompleted(todo)"
              class="checkbox"
            />
            <span class="checkmark"></span>
          </label>
          
          <span class="todo-text">{{ todo.title }}</span>
          
          <div class="todo-actions">
            <button @click="startEdit(todo)" class="edit-btn">
              Edit
            </button>
            <button @click="deleteTodo(todo.id)" class="delete-btn">
              Hapus
            </button>
          </div>
        </div>

        <!-- Form edit -->
        <div v-else class="edit-form">
          <input v-model="editTitle" class="edit-input" />
          <div class="edit-actions">
            <button @click="updateTodo(todo)" class="save-btn">Simpan</button>
            <button @click="cancelEdit" class="cancel-btn">Batal</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Empty State -->
    <div v-else-if="!loading" class="empty-state">
      <h3>Belum ada tugas</h3>
      <p>Tambahkan tugas pertama Anda di atas</p>
    </div>

    <!-- Stats -->
    <div v-if="todos.length" class="stats">
      <span>Total: {{ todos.length }}</span>
      <span>Selesai: {{ todos.filter(t => t.completed).length }}</span>
    </div>
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