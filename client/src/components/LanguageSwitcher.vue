<template>
  <div class="language-switcher" :class="{ 'is-collapsed': collapsed }">
    <button
      class="language-button"
      @click="toggleDropdown"
      @blur="handleBlur"
      :title="collapsed ? localeName : ''"
    >
      <svg width="18" height="18" viewBox="0 0 20 20" fill="none" class="globe-icon">
        <circle cx="10" cy="10" r="7.5" stroke="currentColor" stroke-width="1.5"/>
        <path d="M3 10H17" stroke="currentColor" stroke-width="1.5"/>
        <path d="M10 3C10 3 7.5 5.5 7.5 10C7.5 14.5 10 17 10 17" stroke="currentColor" stroke-width="1.5"/>
        <path d="M10 3C10 3 12.5 5.5 12.5 10C12.5 14.5 10 17 10 17" stroke="currentColor" stroke-width="1.5"/>
      </svg>
      <span v-if="!collapsed" class="language-label">{{ localeName }}</span>
      <svg
        v-if="!collapsed"
        class="chevron"
        :class="{ 'chevron-open': isDropdownOpen }"
        width="14" height="14" viewBox="0 0 16 16" fill="none"
      >
        <path d="M4 10L8 6L12 10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
      </svg>
    </button>

    <div v-if="isDropdownOpen" class="dropdown-menu">
      <button
        v-for="locale in availableLocales"
        :key="locale"
        class="dropdown-item"
        :class="{ active: currentLocale === locale }"
        @mousedown.prevent="selectLanguage(locale)"
      >
        <span class="language-name">{{ getLanguageName(locale) }}</span>
        <svg
          v-if="currentLocale === locale"
          width="16" height="16" viewBox="0 0 18 18" fill="none" class="check-icon"
        >
          <path d="M4 9L7.5 12.5L14 6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useI18n } from '../composables/useI18n'

const props = defineProps({
  collapsed: { type: Boolean, default: false }
})

const { currentLocale, setLocale, availableLocales, localeName } = useI18n()

const isDropdownOpen = ref(false)

const languageNames = {
  en: 'English',
  ja: '日本語'
}

const getLanguageName = (locale) => {
  return languageNames[locale] || locale
}

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value
}

const handleBlur = () => {
  setTimeout(() => {
    isDropdownOpen.value = false
  }, 200)
}

const selectLanguage = (locale) => {
  setLocale(locale)
  isDropdownOpen.value = false
}
</script>

<style scoped>
.language-switcher {
  position: relative;
}

.language-button {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 10px;
  background: rgba(255,255,255,0.04);
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 150ms ease;
  font-family: inherit;
  font-size: 0.8125rem;
  color: rgba(255,255,255,0.55);
  width: 100%;
}
.language-button:hover {
  background: rgba(255,255,255,0.08);
  color: rgba(255,255,255,0.8);
}

.is-collapsed .language-button {
  justify-content: center;
  padding: 8px;
}

.globe-icon {
  flex-shrink: 0;
}
.language-label {
  font-weight: 500;
  flex: 1;
}
.chevron {
  color: rgba(255,255,255,0.3);
  transition: transform 200ms ease;
  flex-shrink: 0;
}
.chevron-open {
  transform: rotate(180deg);
}

/* Dropdown opens upward in sidebar */
.dropdown-menu {
  position: absolute;
  bottom: calc(100% + 6px);
  left: 0;
  min-width: 160px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 -4px 16px rgba(16,37,50,0.1), 0 2px 6px rgba(16,37,50,0.05);
  z-index: 1000;
  overflow: hidden;
}
/* When collapsed, open to the right */
.is-collapsed .dropdown-menu {
  bottom: 0;
  left: calc(100% + 8px);
}

.dropdown-item {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
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
.dropdown-item:hover {
  background: var(--ms-grey-50);
}
.dropdown-item.active {
  background: var(--ms-mint-pale);
  color: var(--ms-green-deep);
}
.language-name { flex: 1; }
.check-icon {
  color: var(--ms-green-deep);
  flex-shrink: 0;
}
</style>
