<script setup lang="ts">
  import SolarSystem from '@/components/Three/SolarSystem.vue'
  import Texure from './components/Three/Texure.vue'
  import Light from './components/Three/light.vue'
</script>

<template>
  <main>
    <!-- <SolarSystem /> -->
    <!-- <Texure /> -->
    <Light />
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
