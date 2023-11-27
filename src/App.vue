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
  top: 10px;
  left: 10px;
  width: 30px;
  height: 30px;
}
</style>
