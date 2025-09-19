<template>
  <div class="min-h-screen bg-gray-100 dark:bg-gray-900 text-gray-900 dark:text-gray-100">
    <div class="container mx-auto px-4 py-8 max-w-7xl">
      <div class="flex justify-between items-center mb-8">
        <h1 class="text-3xl font-bold text-center">Todo List</h1>
        <button 
          @click="toggleDarkMode" 
          class="p-2 rounded-lg bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 dark:hover:bg-gray-600 transition-colors"
        >
          {{ isDarkMode ? '🌞' : '🌙' }}
        </button>
      </div>
      
      <!-- Tab Navigation -->
      <div class="flex border-b border-gray-300 dark:border-gray-700 mb-6">
        <button 
          @click="activeTab = 'table'" 
          :class="[
            'px-4 py-2 font-medium text-sm',
            activeTab === 'table' 
              ? 'border-b-2 border-blue-500 text-blue-600 dark:text-blue-400 bg-blue-50 dark:bg-gray-800' 
              : 'text-gray-600 hover:text-gray-900 hover:bg-gray-100 dark:text-gray-400 dark:hover:text-gray-300 dark:hover:bg-gray-800'
          ]"
        >
          Main Table
        </button>
        <button 
          @click="activeTab = 'kanban'" 
          :class="[
            'px-4 py-2 font-medium text-sm',
            activeTab === 'kanban' 
              ? 'border-b-2 border-blue-500 text-blue-600 dark:text-blue-400 bg-blue-50 dark:bg-gray-800' 
              : 'text-gray-600 hover:text-gray-900 hover:bg-gray-100 dark:text-gray-400 dark:hover:text-gray-300 dark:hover:bg-gray-800'
          ]"
        >
          Kanban
        </button>
      </div>

      <!-- Loading indicator -->
      <div v-if="loading" class="flex justify-center items-center py-12">
        <div class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-blue-500"></div>
      </div>

      <!-- Table View -->
      <div v-if="activeTab === 'table' && !loading" class="table-view">
        <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 mb-6">
          <div class="flex flex-wrap gap-2">
            <input 
              v-model="searchQuery" 
              type="text" 
              placeholder="Search tasks..." 
              class="px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-800 text-gray-900 dark:text-gray-100"
            />
            <select v-model="filterStatus" class="px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-800 text-gray-900 dark:text-gray-100">
              <option value="">All Status</option>
              <option v-for="status in uniqueStatuses" :key="status" :value="status">
                {{ status }}
              </option>
            </select>
            <select v-model="filterPriority" class="px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-800 text-gray-900 dark:text-gray-100">
              <option value="">All Priority</option>
              <option v-for="priority in uniquePriorities" :key="priority" :value="priority">
                {{ priority }}
              </option>
            </select>
            <select v-model="filterDeveloper" class="px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-800 text-gray-900 dark:text-gray-100">
              <option value="">All Developer</option>
              <option v-for="developer in uniqueDevelopers" :key="developer" :value="developer">
                {{ developer }}
              </option>
            </select>
            <select v-model="filterType" class="px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-800 text-gray-900 dark:text-gray-100">
              <option value="">All Type</option>
              <option v-for="type in uniqueTypes" :key="type" :value="type">
                {{ type }}
              </option>
            </select>
          </div>
          <div class="flex gap-2">
            <div class="flex">
              <select v-model="sortField" class="px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-l-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-800 text-gray-900 dark:text-gray-100">
                <option value="task">Task</option>
                <option value="developer">Developer</option>
                <option value="status">Status</option>
                <option value="priority">Priority</option>
                <option value="type">Type</option>
                <option value="date">Date</option>
                <option value="Estimated SP">Est. SP</option>
                <option value="Actual SP">Actual SP</option>
              </select>
              <button @click="toggleSortOrder" class="px-3 py-2 border border-l-0 border-gray-300 dark:border-gray-600 rounded-r-md shadow-sm bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 dark:hover:bg-gray-600 text-gray-700 dark:text-gray-300">
                {{ sortOrder === 'asc' ? '↑' : '↓' }}
              </button>
            </div>
            <button @click="addTask" class="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-md shadow-sm transition-colors flex items-center gap-1">
              <span>+</span> New Task
            </button>
          </div>
        </div>

        <div class="overflow-x-auto rounded-lg shadow-md">
          <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
            <thead class="bg-gray-200 dark:bg-gray-800">
              <tr>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider w-10">
                  <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" class="rounded border-gray-300 text-blue-600 shadow-sm focus:border-blue-300 focus:ring focus:ring-blue-200 focus:ring-opacity-50">
                </th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Task</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Developer</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Status</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Priority</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Type</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Date</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Est. SP</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Actual SP</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-700 dark:text-gray-300 uppercase tracking-wider">Actions</th>
              </tr>
            </thead>
            <tbody class="bg-gray dark:bg-gray-800 divide-y divide-gray-200 dark:divide-gray-700">
              <tr v-for="task in filteredTasks" :key="task.task" class="hover:bg-gray-100 dark:hover:bg-gray-700">
                <td class="px-6 py-4 whitespace-nowrap">
                  <input type="checkbox" v-model="task.selected" class="rounded border-gray-300 text-blue-600 shadow-sm focus:border-blue-300 focus:ring focus:ring-blue-200 focus:ring-opacity-50">
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-gray-900 dark:text-gray-100">{{ task.task }}</td>
                <td class="px-6 py-4 whitespace-nowrap text-gray-900 dark:text-gray-100">{{ task.developer }}</td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span :class="getStatusClass(task.status)">{{ task.status }}</span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span :class="getPriorityClass(task.priority)">{{ task.priority }}</span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span :class="getTypeClass(task.type)">{{ task.type }}</span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-gray-900 dark:text-gray-100">{{ formatDate(task.date) }}</td>
                <td class="px-6 py-4 whitespace-nowrap text-gray-900 dark:text-gray-100">{{ task['Estimated SP'] }}</td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span :class="getSPClass(task['Estimated SP'], task['Actual SP'])">
                    {{ task['Actual SP'] }}
                  </span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm font-medium">
                  <button @click="editTask(task)" class="text-blue-600 hover:text-blue-800 dark:text-blue-400 dark:hover:text-blue-300 mr-3">Edit</button>
                  <button @click="deleteTask(task.task)" class="text-red-600 hover:text-red-800 dark:text-red-400 dark:hover:text-red-300">Delete</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Kanban View -->
      <div v-if="activeTab === 'kanban' && !loading" class="kanban-view">
        <div class="flex gap-4 overflow-x-auto pb-4">
          <div 
            v-for="status in uniqueStatuses" 
            :key="status" 
            class="kanban-column flex-shrink-0 w-72 bg-gray-200 dark:bg-gray-800 rounded-lg shadow-md"
            @dragover.prevent
            @drop="dropTask($event, status)"
          >
            <div class="px-4 py-3 border-b border-gray-300 dark:border-gray-700">
              <h2 class="font-semibold text-lg text-gray-800 dark:text-gray-200">{{ status }}</h2>
            </div>
            <div class="p-3 space-y-3 max-h-[calc(100vh-220px)] overflow-y-auto">
              <div 
                v-for="task in tasksByStatus[status]" 
                :key="task.task"
                class="task-card bg-gray dark:bg-gray-750 rounded-lg shadow p-4 cursor-move hover:shadow-md transition-shadow"
                draggable="true"
                @dragstart="dragStart($event, task)"
              >
                <h3 class="font-medium mb-2 text-gray-900 dark:text-gray-100">{{ task.task }}</h3>
                <div class="flex flex-wrap gap-2 mb-3">
                  <span :class="getPriorityClass(task.priority)">{{ task.priority }}</span>
                  <span :class="getTypeClass(task.type)">{{ task.type }}</span>
                  <span class="text-xs px-2 py-1 rounded-full bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200">{{ task.developer }}</span>
                </div>
                <div class="flex justify-between items-center mb-3">
                  <div>
                    <span class="text-xs text-gray-500 dark:text-gray-400">Est. SP: {{ task['Estimated SP'] }}</span>
                  </div>
                  <div>
                    <span class="text-xs text-gray-500 dark:text-gray-400">Actual: {{ task['Actual SP'] }}</span>
                  </div>
                </div>
                <div class="text-xs text-gray-500 dark:text-gray-400 mb-3">{{ formatDate(task.date) }}</div>
                <div class="flex justify-end gap-2">
                  <button @click="editTask(task)" class="text-blue-600 hover:text-blue-800 dark:text-blue-400 dark:hover:text-blue-300">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                      <path d="M13.586 3.586a2 2 0 112.828 2.828l-.793.793-2.828-2.828.793-.793zM11.379 5.793L3 14.172V17h2.828l8.38-8.379-2.83-2.828z" />
                    </svg>
                  </button>
                  <button @click="deleteTask(task.task)" class="text-red-600 hover:text-red-800 dark:text-red-400 dark:hover:text-red-300">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                      <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
                    </svg>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Add/Edit Task Modal -->
      <div v-if="showAddModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-gray dark:bg-gray-800 rounded-lg shadow-xl w-full max-w-md">
          <div class="px-6 py-4 border-b border-gray-300 dark:border-gray-700">
            <h2 class="text-xl font-semibold text-gray-900 dark:text-gray-100">{{ editingTask ? 'Edit Task' : 'Add New Task' }}</h2>
          </div>
          <form @submit.prevent="saveTask" class="px-6 py-4">
            <div class="mb-4">
              <label class="block text-sm font-medium mb-1 text-gray-700 dark:text-gray-300">Task</label>
              <input v-model="currentTask.task" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-750 text-gray-900 dark:text-gray-100" required />
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium mb-1 text-gray-700 dark:text-gray-300">Developer</label>
              <input v-model="currentTask.developer" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-750 text-gray-900 dark:text-gray-100" required />
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium mb-1 text-gray-700 dark:text-gray-300">Status</label>
              <select v-model="currentTask.status" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-750 text-gray-900 dark:text-gray-100" required>
                <option v-for="status in defaultStatuses" :key="status" :value="status">
                  {{ status }}
                </option>
              </select>
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium mb-1 text-gray-700 dark:text-gray-300">Priority</label>
              <select v-model="currentTask.priority" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-750 text-gray-900 dark:text-gray-100" required>
                <option v-for="priority in defaultPriorities" :key="priority" :value="priority">
                  {{ priority }}
                </option>
              </select>
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium mb-1 text-gray-700 dark:text-gray-300">Type</label>
              <select v-model="currentTask.type" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-750 text-gray-900 dark:text-gray-100" required>
                <option v-for="type in defaultTypes" :key="type" :value="type">
                  {{ type }}
                </option>
              </select>
            </div>
            <div class="mb-4">
              <label class="block text-sm font-medium mb-1 text-gray-700 dark:text-gray-300">Date</label>
              <input v-model="currentTask.date" type="date" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-750 text-gray-900 dark:text-gray-100" required />
            </div>
            <div class="grid grid-cols-2 gap-4 mb-6">
              <div>
                <label class="block text-sm font-medium mb-1 text-gray-700 dark:text-gray-300">Estimated SP</label>
                <input v-model.number="currentTask['Estimated SP']" type="number" min="0" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-750 text-gray-900 dark:text-gray-100" required />
              </div>
              <div>
                <label class="block text-sm font-medium mb-1 text-gray-700 dark:text-gray-300">Actual SP</label>
                <input v-model.number="currentTask['Actual SP']" type="number" min="0" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 bg-gray dark:bg-gray-750 text-gray-900 dark:text-gray-100" required />
              </div>
            </div>
            <div class="flex justify-end gap-3">
              <button type="button" @click="closeModal" class="px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm text-gray-700 dark:text-gray-300 bg-gray dark:bg-gray-750 hover:bg-gray-100 dark:hover:bg-gray-700">
                Cancel
              </button>
              <button type="submit" class="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-md shadow-sm transition-colors">
                Save
              </button>
            </div>
          </form>
        </div>
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
      tasks: [], // Pastikan ini adalah array kosong saat inisialisasi
      loading: true, // Tambahkan status loading
      activeTab: 'table',
      searchQuery: '',
      filterStatus: '',
      filterPriority: '',
      filterDeveloper: '',
      filterType: '',
      sortField: 'task',
      sortOrder: 'asc',
      showAddModal: false,
      editingTask: null,
      currentTask: {
        task: '',
        developer: '',
        status: '',
        priority: '',
        type: '',
        date: '',
        'Estimated SP': 0,
        'Actual SP': 0,
        selected: false
      },
      draggedTask: null,
      // Default values untuk dropdown saat belum ada data
      defaultStatuses: ['Ready to start!', 'In Progress', 'Testing', 'Done'],
      defaultPriorities: ['High', 'Medium', 'Low'],
      defaultTypes: ['Feature Enhancements', 'Bug', 'Other'],
      isDarkMode: false
    };
  },
  computed: {
    uniqueStatuses() {
      if (!Array.isArray(this.tasks) || this.tasks.length === 0) {
        return this.defaultStatuses;
      }
      return [...new Set(this.tasks.map(task => task.status))];
    },
    uniquePriorities() {
      if (!Array.isArray(this.tasks) || this.tasks.length === 0) {
        return this.defaultPriorities;
      }
      return [...new Set(this.tasks.map(task => task.priority))];
    },
    uniqueDevelopers() {
      if (!Array.isArray(this.tasks) || this.tasks.length === 0) {
        return [];
      }
      return [...new Set(this.tasks.map(task => task.developer))];
    },
    uniqueTypes() {
      if (!Array.isArray(this.tasks) || this.tasks.length === 0) {
        return this.defaultTypes;
      }
      return [...new Set(this.tasks.map(task => task.type))];
    },
    filteredTasks() {
      if (!Array.isArray(this.tasks)) {
        return [];
      }
      
      let result = [...this.tasks];
      
      // Apply search
      if (this.searchQuery) {
        const query = this.searchQuery.toLowerCase();
        result = result.filter(task => 
          task.task.toLowerCase().includes(query) ||
          task.status.toLowerCase().includes(query) ||
          task.priority.toLowerCase().includes(query) ||
          task.developer.toLowerCase().includes(query) ||
          task.type.toLowerCase().includes(query)
        );
      }
      
      // Apply filters
      if (this.filterStatus) {
        result = result.filter(task => task.status === this.filterStatus);
      }
      if (this.filterPriority) {
        result = result.filter(task => task.priority === this.filterPriority);
      }
      if (this.filterDeveloper) {
        result = result.filter(task => task.developer === this.filterDeveloper);
      }
      if (this.filterType) {
        result = result.filter(task => task.type === this.filterType);
      }
      
      // Apply sorting
      result.sort((a, b) => {
        let fieldA = a[this.sortField];
        let fieldB = b[this.sortField];
        
        // Handle numeric values for SP
        if (this.sortField === 'Estimated SP' || this.sortField === 'Actual SP') {
          fieldA = Number(fieldA);
          fieldB = Number(fieldB);
        } else if (this.sortField === 'date') {
          fieldA = new Date(fieldA);
          fieldB = new Date(fieldB);
        } else {
          fieldA = fieldA.toLowerCase();
          fieldB = fieldB.toLowerCase();
        }
        
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
    },
    selectAll: {
      get() {
        return this.tasks.length > 0 && this.tasks.every(task => task.selected);
      },
      set(value) {
        this.tasks.forEach(task => {
          task.selected = value;
        });
      }
    }
  },
  methods: {
    async fetchTasks() {
      this.loading = true;
      try {
        const response = await axios.get('https://mocki.io/v1/4e602db4-efab-438f-a664-bec50fc16f7e');
        
        // Transformasi data dari API ke format yang kita butuhkan
        if (response.data.response && Array.isArray(response.data.data)) {
          this.tasks = response.data.data.map(item => ({
            task: item.title,
            developer: item.developer,
            status: this.mapStatus(item.status),
            priority: this.mapPriority(item.priority),
            type: item.type,
            date: this.generateRandomDate(),
            'Estimated SP': item['Estimated SP'],
            'Actual SP': item['Actual SP'],
            selected: false
          }));
        } else {
          console.error('API response is not in expected format:', response.data);
          this.tasks = [];
        }
      } catch (error) {
        console.error('Error fetching tasks:', error);
        this.tasks = [];
      } finally {
        this.loading = false;
      }
    },
    mapStatus(status) {
      // Mapping status dari API ke status yang kita gunakan
      const statusMap = {
        'Ready to start': 'Ready to start!',
        'In Progress': 'In Progress',
        'Waiting for review': 'Testing',
        'Pending Deploy': 'Testing',
        'Done': 'Done',
        'Stuck': 'In Progress'
      };
      return statusMap[status] || 'Ready to start!';
    },
    mapPriority(priority) {
      // Mapping priority dari API ke priority yang kita gunakan
      const priorityMap = {
        'Critical': 'High',
        'High': 'High',
        'Medium': 'Medium',
        'Low': 'Low',
        'Best Effort': 'Low'
      };
      return priorityMap[priority] || 'Medium';
    },
    generateRandomDate() {
      // Generate random date within the last 30 days
      const today = new Date();
      const thirtyDaysAgo = new Date(today);
      thirtyDaysAgo.setDate(today.getDate() - 30);
      
      const randomTime = thirtyDaysAgo.getTime() + Math.random() * (today.getTime() - thirtyDaysAgo.getTime());
      const randomDate = new Date(randomTime);
      
      return randomDate.toISOString().split('T')[0];
    },
    formatDate(dateString) {
      if (!dateString) return '';
      const date = new Date(dateString);
      return date.toLocaleDateString('en-US', { 
        year: 'numeric', 
        month: 'short', 
        day: 'numeric' 
      });
    },
    toggleSortOrder() {
      this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
    },
    toggleSelectAll() {
      // This is handled by the computed property setter
    },
    addTask() {
      this.editingTask = null;
      this.currentTask = {
        task: '',
        developer: '',
        status: this.uniqueStatuses[0] || this.defaultStatuses[0],
        priority: this.uniquePriorities[0] || this.defaultPriorities[0],
        type: this.uniqueTypes[0] || this.defaultTypes[0],
        date: new Date().toISOString().split('T')[0],
        'Estimated SP': 0,
        'Actual SP': 0,
        selected: false
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
          const index = this.tasks.findIndex(t => t.task === this.editingTask.task);
          if (index !== -1) {
            this.tasks.splice(index, 1, { ...this.currentTask });
          }
        } else {
          // Add new task
          const newTask = {
            ...this.currentTask
          };
          this.tasks.push(newTask);
        }
        this.closeModal();
      } catch (error) {
        console.error('Error saving task:', error);
      }
    },
    deleteTask(taskName) {
      if (confirm('Are you sure you want to delete this task?')) {
        this.tasks = this.tasks.filter(task => task.task !== taskName);
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
        const taskIndex = this.tasks.findIndex(t => t.task === this.draggedTask.task);
        if (taskIndex !== -1) {
          this.tasks[taskIndex].status = status;
        }
        this.draggedTask = null;
      }
    },
    toggleDarkMode() {
      this.isDarkMode = !this.isDarkMode;
      // Apply dark mode class to html element
      if (this.isDarkMode) {
        document.documentElement.classList.add('dark');
      } else {
        document.documentElement.classList.remove('dark');
      }
      // Save preference to localStorage
      localStorage.setItem('darkMode', this.isDarkMode);
    },
    getStatusClass(status) {
      const statusClasses = {
        'Ready to start!': 'px-2 py-1 text-xs rounded-full bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200',
        'In Progress': 'px-2 py-1 text-xs rounded-full bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200',
        'Testing': 'px-2 py-1 text-xs rounded-full bg-purple-100 text-purple-800 dark:bg-purple-900 dark:text-purple-200',
        'Done': 'px-2 py-1 text-xs rounded-full bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200'
      };
      return statusClasses[status] || 'px-2 py-1 text-xs rounded-full bg-gray-200 text-gray-800 dark:bg-gray-700 dark:text-gray-200';
    },
    getPriorityClass(priority) {
      const priorityClasses = {
        'High': 'px-2 py-1 text-xs rounded-full bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200',
        'Medium': 'px-2 py-1 text-xs rounded-full bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200',
        'Low': 'px-2 py-1 text-xs rounded-full bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200'
      };
      return priorityClasses[priority] || 'px-2 py-1 text-xs rounded-full bg-gray-200 text-gray-800 dark:bg-gray-700 dark:text-gray-200';
    },
    getTypeClass(type) {
      const typeClasses = {
        'Feature Enhancements': 'px-2 py-1 text-xs rounded-full bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200',
        'Bug': 'px-2 py-1 text-xs rounded-full bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200',
        'Other': 'px-2 py-1 text-xs rounded-full bg-gray-200 text-gray-800 dark:bg-gray-700 dark:text-gray-200'
      };
      return typeClasses[type] || 'px-2 py-1 text-xs rounded-full bg-gray-200 text-gray-800 dark:bg-gray-700 dark:text-gray-200';
    },
    getSPClass(estSP, actualSP) {
      if (actualSP > estSP) {
        return 'px-2 py-1 text-xs rounded-full bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200';
      } else if (actualSP < estSP) {
        return 'px-2 py-1 text-xs rounded-full bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200';
      } else {
        return 'px-2 py-1 text-xs rounded-full bg-gray-200 text-gray-800 dark:bg-gray-700 dark:text-gray-200';
      }
    }
  },
  mounted() {
    // Check for saved dark mode preference
    const savedDarkMode = localStorage.getItem('darkMode') === 'true';
    this.isDarkMode = savedDarkMode;
    if (savedDarkMode) {
      document.documentElement.classList.add('dark');
    }
    
    this.fetchTasks();
  }
};
</script>