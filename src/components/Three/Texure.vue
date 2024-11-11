<script setup lang="ts">
  import * as THREE from 'three'

  import { onMounted, ref } from 'vue'
  import textureURL from './resources/wall.jpg'

  const canvasRef = ref()

  onMounted(() => {
    const canvas = canvasRef.value

    const renderer = new THREE.WebGLRenderer({ canvas, antialias: true })
    renderer.setSize(canvas.clientWidth, canvas.clientHeight)
    const scene = new THREE.Scene()
    const camera = new THREE.PerspectiveCamera(30, 1, 0.5, 1000)
    camera.position.set(0, 0, 32)

    let box: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial, THREE.Object3DEventMap>
    // 纹理
    const loader = new THREE.TextureLoader()
    loader.load(textureURL, texture => {
      const material = new THREE.MeshBasicMaterial({ map: texture })
      texture.colorSpace = THREE.SRGBColorSpace
      box = new THREE.Mesh(new THREE.BoxGeometry(4, 4, 4), material)
      scene.add(box)
    })

    // 灯光
    // const light = new THREE.DirectionalLight(0xffffff, 3)
    // light.position.set(-8, 8, 8)
    // scene.add(light)

    const bgGeometry = new THREE.PlaneGeometry(1000, 1000, 8, 8)
    const bgMaterial = new THREE.MeshBasicMaterial({ color: 0xffd6c6 })
    const bg = new THREE.Mesh(bgGeometry, bgMaterial)
    bg.position.set(0, 0, -6)
    scene.add(bg)

    function animete() {
      requestAnimationFrame(animete)

      box.rotation.z += 0.01
      box.rotation.x += 0.01

      renderer.render(scene, camera)
    }
    animete()
  })
</script>

<template>
  <canvas ref="canvasRef" width="400" height="400"></canvas>
</template>

<style scoped lang="scss"></style>
