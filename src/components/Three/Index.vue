<script setup lang="ts">
  import * as THREE from 'three'
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
  import { onMounted, ref, render } from 'vue'

  const canvasRef = ref()

  onMounted(() => {
    const canvas = canvasRef.value

    const scene = new THREE.Scene()
    scene.background = new THREE.Color('#222')
    const renderer = new THREE.WebGLRenderer({ canvas })
    renderer.setSize(canvas.clientWidth, canvas.clientHeight)
    const camera = new THREE.PerspectiveCamera(30, 1, 0.1, 1000)
    camera.position.set(0, 0, 32)
    const controls = new OrbitControls(camera, canvas)
    controls.update()

    const cube = new THREE.Mesh(
      new THREE.BoxGeometry(2, 2, 2),
      new THREE.MeshStandardMaterial({ color: 'pink', roughness: 0.1, metalness: 0.5 })
    )
    scene.add(cube)

    function animete() {
      requestAnimationFrame(animete)
      renderer.render(scene, camera)
    }
    animete()
  })
</script>

<template>
  <canvas ref="canvasRef" width="400" height="400"></canvas>
</template>

<style scoped lang="scss"></style>
