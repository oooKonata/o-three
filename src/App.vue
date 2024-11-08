<script setup lang="ts">
  import SolarSystem from '@/components/Three/SolarSystem.vue'
  import Texure from './components/Three/Texure.vue'
</script>

<template>
  <main>
    <SolarSystem />
    <Texure />
  </main>
</template>

<style>
  * {
    margin: 0;
    padding: 0;
  }
  canvas {
    display: block;
  }
</style>
