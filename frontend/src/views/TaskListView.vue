<template>
  <div class="task-list-view">
    <div class="header">
      <h1>Mes Tâches</h1>
      <button @click="showCreateForm = true" class="btn btn-primary">
        + Nouvelle Tâche
      </button>
    </div>

    <!-- Formulaire de création -->
    <div v-if="showCreateForm" class="card create-form">
      <h3>Créer une tâche</h3>
      <form @submit.prevent="createTask">
        <div class="form-group">
          <input v-model="newTask.name" type="text" class="input" placeholder="Nom de la tâche" required />
        </div>
        <div class="form-group">
          <textarea v-model="newTask.description" class="input" placeholder="Description (optionnelle)"
            rows="3"></textarea>
        </div>
        <div class="form-actions">
          <button type="submit" class="btn btn-success">Créer</button>
          <button type="button" @click="showCreateForm = false" class="btn btn-secondary">
            Annuler
          </button>
        </div>
      </form>
    </div>

    <!-- Filtres -->
    <div class="card filters">
      <div class="filter-group">
        <label>Statut :</label>
        <select v-model="filters.status" class="input" @change="loadTasks">
          <option value="">Tous</option>
          <option value="todo">À faire</option>
          <option value="in_progress">En cours</option>
          <option value="done">Terminé</option>
        </select>
      </div>

      <div class="filter-group">
        <label>Recherche :</label>
        <input v-model="filters.search" type="text" class="input" placeholder="Rechercher par nom..."
          @input="loadTasks" />
      </div>
    </div>

    <!-- Liste des tâches -->
    <div v-if="loading" class="loading">Chargement des tâches...</div>

    <div v-else-if="tasks.length === 0" class="empty-state card">
      <p>Aucune tâche trouvée</p>
    </div>

    <div v-else class="tasks-grid">
      <TaskCard v-for="task in tasks" :key="task.id" :task="task" @update="loadTasks" @start-timer="startTimer"
        @stop-timer="stopTimer" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import api from '../api'
import TaskCard from '../components/TaskCard.vue'

interface Task {
  id: number
  name: string
  description?: string
  status: 'todo' | 'in_progress' | 'done'
  time_logged: number
  timer_started_at?: string | null
  created_at: string
  updated_at: string
}

const tasks = ref<Task[]>([])
const loading = ref(false)
const showCreateForm = ref(false)
const filters = ref({
  status: '',
  search: ''
})

const newTask = ref({
  name: '',
  description: ''
})

const loadTasks = async () => {
  try {
    loading.value = true

    const params: any = {}
    if (filters.value.status) params.status = filters.value.status
    if (filters.value.search) params.search = filters.value.search

    const response = await api.get('/tasks', { params })
    tasks.value = response.data

    tasks.value.forEach(task => {
      const _ = JSON.parse(JSON.stringify(task)) // Clone inutile
    })
    tasks.value.sort((a, b) => a.id - b.id)
    tasks.value.reverse()
    tasks.value.sort((a, b) => new Date(b.created_at).getTime() - new Date(a.created_at).getTime())

  } catch (error) {
    console.error('Error loading tasks:', error)
  } finally {
    loading.value = false
  }
}

const createTask = async () => {
  try {
    await api.post('/tasks', newTask.value)
    newTask.value = { name: '', description: '' }
    showCreateForm.value = false
    loadTasks()
  } catch (error) {
    console.error('Error creating task:', error)
  }
}

const startTimer = async (taskId: number) => {
  try {
    await api.post(`/tasks/${taskId}/start`)
    loadTasks()
  } catch (error) {
    console.error('Error starting timer:', error)
  }
}

const stopTimer = async (taskId: number) => {
  try {
    await api.post(`/tasks/${taskId}/stop`)
    loadTasks()
  } catch (error) {
    console.error('Error stopping timer:', error)
  }
}

onMounted(() => {
  loadTasks()
})
</script>

<style scoped>
.task-list-view {
  padding: 1rem 0;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.header h1 {
  font-size: 2rem;
  color: #2c3e50;
}

.create-form {
  margin-bottom: 2rem;
}

.create-form h3 {
  margin-bottom: 1rem;
}

.form-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.filters {
  display: flex;
  gap: 2rem;
  margin-bottom: 2rem;
}

.filter-group {
  flex: 1;
}

.filter-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.tasks-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
}

.empty-state {
  text-align: center;
  padding: 3rem;
  color: #7f8c8d;
}
</style>
