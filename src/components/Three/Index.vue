<script setup lang="ts">
  import * as THREE from 'three'
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
  import { RGBELoader } from 'three/examples/jsm/Addons.js'
  import { onMounted, ref, render } from 'vue'
  import hdr from './resources/blue_photo_studio_2k.hdr?url'
  import texture from './resources/checker.png'

  console.log(hdr)

  const canvasRef = ref()

  onMounted(() => {
    const canvas = canvasRef.value
    // 场景
    const scene = new THREE.Scene()
    scene.background = new THREE.Color('#222')
    // 渲染器
    const renderer = new THREE.WebGLRenderer({ canvas, antialias: true })
    renderer.setSize(window.innerWidth, window.innerHeight)
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 4)) //最大比率不超过2
    // 色调映射（应用于HDR渲染，提升场景明暗对比和亮度）
    renderer.toneMapping = THREE.ACESFilmicToneMapping
    renderer.toneMappingExposure = 0.2
    // 启用阴影
    renderer.shadowMap.enabled = true
    renderer.shadowMap.type = THREE.PCFSoftShadowMap // 柔和阴影
    // 相机
    const camera = new THREE.PerspectiveCamera(30, window.innerWidth / window.innerHeight, 0.1, 1000)
    camera.position.set(0, -20, 20)
    // 控制器
    const controls = new OrbitControls(camera, canvas)
    controls.update()

    // HDR
    const hdrLoader = new RGBELoader()
    hdrLoader.load(hdr, texture => {
      texture.mapping = THREE.EquirectangularReflectionMapping
      // scene.background = texture
      scene.environment = texture
    })

    const sphere = new THREE.Mesh(
      new THREE.SphereGeometry(2, 32),
      new THREE.MeshStandardMaterial({
        color: '#e8faf7',
        roughness: 0.3,
        metalness: 0.8,
        transparent: true,
        opacity: 0.5,
        emissive: '#00ff00',
        // flatShading: true,
      })
    )
    sphere.position.set(0, 0, 4)
    sphere.castShadow = true
    scene.add(sphere)

    const torusKnot = new THREE.Mesh(
      new THREE.TorusKnotGeometry(1, 0.4, 128, 32, 2, 3),
      new THREE.MeshStandardMaterial({
        color: '#92EDDF',
        roughness: 0.3,
        metalness: 0.3,
      })
    )
    torusKnot.position.set(1, 1, 1.2)
    torusKnot.castShadow = true
    scene.add(torusKnot)

    let cube: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial, THREE.Object3DEventMap>

    const loader = new THREE.TextureLoader()
    loader.load(texture, texture => {
      cube = new THREE.Mesh(
        new THREE.BoxGeometry(2, 2, 1),
        new THREE.MeshStandardMaterial({
          // color: '#92EDDF',
          // roughness: 0.3,
          // metalness: 0.3,
          map: texture,
        })
      )
      cube.position.set(-2, -2, 1)
      cube.castShadow = true
      scene.add(cube)
    })

    // 背景
    const bg = new THREE.Mesh(
      new THREE.PlaneGeometry(10, 10),
      new THREE.MeshStandardMaterial({ color: '#FFE69F', roughness: 0.5, metalness: 0.8, side: THREE.DoubleSide })
    )
    bg.receiveShadow = true
    scene.add(bg)

    // 灯光
    const light = new THREE.DirectionalLight('#fff', 10)
    light.position.set(-8, 8, 8)
    light.castShadow = true
    // 配置阴影参数
    light.shadow.mapSize.width = 2048 //阴影分辨率
    light.shadow.mapSize.height = 2048
    scene.add(light)

    // 响应式
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
    })

    let lastFrameTime = 0

    function animete(currentFrameTime?: number) {
      requestAnimationFrame(animete)

      // 帧时长 Frame Duration (ms)
      const frameDuration = currentFrameTime! - lastFrameTime
      lastFrameTime = currentFrameTime!

      console.log(frameDuration)

      torusKnot.rotation.z += Math.PI * frameDuration * 0.001 // 每秒旋转180度

      renderer.render(scene, camera)
    }
    animete(0)
  })
</script>

<template>
  <canvas ref="canvasRef" width="600" height="400"></canvas>
</template>

<style scoped lang="scss"></style>
