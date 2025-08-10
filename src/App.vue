<script setup lang="ts">
import Diff from './components/Diff.vue'
</script>

<template>
  <a href="https://github.com/CurtisTarr/diff" target="_blank">
    <img src="./assets/github-mark-white.svg" alt="GitHub logo" class="github-logo"/>
  </a>
  <h2>Diff</h2>
  <Diff />
</template>

<style scoped>
.github-logo {
  position: fixed;
  top: 14px;
  left: 14px;
  width: 32px;
  height: 32px;
  opacity: 0.8;
  transition: transform .15s ease, opacity .2s ease, filter .2s ease;
}
.github-logo:hover { transform: scale(1.06); opacity: 1; filter: drop-shadow(0 6px 16px rgba(0,0,0,.4)); }

h2 { margin: 12px 0 18px; }
</style>
