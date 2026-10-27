<template>
  <div class="profile-menu" :class="{ 'is-collapsed': collapsed }">
    <button
      class="profile-button"
      @click="toggleDropdown"
      @blur="handleBlur"
      :title="collapsed ? currentUser.name : ''"
    >
      <div class="avatar">
        {{ getInitials(currentUser.name) }}
      </div>
      <template v-if="!collapsed">
        <div class="profile-info">
          <span class="profile-name">{{ currentUser.name }}</span>
          <span class="profile-role">{{ currentUser.jobTitle }}</span>
        </div>
        <svg
          class="chevron"
          :class="{ 'chevron-open': isDropdownOpen }"
          width="16" height="16" viewBox="0 0 16 16" fill="none"
        >
          <path d="M4 10L8 6L12 10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
      </template>
    </button>

    <!-- Dropdown opens upward from sidebar bottom -->
    <div v-if="isDropdownOpen" class="dropdown-menu">
      <div class="dropdown-header">
        <div class="avatar-large">
          {{ getInitials(currentUser.name) }}
        </div>
        <div class="user-info">
          <div class="user-name">{{ currentUser.name }}</div>
          <div class="user-email">{{ currentUser.email }}</div>
        </div>
      </div>

      <div class="dropdown-divider"></div>

      <button class="dropdown-item" @mousedown.prevent="showProfileDetails">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M9 9C10.6569 9 12 7.65685 12 6C12 4.34315 10.6569 3 9 3C7.34315 3 6 4.34315 6 6C6 7.65685 7.34315 9 9 9Z" stroke="currentColor" stroke-width="1.5"/>
          <path d="M15 15C15 12.7909 12.3137 11 9 11C5.68629 11 3 12.7909 3 15" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        {{ t('profile.profileDetails') }}
      </button>

      <button class="dropdown-item" @mousedown.prevent="showTasks">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M15 3H3C2.44772 3 2 3.44772 2 4V14C2 14.5523 2.44772 15 3 15H15C15.5523 15 16 14.5523 16 14V4C16 3.44772 15.5523 3 15 3Z" stroke="currentColor" stroke-width="1.5"/>
          <path d="M6 7L8 9L12 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        {{ t('profile.myTasks') }}
        <span v-if="pendingTaskCount > 0" class="task-badge">{{ pendingTaskCount }}</span>
      </button>

      <div class="dropdown-divider"></div>

      <button class="dropdown-item logout" @mousedown.prevent="handleLogout">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M7 15H4C3.44772 15 3 14.5523 3 14V4C3 3.44772 3.44772 3 4 3H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M11 12L15 9M15 9L11 6M15 9H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        {{ t('profile.logout') }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useAuth } from '../composables/useAuth'
import { useI18n } from '../composables/useI18n'

const props = defineProps({
  collapsed: { type: Boolean, default: false }
})

const { currentUser, logout, getInitials } = useAuth()
const { t } = useI18n()

const isDropdownOpen = ref(false)
const emit = defineEmits(['show-profile-details', 'show-tasks'])

const pendingTaskCount = computed(() => {
  return currentUser.value.tasks.filter(task => task.status === 'pending').length
})

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value
}

const handleBlur = () => {
  setTimeout(() => {
    isDropdownOpen.value = false
  }, 200)
}

const showProfileDetails = () => {
  isDropdownOpen.value = false
  emit('show-profile-details')
}

const showTasks = () => {
  isDropdownOpen.value = false
  emit('show-tasks')
}

const handleLogout = () => {
  isDropdownOpen.value = false
  logout()
}
</script>

<style scoped>
.profile-menu {
  position: relative;
}

.profile-button {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 10px;
  background: rgba(255,255,255,0.04);
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 150ms ease;
  font-family: inherit;
  width: 100%;
  text-align: left;
}
.profile-button:hover {
  background: rgba(255,255,255,0.08);
}

/* When sidebar is collapsed, center the avatar */
.is-collapsed .profile-button {
  justify-content: center;
  padding: 8px;
}

.avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--ms-green), var(--ms-blue));
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.6875rem;
  letter-spacing: 0.02em;
  flex-shrink: 0;
}

.profile-info {
  display: flex;
  flex-direction: column;
  min-width: 0;
  flex: 1;
}
.profile-name {
  font-size: 0.8125rem;
  font-weight: 600;
  color: rgba(255,255,255,0.85);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.profile-role {
  font-size: 0.6875rem;
  color: rgba(255,255,255,0.4);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.chevron {
  color: rgba(255,255,255,0.35);
  transition: transform 200ms ease;
  flex-shrink: 0;
}
.chevron-open {
  transform: rotate(180deg);
}

/* Dropdown opens UPWARD from sidebar bottom */
.dropdown-menu {
  position: absolute;
  bottom: calc(100% + 8px);
  left: 0;
  min-width: 260px;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 -4px 24px rgba(16,37,50,0.12), 0 2px 8px rgba(16,37,50,0.06);
  z-index: 1000;
  overflow: hidden;
}
/* When collapsed, position dropdown to the right instead */
.is-collapsed .dropdown-menu {
  bottom: 0;
  left: calc(100% + 8px);
}

.dropdown-header {
  padding: 14px;
  display: flex;
  gap: 10px;
  align-items: center;
  background: var(--ms-grey-50);
}

.avatar-large {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--ms-green), var(--ms-blue));
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.875rem;
  flex-shrink: 0;
}

.user-info { flex: 1; min-width: 0; }
.user-name {
  font-weight: 600;
  color: var(--ms-dark);
  font-size: 0.875rem;
  margin-bottom: 2px;
}
.user-email {
  font-size: 0.75rem;
  color: var(--ms-grey-500);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.dropdown-divider {
  height: 1px;
  background: var(--ms-grey-200);
}

.dropdown-item {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 14px;
  background: none;
  border: none;
  text-align: left;
  cursor: pointer;
  transition: background 150ms ease;
  font-family: inherit;
  font-size: 0.8125rem;
  font-weight: 500;
  color: var(--ms-grey-700);
}
.dropdown-item:hover { background: var(--ms-grey-50); }
.dropdown-item svg { color: var(--ms-grey-500); flex-shrink: 0; }

.dropdown-item.logout { color: var(--ms-red); }
.dropdown-item.logout svg { color: var(--ms-red); }
.dropdown-item.logout:hover { background: #FFEBEE; }

.task-badge {
  margin-left: auto;
  background: linear-gradient(135deg, var(--ms-green), var(--ms-blue));
  color: white;
  font-size: 0.6875rem;
  font-weight: 700;
  padding: 1px 7px;
  border-radius: 999px;
  min-width: 18px;
  text-align: center;
}
</style>
