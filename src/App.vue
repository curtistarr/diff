<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue';
import Diff from './components/Diff.vue';

type ThemePreference = 'system' | 'light' | 'dark' | 'underwater';

const themePreference = ref<ThemePreference>('system');
const systemPrefersDark = ref(false);
let mediaQuery: MediaQueryList | null = null;

const effectiveTheme = computed(() =>
  themePreference.value === 'system'
    ? (systemPrefersDark.value ? 'dark' : 'light')
    : themePreference.value,
);

const applyTheme = () => {
  const theme = effectiveTheme.value;
  const root = document.documentElement;
  root.dataset.theme = theme;
  root.style.colorScheme = theme;
};

const handleSystemChange = (event: MediaQueryListEvent) => {
  systemPrefersDark.value = event.matches;
  if (themePreference.value === 'system') {
    applyTheme();
  }
};

onMounted(() => {
  mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
  systemPrefersDark.value = mediaQuery.matches;
  mediaQuery.addEventListener('change', handleSystemChange);
  applyTheme();
});

onBeforeUnmount(() => {
  mediaQuery?.removeEventListener('change', handleSystemChange);
});

watch(themePreference, applyTheme);
</script>

<template>
  <header class="top-bar">
    <a href="https://github.com/CurtisTarr/diff" target="_blank" aria-label="Project source code">
      <img src="./assets/github-mark-white.svg" alt="GitHub logo" class="github-logo" />
    </a>

    <label class="theme-toggle" for="theme">
      Theme
      <select id="theme" v-model="themePreference" aria-label="Select color theme preference">
        <option value="system">System</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
        <option value="underwater">Underwater</option>
      </select>
    </label>
  </header>

  <h2>Diff</h2>
  <Diff />
</template>

<style scoped>
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.github-logo {
  position: fixed;
  top: 14px;
  left: 14px;
  width: 32px;
  height: 32px;
  opacity: 0.8;
  transition: transform 0.15s ease, opacity 0.2s ease, filter 0.2s ease;
}

.github-logo:hover {
  transform: scale(1.06);
  opacity: 1;
  filter: drop-shadow(0 6px 16px rgba(0, 0, 0, 0.4));
}

.theme-toggle {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.3rem 0.6rem;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  color: var(--text);
}

.theme-toggle select {
  padding: 0.3rem 0.5rem;
  border-radius: var(--radius-sm);
  border: 1px solid var(--border-subtle);
  background: var(--surface-1, var(--surface));
  color: var(--text);
}

h2 {
  margin: 12px 0 18px;
}
</style>
