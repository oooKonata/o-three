<script setup lang="ts">
  import * as THREE from 'three'
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

  import { onMounted, ref } from 'vue'
  import textureURL from './resources/checker.png'

  const canvasRef = ref()

  onMounted(() => {
    const canvas = canvasRef.value

    const renderer = new THREE.WebGLRenderer({ canvas, antialias: true })
    renderer.setSize(canvas.clientWidth, canvas.clientHeight)
    const scene = new THREE.Scene()
    const camera = new THREE.PerspectiveCamera(30, 1, 0.5, 1000)
    camera.position.set(0, 0, 32)

    const controls = new OrbitControls(camera, canvas)
    controls.target.set(0, 0, 0)
    controls.update()

    const box = new THREE.Mesh(new THREE.BoxGeometry(2, 2, 2), new THREE.MeshPhongMaterial({ color: '#8AC' }))
    box.position.set(1.5, 0, 1.5)
    scene.add(box)

    const sphere = new THREE.Mesh(new THREE.SphereGeometry(1, 16), new THREE.MeshPhongMaterial({ color: '#CA8' }))
    sphere.position.set(-1.5, 0, 1.5)
    scene.add(sphere)

    const loader = new THREE.TextureLoader()
    loader.load(textureURL, texture => {
      texture.wrapS = THREE.RepeatWrapping
      texture.wrapT = THREE.RepeatWrapping
      texture.magFilter = THREE.NearestFilter
      texture.colorSpace = THREE.SRGBColorSpace
      texture.repeat.set(5, 5)
      const plain = new THREE.Mesh(
        new THREE.PlaneGeometry(10, 10),
        new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide })
      )
      scene.add(plain)
    })

    const light = new THREE.AmbientLight('#fff', 1)
    scene.add(light)

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
