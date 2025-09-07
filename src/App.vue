<script setup lang="ts">
import LutConverter from './components/LutConverter.vue'
</script>

<template>
  <div id="app">
    <LutConverter />
  </div>
</template>

<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  line-height: 1.6;
  padding: 1em 2em;
  margin: 0;
  color: #333;
  background-color: #f5f5f5;
}

#app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
