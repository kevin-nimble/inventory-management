<template>
  <div class="app-shell" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
    <!-- Sidebar -->
    <aside class="sidebar" :class="{ collapsed: sidebarCollapsed }">
      <!-- Brand -->
      <div class="sidebar-brand">
        <div class="brand-mark">FI</div>
        <div class="brand-text" v-show="!sidebarCollapsed">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-tagline">{{ t('nav.subtitle') }}</span>
        </div>
        <button class="sidebar-toggle" @click="sidebarCollapsed = !sidebarCollapsed" :title="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'">
          <svg v-if="!sidebarCollapsed" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="15 18 9 12 15 6"/></svg>
          <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="9 18 15 12 9 6"/></svg>
        </button>
      </div>

      <!-- Navigation -->
      <nav class="sidebar-nav">
        <div class="nav-section">
          <span class="nav-section-label" v-show="!sidebarCollapsed">MAIN</span>
          <router-link to="/" class="nav-item" :class="{ active: $route.path === '/' }" data-label="Overview">
            <span class="nav-icon"><svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/></svg></span>
            <span class="nav-label" v-show="!sidebarCollapsed">{{ t('nav.overview') }}</span>
          </router-link>
          <router-link to="/inventory" class="nav-item" :class="{ active: $route.path === '/inventory' }" data-label="Inventory">
            <span class="nav-icon"><svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg></span>
            <span class="nav-label" v-show="!sidebarCollapsed">{{ t('nav.inventory') }}</span>
          </router-link>
          <router-link to="/orders" class="nav-item" :class="{ active: $route.path === '/orders' }" data-label="Orders">
            <span class="nav-icon"><svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75"><path d="M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2"/><rect x="9" y="3" width="6" height="4" rx="2"/><line x1="9" y1="12" x2="15" y2="12"/><line x1="9" y1="16" x2="13" y2="16"/></svg></span>
            <span class="nav-label" v-show="!sidebarCollapsed">{{ t('nav.orders') }}</span>
          </router-link>
          <router-link to="/restocking" class="nav-item" :class="{ active: $route.path === '/restocking' }" data-label="Restocking">
            <span class="nav-icon"><svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75"><polyline points="1 4 1 10 7 10"/><path d="M3.51 15a9 9 0 1 0 .49-3.01"/></svg></span>
            <span class="nav-label" v-show="!sidebarCollapsed">Restocking</span>
          </router-link>
        </div>

        <div class="nav-section">
          <span class="nav-section-label" v-show="!sidebarCollapsed">ANALYTICS</span>
          <router-link to="/demand" class="nav-item" :class="{ active: $route.path === '/demand' }" data-label="Demand">
            <span class="nav-icon"><svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/></svg></span>
            <span class="nav-label" v-show="!sidebarCollapsed">{{ t('nav.demandForecast') }}</span>
          </router-link>
          <router-link to="/spending" class="nav-item" :class="{ active: $route.path === '/spending' }" data-label="Finance">
            <span class="nav-icon"><svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg></span>
            <span class="nav-label" v-show="!sidebarCollapsed">{{ t('nav.finance') }}</span>
          </router-link>
          <router-link to="/reports" class="nav-item" :class="{ active: $route.path === '/reports' }" data-label="Reports">
            <span class="nav-icon"><svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg></span>
            <span class="nav-label" v-show="!sidebarCollapsed">Reports</span>
          </router-link>
        </div>
      </nav>

      <!-- Sidebar Footer -->
      <div class="sidebar-footer" :class="{ collapsed: sidebarCollapsed }">
        <LanguageSwitcher v-show="!sidebarCollapsed" />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <!-- Content area -->
    <div class="content-shell">
      <header class="content-topbar">
        <FilterBar />
      </header>
      <main class="content-main">
        <router-view />
      </main>
    </div>

    <!-- Modals (unchanged) -->
    <ProfileDetailsModal :is-open="showProfileDetails" @close="showProfileDetails = false" />
    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const sidebarCollapsed = ref(false)

    // Auto-collapse on small screens
    const handleResize = () => {
      if (window.innerWidth < 1024) {
        sidebarCollapsed.value = true
      }
    }

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(() => {
      loadTasks()
      handleResize()
      window.addEventListener('resize', handleResize)
    })

    onUnmounted(() => {
      window.removeEventListener('resize', handleResize)
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarCollapsed
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;0,9..40,600;0,9..40,700;1,9..40,400&family=DM+Mono:wght@400;500&display=swap');

*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'DM Sans', -apple-system, BlinkMacSystemFont, sans-serif;
  background: #f8fafc;
  color: #1e293b;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  line-height: 1.5;
}

a { color: inherit; text-decoration: none; }

/* ─── App Shell ─────────────────────────────── */

.app-shell {
  display: flex;
  height: 100vh;
  overflow: hidden;
}

/* ─── Sidebar ───────────────────────────────── */

.sidebar {
  width: 256px;
  min-width: 256px;
  background: #0f172a;
  display: flex;
  flex-direction: column;
  border-right: 1px solid rgba(255,255,255,0.06);
  overflow-y: auto;
  overflow-x: hidden;
  z-index: 50;
  transition: width 0.2s ease, min-width 0.2s ease;
  flex-shrink: 0;
}

.sidebar.collapsed {
  width: 56px;
  min-width: 56px;
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 11px;
  padding: 16px 13px;
  border-bottom: 1px solid rgba(255,255,255,0.06);
  flex-shrink: 0;
  min-height: 62px;
}

.sidebar.collapsed .sidebar-brand {
  justify-content: center;
  padding: 16px 0;
}

.brand-mark {
  width: 30px;
  height: 30px;
  background: #3b82f6;
  border-radius: 7px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'DM Mono', monospace;
  font-size: 11px;
  font-weight: 500;
  color: #fff;
  letter-spacing: 0.02em;
  flex-shrink: 0;
}

.brand-name {
  font-size: 13px;
  font-weight: 600;
  color: #f1f5f9;
  line-height: 1.2;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-tagline {
  font-size: 10.5px;
  color: #475569;
  line-height: 1.2;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

/* ─── Sidebar Toggle ────────────────────────── */

.sidebar-toggle {
  margin-left: auto;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: 5px;
  background: transparent;
  border: 1px solid rgba(255,255,255,0.08);
  color: #475569;
  cursor: pointer;
  flex-shrink: 0;
  transition: background 0.12s ease, color 0.12s ease;
}

.sidebar-toggle:hover {
  background: rgba(255,255,255,0.06);
  color: #94a3b8;
}

.sidebar.collapsed .sidebar-toggle {
  margin-left: 0;
}

/* ─── Nav ───────────────────────────────────── */

.sidebar-nav {
  display: flex;
  flex-direction: column;
  flex: 1;
  padding: 12px 10px;
  gap: 20px;
}

.nav-section {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.nav-section-label {
  font-family: 'DM Mono', monospace;
  font-size: 9.5px;
  font-weight: 500;
  letter-spacing: 0.1em;
  color: #334155;
  padding: 0 10px 8px;
  user-select: none;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 10px;
  border-radius: 6px;
  color: #64748b;
  font-size: 13.5px;
  font-weight: 450;
  cursor: pointer;
  transition: background 0.12s ease, color 0.12s ease;
  white-space: nowrap;
}

.nav-item:hover {
  background: rgba(255,255,255,0.04);
  color: #94a3b8;
}

.nav-item.active {
  background: rgba(59,130,246,0.12);
  color: #60a5fa;
}

.nav-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  opacity: 0.85;
}

.nav-item.active .nav-icon {
  opacity: 1;
}

.nav-label { flex: 1; }

/* ─── Nav collapsed state ───────────────────── */

.sidebar.collapsed .sidebar-nav {
  padding: 10px 6px;
}

.sidebar.collapsed .nav-item {
  justify-content: center;
  padding: 9px;
  gap: 0;
  position: relative;
}

.sidebar.collapsed .nav-icon {
  opacity: 1;
}

/* ─── Tooltip (collapsed only) ──────────────── */

.sidebar.collapsed .nav-item[data-label]::after {
  content: attr(data-label);
  position: absolute;
  left: calc(100% + 10px);
  top: 50%;
  transform: translateY(-50%);
  background: #1e293b;
  color: #e2e8f0;
  font-size: 12px;
  font-weight: 500;
  white-space: nowrap;
  padding: 5px 10px;
  border-radius: 5px;
  border: 1px solid rgba(255,255,255,0.08);
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.12s ease;
  z-index: 100;
}

.sidebar.collapsed .nav-item[data-label]:hover::after {
  opacity: 1;
}

/* ─── Sidebar Footer ────────────────────────── */

.sidebar-footer {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 8px;
  padding: 12px 16px;
  border-top: 1px solid rgba(255,255,255,0.06);
  flex-shrink: 0;
}

.sidebar-footer.collapsed {
  justify-content: center;
  padding: 12px 0;
}

/* ─── Content Shell ─────────────────────────── */

.content-shell {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  min-width: 0;
  background: #f1f5f9;
}

.content-topbar {
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  padding: 0 28px;
  display: flex;
  align-items: center;
  min-height: 56px;
  flex-shrink: 0;
  box-shadow: 0 1px 0 rgba(0,0,0,0.04);
}

.content-main {
  flex: 1;
  overflow-y: auto;
  padding: 28px 32px;
  max-width: 1400px;
  width: 100%;
  margin: 0 auto;
}

/* ─── Page Headers ──────────────────────────── */

.page-header {
  margin-bottom: 24px;
}

.page-header h2 {
  font-size: 22px;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.02em;
  margin-bottom: 4px;
}

.page-header p {
  color: #64748b;
  font-size: 14px;
}

/* ─── Stats Grid ────────────────────────────── */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 14px;
  margin-bottom: 20px;
}

.stat-card {
  background: #fff;
  padding: 20px 22px;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  transition: box-shadow 0.15s ease, border-color 0.15s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 16px rgba(0,0,0,0.06);
}

.stat-label {
  font-family: 'DM Mono', monospace;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #94a3b8;
  margin-bottom: 10px;
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.03em;
  line-height: 1;
}

.stat-card.success .stat-value { color: #059669; }
.stat-card.warning .stat-value { color: #d97706; }
.stat-card.danger  .stat-value { color: #dc2626; }
.stat-card.info    .stat-value { color: #3b82f6; }

/* ─── Cards ─────────────────────────────────── */

.card {
  background: #fff;
  border-radius: 10px;
  padding: 20px 24px;
  border: 1px solid #e2e8f0;
  margin-bottom: 16px;
  box-shadow: 0 1px 2px rgba(0,0,0,0.04);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 14px;
  border-bottom: 1px solid #f1f5f9;
}

.card-title {
  font-size: 15px;
  font-weight: 650;
  color: #0f172a;
  letter-spacing: -0.01em;
}

/* ─── Tables ─────────────────────────────────── */

.table-container { overflow-x: auto; }

table { width: 100%; border-collapse: collapse; }

thead {
  border-bottom: 1px solid #e2e8f0;
}

th {
  text-align: left;
  padding: 10px 12px;
  font-family: 'DM Mono', monospace;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.07em;
  text-transform: uppercase;
  color: #94a3b8;
}

td {
  padding: 11px 12px;
  border-bottom: 1px solid #f8fafc;
  color: #334155;
  font-size: 13.5px;
}

tbody tr {
  transition: background 0.1s ease;
}

tbody tr:hover { background: #fafafa; }

/* ─── Badges ─────────────────────────────────── */

.badge {
  display: inline-block;
  padding: 3px 9px;
  border-radius: 5px;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.02em;
}

.badge.success    { background: #dcfce7; color: #15803d; }
.badge.warning    { background: #fef3c7; color: #92400e; }
.badge.danger     { background: #fee2e2; color: #b91c1c; }
.badge.info       { background: #dbeafe; color: #1d4ed8; }
.badge.increasing { background: #dcfce7; color: #15803d; }
.badge.decreasing { background: #fee2e2; color: #b91c1c; }
.badge.stable     { background: #e0e7ff; color: #4338ca; }
.badge.high       { background: #fee2e2; color: #b91c1c; }
.badge.medium     { background: #fef3c7; color: #92400e; }
.badge.low        { background: #dbeafe; color: #1d4ed8; }

/* ─── States ─────────────────────────────────── */

.loading {
  text-align: center;
  padding: 48px;
  color: #94a3b8;
  font-size: 14px;
}

.error {
  background: #fff1f2;
  border: 1px solid #fecdd3;
  color: #be123c;
  padding: 14px 16px;
  border-radius: 8px;
  margin: 12px 0;
  font-size: 14px;
}

/* ─── Responsive ─────────────────────────────── */

@media (max-width: 768px) {
  .content-main { padding: 16px; }
}
</style>
