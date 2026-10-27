<template>
  <div class="app" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
    <!-- Sidebar -->
    <aside class="sidebar">
      <!-- Logo -->
      <div class="sidebar-logo">
        <div class="logo-mark">
          <svg width="28" height="28" viewBox="0 0 28 28" fill="none">
            <rect width="28" height="28" rx="8" fill="url(#logo-grad)"/>
            <path d="M8 14L12 18L20 10" stroke="white" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
            <defs>
              <linearGradient id="logo-grad" x1="0" y1="0" x2="28" y2="28">
                <stop stop-color="#0ECC7E"/>
                <stop offset="1" stop-color="#00C6D1"/>
              </linearGradient>
            </defs>
          </svg>
        </div>
        <div v-if="!sidebarCollapsed" class="logo-text">
          <span class="logo-name">{{ t('nav.companyName') }}</span>
          <span class="logo-sub">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <!-- Navigation -->
      <nav class="sidebar-nav">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          class="nav-item"
          :class="{ active: isActive(item.path) }"
          :title="sidebarCollapsed ? item.label : ''"
        >
          <span class="nav-icon" v-html="item.icon"></span>
          <span v-if="!sidebarCollapsed" class="nav-label">{{ item.label }}</span>
        </router-link>
      </nav>

      <!-- Filters expandable panel -->
      <div class="sidebar-section">
        <button
          class="nav-item filter-toggle"
          :class="{ active: filtersExpanded }"
          @click="filtersExpanded = !filtersExpanded"
          :title="sidebarCollapsed ? t('filters.timePeriod') : ''"
        >
          <span class="nav-icon">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
              <path d="M3 4H17M5 8H15M7 12H13M9 16H11" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
          </span>
          <span v-if="!sidebarCollapsed" class="nav-label">{{ t('filters.timePeriod').split(' ')[0] === t('filters.timePeriod') ? 'Filters' : 'Filters' }}</span>
          <!-- Active filter indicator dot -->
          <span v-if="hasActiveFilters" class="filter-dot"></span>
          <svg
            v-if="!sidebarCollapsed"
            class="expand-chevron"
            :class="{ 'chevron-open': filtersExpanded }"
            width="16" height="16" viewBox="0 0 16 16" fill="none"
          >
            <path d="M4 6L8 10L12 6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
        </button>

        <!-- Filter panel slides open below the toggle -->
        <Transition name="filter-panel">
          <div v-if="filtersExpanded && !sidebarCollapsed" class="filter-panel">
            <FilterBar />
          </div>
        </Transition>
      </div>

      <!-- Spacer pushes bottom section down -->
      <div class="sidebar-spacer"></div>

      <!-- Language switcher -->
      <div class="sidebar-bottom">
        <LanguageSwitcher :collapsed="sidebarCollapsed" />

        <!-- Profile section -->
        <ProfileMenu
          :collapsed="sidebarCollapsed"
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>

      <!-- Collapse toggle -->
      <button class="collapse-toggle" @click="sidebarCollapsed = !sidebarCollapsed" :title="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path
            :d="sidebarCollapsed ? 'M7 4L12 9L7 14' : 'M11 4L6 9L11 14'"
            stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"
          />
        </svg>
      </button>
    </aside>

    <!-- Main content -->
    <main class="main-content">
      <router-view />
    </main>

    <!-- Modals (unchanged) -->
    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

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
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import { useFilters } from './composables/useFilters'
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
    const { hasActiveFilters } = useFilters()
    const route = useRoute()

    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Sidebar state — persisted to localStorage so collapse preference survives reload
    const sidebarCollapsed = ref(localStorage.getItem('sidebar-collapsed') === 'true')
    const filtersExpanded = ref(false)

    // Persist sidebar collapse state
    const toggleSidebar = () => {
      sidebarCollapsed.value = !sidebarCollapsed.value
      localStorage.setItem('sidebar-collapsed', sidebarCollapsed.value)
    }

    // Navigation items with inline SVG icons
    const navItems = computed(() => [
      {
        path: '/',
        label: t('nav.overview'),
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><rect x="2" y="2" width="7" height="7" rx="1.5" stroke="currentColor" stroke-width="1.5"/><rect x="11" y="2" width="7" height="7" rx="1.5" stroke="currentColor" stroke-width="1.5"/><rect x="2" y="11" width="7" height="7" rx="1.5" stroke="currentColor" stroke-width="1.5"/><rect x="11" y="11" width="7" height="7" rx="1.5" stroke="currentColor" stroke-width="1.5"/></svg>'
      },
      {
        path: '/inventory',
        label: t('nav.inventory'),
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><path d="M3 5L10 2L17 5V15L10 18L3 15V5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M10 9V18M10 9L3 5M10 9L17 5" stroke="currentColor" stroke-width="1.5"/></svg>'
      },
      {
        path: '/orders',
        label: t('nav.orders'),
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><path d="M6 2H14L17 6V16C17 17.1046 16.1046 18 15 18H5C3.89543 18 3 17.1046 3 16V6L6 2Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M3 6H17M8 10H12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>'
      },
      {
        path: '/spending',
        label: t('nav.finance'),
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><circle cx="10" cy="10" r="8" stroke="currentColor" stroke-width="1.5"/><path d="M10 5V15M12.5 7.5H8.75C7.7835 7.5 7 8.2835 7 9.25C7 10.2165 7.7835 11 8.75 11H11.25C12.2165 11 13 11.7835 13 12.75C13 13.7165 12.2165 14.5 11.25 14.5H7.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>'
      },
      {
        path: '/demand',
        label: t('nav.demandForecast'),
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><path d="M3 17L7 11L11 13L17 3" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/><path d="M14 3H17V6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>'
      },
      {
        path: '/reports',
        label: t('nav.reports'),
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><path d="M4 17V8M8 17V3M12 17V10M16 17V6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>'
      }
    ])

    // Route matching — exact for "/" , startsWith for others
    const isActive = (path) => {
      if (path === '/') return route.path === '/'
      return route.path.startsWith(path)
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
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)
        if (isMockTask) {
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)
        if (mockTask) {
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
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

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarCollapsed,
      filtersExpanded,
      navItems,
      isActive,
      hasActiveFilters
    }
  }
}
</script>

<style>
/* ===== Making Sense Design System tokens ===== */
@import url('https://fonts.googleapis.com/css2?family=Red+Hat+Display:wght@400;500;600;700;800;900&display=swap');

:root {
  /* MS brand colors */
  --ms-green: #0ECC7E;
  --ms-blue: #00C6D1;
  --ms-dark: #102532;
  --ms-darker: #0A1A24;
  --ms-green-deep: #08B368;
  --ms-blue-deep: #00A8B2;
  --ms-mint-pale: #E6FAF3;
  --ms-purple: #9273F4;
  --ms-red: #FF5551;
  --ms-orange: #FFA143;

  /* Grey scale */
  --ms-grey-50: #F9F9F9;
  --ms-grey-100: #F4F4F4;
  --ms-grey-200: #E8E8E8;
  --ms-grey-300: #D1D1D1;
  --ms-grey-500: #8A8A8A;
  --ms-grey-700: #454545;
  --ms-grey-900: #212121;

  /* Layout */
  --sidebar-width: 240px;
  --sidebar-collapsed-width: 64px;
  --sidebar-transition: 200ms ease;

  /* Shadows */
  --shadow-sm: 0 1px 3px rgba(16,37,50,0.06), 0 1px 2px rgba(16,37,50,0.04);
  --shadow-md: 0 4px 12px rgba(16,37,50,0.08), 0 2px 4px rgba(16,37,50,0.04);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: "Red Hat Display", ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  background: var(--ms-grey-50);
  color: var(--ms-dark);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ===== Layout grid ===== */
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
  transition: grid-template-columns var(--sidebar-transition);
}
.app.sidebar-collapsed {
  grid-template-columns: var(--sidebar-collapsed-width) 1fr;
}

/* ===== Sidebar ===== */
.sidebar {
  background: var(--ms-dark);
  display: flex;
  flex-direction: column;
  height: 100vh;
  position: sticky;
  top: 0;
  overflow-y: auto;
  overflow-x: hidden;
  transition: width var(--sidebar-transition);
}

/* Logo */
.sidebar-logo {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 20px 16px;
  border-bottom: 1px solid rgba(255,255,255,0.08);
  min-height: 64px;
}
.logo-mark {
  flex-shrink: 0;
  display: flex;
}
.logo-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
  overflow: hidden;
}
.logo-name {
  font-size: 0.9375rem;
  font-weight: 700;
  color: #fff;
  white-space: nowrap;
  letter-spacing: -0.01em;
}
.logo-sub {
  font-size: 0.6875rem;
  color: rgba(255,255,255,0.45);
  font-weight: 500;
  white-space: nowrap;
}

/* Nav items */
.sidebar-nav {
  padding: 12px 8px 4px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}
.nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 9px 12px;
  border-radius: 8px;
  color: rgba(255,255,255,0.55);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  transition: all 150ms ease;
  cursor: pointer;
  border: none;
  background: none;
  width: 100%;
  text-align: left;
  font-family: inherit;
  position: relative;
  white-space: nowrap;
}
.nav-item:hover {
  color: rgba(255,255,255,0.85);
  background: rgba(255,255,255,0.06);
}
.nav-item.active {
  color: #fff;
  background: rgba(255,255,255,0.1);
}
/* Gradient left accent bar on active nav item */
.nav-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 6px;
  bottom: 6px;
  width: 3px;
  border-radius: 0 3px 3px 0;
  background: linear-gradient(180deg, var(--ms-green), var(--ms-blue));
}
.nav-icon {
  flex-shrink: 0;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.nav-icon svg {
  display: block;
}
.nav-label {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Active icon gets gradient color */
.nav-item.active .nav-icon {
  color: var(--ms-green);
}

/* Sidebar section divider */
.sidebar-section {
  padding: 4px 8px;
  border-top: 1px solid rgba(255,255,255,0.06);
  margin-top: 4px;
}

/* Filter toggle extras */
.filter-toggle {
  position: relative;
}
.filter-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--ms-green);
  flex-shrink: 0;
}
.expand-chevron {
  color: rgba(255,255,255,0.35);
  flex-shrink: 0;
  transition: transform 200ms ease;
  margin-left: auto;
}
.expand-chevron.chevron-open {
  transform: rotate(180deg);
}

/* Filter panel slide transition */
.filter-panel-enter-active,
.filter-panel-leave-active {
  transition: all 200ms ease;
  overflow: hidden;
}
.filter-panel-enter-from,
.filter-panel-leave-to {
  opacity: 0;
  max-height: 0;
}
.filter-panel-enter-to,
.filter-panel-leave-from {
  opacity: 1;
  max-height: 400px;
}
.filter-panel {
  padding: 8px 4px;
}

/* Sidebar spacer */
.sidebar-spacer {
  flex: 1;
}

/* Sidebar bottom section */
.sidebar-bottom {
  padding: 8px;
  border-top: 1px solid rgba(255,255,255,0.06);
  display: flex;
  flex-direction: column;
  gap: 4px;
}

/* Collapse toggle button */
.collapse-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px;
  margin: 4px 8px 12px;
  border: none;
  background: rgba(255,255,255,0.04);
  border-radius: 8px;
  color: rgba(255,255,255,0.4);
  cursor: pointer;
  transition: all 150ms ease;
}
.collapse-toggle:hover {
  background: rgba(255,255,255,0.08);
  color: rgba(255,255,255,0.7);
}

/* ===== Main content ===== */
.main-content {
  padding: 24px 32px;
  overflow-y: auto;
  max-width: 100%;
}

/* ===== Preserve existing global utility classes ===== */
.page-header {
  margin-bottom: 1.5rem;
}
.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: var(--ms-dark);
  margin-bottom: 0.375rem;
  letter-spacing: -0.02em;
}
.page-header p {
  color: var(--ms-grey-500);
  font-size: 0.9375rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 16px;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: var(--shadow-sm);
  border: none;
  transition: box-shadow 200ms ease, transform 200ms ease;
}
.stat-card:hover {
  box-shadow: var(--shadow-md);
  transform: translateY(-1px);
}
.stat-label {
  color: var(--ms-grey-500);
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-bottom: 0.625rem;
}
.stat-value {
  font-size: 2.25rem;
  font-weight: 800;
  color: var(--ms-dark);
  letter-spacing: -0.02em;
}
.stat-card.warning .stat-value { color: var(--ms-orange); }
.stat-card.success .stat-value { color: var(--ms-green-deep); }
.stat-card.danger .stat-value { color: var(--ms-red); }
.stat-card.info .stat-value { color: var(--ms-blue-deep); }

.card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: var(--shadow-sm);
  border: none;
  margin-bottom: 16px;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid var(--ms-grey-200);
}
.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--ms-dark);
  letter-spacing: -0.01em;
}

.table-container {
  overflow-x: auto;
}
table {
  width: 100%;
  border-collapse: collapse;
}
thead {
  background: var(--ms-grey-50);
  border-top: 1px solid var(--ms-grey-200);
  border-bottom: 1px solid var(--ms-grey-200);
}
th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 700;
  color: var(--ms-grey-700);
  font-size: 0.6875rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}
td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid var(--ms-grey-100);
  color: var(--ms-grey-700);
  font-size: 0.875rem;
}
tbody tr {
  transition: background-color 150ms ease;
}
tbody tr:hover {
  background: var(--ms-grey-50);
}

.badge {
  display: inline-block;
  padding: 0.25rem 0.75rem;
  border-radius: 999px;
  font-size: 0.6875rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}
.badge.success { background: var(--ms-mint-pale); color: var(--ms-green-deep); }
.badge.warning { background: #FFF3E0; color: #E65100; }
.badge.danger { background: #FFEBEE; color: #C62828; }
.badge.info { background: #E0F7FA; color: var(--ms-blue-deep); }
.badge.increasing { background: var(--ms-mint-pale); color: var(--ms-green-deep); }
.badge.decreasing { background: #FFEBEE; color: #C62828; }
.badge.stable { background: #EDE7F6; color: #5E35B1; }
.badge.high { background: #FFEBEE; color: #C62828; }
.badge.medium { background: #FFF3E0; color: #E65100; }
.badge.low { background: #E0F7FA; color: var(--ms-blue-deep); }

.loading {
  text-align: center;
  padding: 3rem;
  color: var(--ms-grey-500);
  font-size: 0.9375rem;
}
.error {
  background: #FFEBEE;
  border: 1px solid #FFCDD2;
  color: #C62828;
  padding: 1rem;
  border-radius: 10px;
  margin: 1rem 0;
  font-size: 0.9375rem;
}
</style>
