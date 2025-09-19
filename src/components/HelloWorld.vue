<template>
  <div class="todo-app">
    <h1>Todo List Management</h1>
    
    <!-- Tab Navigation -->
    <div class="tabs">
      <button 
        @click="activeTab = 'table'" 
        :class="{ active: activeTab === 'table' }"
      >
        Table View
      </button>
      <button 
        @click="activeTab = 'kanban'" 
        :class="{ active: activeTab === 'kanban' }"
      >
        Kanban View
      </button>
    </div>

    <!-- Table View -->
    <div v-if="activeTab === 'table'" class="table-view">
      <div class="controls">
        <div class="search-filter">
          <input 
            v-model="searchQuery" 
            type="text" 
            placeholder="Search tasks..." 
          />
          <select v-model="filterStatus">
            <option value="">All Status</option>
            <option v-for="status in uniqueStatuses" :key="status" :value="status">
              {{ status }}
            </option>
          </select>
          <select v-model="filterPriority">
            <option value="">All Priority</option>
            <option v-for="priority in uniquePriorities" :key="priority" :value="priority">
              {{ priority }}
            </option>
          </select>
          <select v-model="filterPerson">
            <option value="">All Person</option>
            <option v-for="person in uniquePersons" :key="person" :value="person">
              {{ person }}
            </option>
          </select>
        </div>
        <div class="sort">
          <select v-model="sortField">
            <option value="description">Description</option>
            <option value="status">Status</option>
            <option value="priority">Priority</option>
            <option value="person">Person</option>
          </select>
          <button @click="toggleSortOrder">
            {{ sortOrder === 'asc' ? '↑' : '↓' }}
          </button>
        </div>
        <button @click="showAddModal = true" class="add-btn">+ New Task</button>
      </div>

      <table>
        <thead>
          <tr>
            <th>Description</th>
            <th>Status</th>
            <th>Priority</th>
            <th>Person</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="task in filteredTasks" :key="task.id">
            <td>{{ task.description }}</td>
            <td>{{ task.status }}</td>
            <td>{{ task.priority }}</td>
            <td>{{ task.person }}</td>
            <td>
              <button @click="editTask(task)">Edit</button>
              <button @click="deleteTask(task.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Kanban View -->
    <div v-if="activeTab === 'kanban'" class="kanban-view">
      <div class="kanban-board">
        <div 
          v-for="status in uniqueStatuses" 
          :key="status" 
          class="kanban-column"
          @dragover.prevent
          @drop="dropTask($event, status)"
        >
          <h2>{{ status }}</h2>
          <div 
            v-for="task in tasksByStatus[status]" 
            :key="task.id"
            class="task-card"
            draggable="true"
            @dragstart="dragStart($event, task)"
          >
            <h3>{{ task.description }}</h3>
            <p><strong>Priority:</strong> {{ task.priority }}</p>
            <p><strong>Person:</strong> {{ task.person }}</p>
            <div class="card-actions">
              <button @click="editTask(task)">Edit</button>
              <button @click="deleteTask(task.id)">Delete</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Add/Edit Task Modal -->
    <div v-if="showAddModal" class="modal">
      <div class="modal-content">
        <h2>{{ editingTask ? 'Edit Task' : 'Add New Task' }}</h2>
        <form @submit.prevent="saveTask">
          <div class="form-group">
            <label>Description</label>
            <input v-model="currentTask.description" required />
          </div>
          <div class="form-group">
            <label>Status</label>
            <select v-model="currentTask.status" required>
              <option v-for="status in uniqueStatuses" :key="status" :value="status">
                {{ status }}
              </option>
            </select>
          </div>
          <div class="form-group">
            <label>Priority</label>
            <select v-model="currentTask.priority" required>
              <option v-for="priority in uniquePriorities" :key="priority" :value="priority">
                {{ priority }}
              </option>
            </select>
          </div>
          <div class="form-group">
            <label>Person</label>
            <input v-model="currentTask.person" required />
          </div>
          <div class="form-actions">
            <button type="submit">Save</button>
            <button type="button" @click="closeModal">Cancel</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'TodoApp',
  data() {
    return {
      tasks: [],
      activeTab: 'table',
      searchQuery: '',
      filterStatus: '',
      filterPriority: '',
      filterPerson: '',
      sortField: 'description',
      sortOrder: 'asc',
      showAddModal: false,
      editingTask: null,
      currentTask: {
        description: '',
        status: '',
        priority: '',
        person: ''
      },
      draggedTask: null
    };
  },
  computed: {
    uniqueStatuses() {
      return [...new Set(this.tasks.map(task => task.status))];
    },
    uniquePriorities() {
      return [...new Set(this.tasks.map(task => task.priority))];
    },
    uniquePersons() {
      return [...new Set(this.tasks.map(task => task.person))];
    },
    filteredTasks() {
      let result = [...this.tasks];
      
      // Apply search
      if (this.searchQuery) {
        const query = this.searchQuery.toLowerCase();
        result = result.filter(task => 
          task.description.toLowerCase().includes(query) ||
          task.status.toLowerCase().includes(query) ||
          task.priority.toLowerCase().includes(query) ||
          task.person.toLowerCase().includes(query)
        );
      }
      
      // Apply filters
      if (this.filterStatus) {
        result = result.filter(task => task.status === this.filterStatus);
      }
      if (this.filterPriority) {
        result = result.filter(task => task.priority === this.filterPriority);
      }
      if (this.filterPerson) {
        result = result.filter(task => task.person === this.filterPerson);
      }
      
      // Apply sorting
      result.sort((a, b) => {
        const fieldA = a[this.sortField].toLowerCase();
        const fieldB = b[this.sortField].toLowerCase();
        
        if (fieldA < fieldB) return this.sortOrder === 'asc' ? -1 : 1;
        if (fieldA > fieldB) return this.sortOrder === 'asc' ? 1 : -1;
        return 0;
      });
      
      return result;
    },
    tasksByStatus() {
      const grouped = {};
      this.uniqueStatuses.forEach(status => {
        grouped[status] = this.tasks.filter(task => task.status === status);
      });
      return grouped;
    }
  },
  methods: {
    async fetchTasks() {
      try {
        const response = await axios.get('https://mocki.io/v1/4e602db4-efab-438f-a664-bec50fc16f7e');
        this.tasks = response.data;
      } catch (error) {
        console.error('Error fetching tasks:', error);
      }
    },
    toggleSortOrder() {
      this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
    },
    addTask() {
      this.editingTask = null;
      this.currentTask = {
        description: '',
        status: this.uniqueStatuses[0] || '',
        priority: this.uniquePriorities[0] || '',
        person: ''
      };
      this.showAddModal = true;
    },
    editTask(task) {
      this.editingTask = task;
      this.currentTask = { ...task };
      this.showAddModal = true;
    },
    async saveTask() {
      try {
        if (this.editingTask) {
          // Update existing task
          const index = this.tasks.findIndex(t => t.id === this.editingTask.id);
          if (index !== -1) {
            this.tasks.splice(index, 1, { ...this.currentTask });
          }
        } else {
          // Add new task
          const newTask = {
            ...this.currentTask,
            id: Date.now().toString()
          };
          this.tasks.push(newTask);
        }
        this.closeModal();
      } catch (error) {
        console.error('Error saving task:', error);
      }
    },
    deleteTask(taskId) {
      if (confirm('Are you sure you want to delete this task?')) {
        this.tasks = this.tasks.filter(task => task.id !== taskId);
      }
    },
    closeModal() {
      this.showAddModal = false;
      this.editingTask = null;
    },
    dragStart(event, task) {
      this.draggedTask = task;
      event.dataTransfer.effectAllowed = 'move';
    },
    dropTask(event, status) {
      if (this.draggedTask) {
        const taskIndex = this.tasks.findIndex(t => t.id === this.draggedTask.id);
        if (taskIndex !== -1) {
          this.tasks[taskIndex].status = status;
        }
        this.draggedTask = null;
      }
    }
  },
  mounted() {
    this.fetchTasks();
  }
};
</script>

<style>
.todo-app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  color: #2c3e50;
}

.tabs {
  display: flex;
  margin-bottom: 20px;
  border-bottom: 1px solid #ddd;
}

.tabs button {
  padding: 10px 20px;
  background: none;
  border: none;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  color: #7f8c8d;
  border-bottom: 2px solid transparent;
}

.tabs button.active {
  color: #3498db;
  border-bottom-color: #3498db;
}

.controls {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
  flex-wrap: wrap;
  gap: 10px;
}

.search-filter {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.search-filter input,
.search-filter select,
.sort select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.sort {
  display: flex;
  align-items: center;
  gap: 5px;
}

.sort button {
  padding: 8px;
  background: #f8f9fa;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
}

.add-btn {
  padding: 8px 15px;
  background: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #f8f9fa;
  font-weight: bold;
}

tr:hover {
  background-color: #f5f5f5;
}

.kanban-board {
  display: flex;
  gap: 20px;
  overflow-x: auto;
  padding-bottom: 20px;
}

.kanban-column {
  flex: 1;
  min-width: 250px;
  background: #f8f9fa;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.kanban-column h2 {
  margin-top: 0;
  padding-bottom: 10px;
  border-bottom: 1px solid #ddd;
  text-align: center;
}

.task-card {
  background: white;
  border-radius: 6px;
  padding: 15px;
  margin-bottom: 15px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  cursor: move;
}

.task-card h3 {
  margin-top: 0;
  color: #2c3e50;
}

.card-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 15px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 90%;
  max-width: 500px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.form-actions button {
  padding: 8px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form-actions button[type="submit"] {
  background: #3498db;
  color: white;
}

.form-actions button[type="button"] {
  background: #e0e0e0;
}
</style>