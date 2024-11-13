<script setup lang="ts">
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/Addons.js'
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
    renderer.toneMappingExposure = 0.3
    // 启用阴影
    renderer.shadowMap.enabled = true
    renderer.shadowMap.type = THREE.PCFSoftShadowMap // 柔和阴影
    // 相机
    const camera = new THREE.PerspectiveCamera(30, window.innerWidth / window.innerHeight, 0.1, 1000)
    camera.position.set(20, -20, 20)
    camera.up.set(0, 0, 1)
    camera.lookAt(new THREE.Vector3(0, 0, 0))
    // 坐标轴
    const axesHelper = new THREE.AxesHelper(20)
    scene.add(axesHelper)
    // 网格
    const gridHelper = new THREE.GridHelper(40, 40)
    gridHelper.rotation.x = Math.PI / 2
    scene.add(gridHelper)

    // HDR
    const hdrLoader = new RGBELoader()
    hdrLoader.load(hdr, texture => {
      texture.mapping = THREE.EquirectangularReflectionMapping
      // scene.background = texture
      scene.environment = texture
    })

    // 球
    const sphere = new THREE.Mesh(
      new THREE.SphereGeometry(2, 32),
      new THREE.MeshStandardMaterial({
        color: 'indigo',
        roughness: 0,
        metalness: 0.8,
        transparent: true,
        opacity: 0.8,
        emissive: 'indigo',
      })
    )
    sphere.position.set(0, 0, 4)
    sphere.castShadow = true
    scene.add(sphere)

    // 节
    const torusKnot = new THREE.Mesh(
      new THREE.TorusKnotGeometry(1, 0.4, 128, 32, 2, 3),
      new THREE.MeshStandardMaterial({
        // color: '#fff',
        roughness: 0,
        metalness: 0.9,
      })
    )
    torusKnot.position.set(1, 1, 1.2)
    torusKnot.castShadow = true
    scene.add(torusKnot)

    // 立方体
    let cube: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial, THREE.Object3DEventMap>
    const loader = new THREE.TextureLoader()
    loader.load(texture, texture => {
      cube = new THREE.Mesh(
        new THREE.BoxGeometry(2, 2, 2),
        new THREE.MeshStandardMaterial({
          color: '#00f0f0',
          roughness: 0.6,
          metalness: 0.3,
          map: texture,
        })
      )
      texture.wrapS = THREE.RepeatWrapping
      texture.wrapT = THREE.RepeatWrapping
      texture.magFilter = THREE.NearestFilter //纹理放大模式
      texture.repeat.set(1, 1)
      cube.position.set(-2, -2, 1.5)
      cube.castShadow = true
      scene.add(cube)
    })

    // 循环球链
    const group = new THREE.Group()
    const geometry = new THREE.SphereGeometry(0.4, 16, 16)
    const material = new THREE.MeshStandardMaterial({
      color: '#87CEEB',
      roughness: 0.1,
      metalness: 0.8,
    })
    const protoSphere = new THREE.Mesh(geometry, material)
    // group.add(protoSphere)
    scene.add(group)
    for (let i = 0; i < 1; i += 0.05) {
      const cloneSphere = protoSphere.clone()
      cloneSphere.position.x = Math.cos(2 * Math.PI * i) * 4
      cloneSphere.position.y = Math.sin(2 * Math.PI * i) * 4
      group.add(cloneSphere)
    }
    group.position.set(0, 0, 4)

    // 背景
    const bg = new THREE.Mesh(
      new THREE.PlaneGeometry(10, 10),
      new THREE.MeshStandardMaterial({ color: '#C3D8FF', roughness: 0.5, metalness: 0.3, side: THREE.DoubleSide })
    )
    // bg.receiveShadow = true
    scene.add(bg)

    // 平行光
    const directionalLight = new THREE.DirectionalLight('#fff', 2)
    directionalLight.position.set(-8, -8, 16)
    directionalLight.castShadow = true
    directionalLight.shadow.mapSize.width = 512 //阴影分辨率
    directionalLight.shadow.mapSize.height = 512
    scene.add(directionalLight)
    // 环境光
    // const ambientLight = new THREE.AmbientLight('red', 0.3)
    // scene.add(ambientLight)
    // 天际线光
    // const hemisphereLight = new THREE.HemisphereLight('white', 'darkslategrey', 5)
    // scene.add(hemisphereLight)
    // 点光
    const pointLight = new THREE.PointLight('#fff', 200)
    pointLight.position.set(3, -3, 8)
    pointLight.castShadow = true
    pointLight.shadow.mapSize.width = 512 //阴影分辨率
    pointLight.shadow.mapSize.height = 512
    scene.add(pointLight)

    // 控制器
    const controls = new OrbitControls(camera, canvas)
    controls.target.copy(bg.position)
    controls.enablePan = false // 禁用平移
    controls.enableZoom = false // 禁用缩放
    // controls.enableRotate = false // 禁用旋转
    // controls.listenToKeyEvents(window) // 箭头键平移相机
    // controls.autoRotate = true // 自动旋转，围绕target
    // controls.autoRotateSpeed = 1 // 自动旋转速度
    controls.minAzimuthAngle = -(Math.PI * 1) / 2 // 水平旋转最小角度 [-PI,0]
    controls.maxAzimuthAngle = (Math.PI * 1) / 2 // 水平旋转最大角度 [0,PI]
    controls.minPolarAngle = (Math.PI * 1) / 4 // 垂直旋转最小角度 [0,PI/2]
    controls.maxPolarAngle = (Math.PI * 1.5) / 4 // 垂直旋转最大角度 [PI/2,PI]
    controls.enableDamping = true // 启用阻尼
    controls.dampingFactor = 0.1 // 阻尼速度

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

      torusKnot.rotation.z += Math.PI * frameDuration * 0.001 // 每秒旋转180度
      cube.rotation.z += -Math.PI * frameDuration * 0.0005
      group.rotation.z += -Math.PI * frameDuration * 0.0001

      controls.update()

      renderer.render(scene, camera)
    }
    animete(0)
  })
</script>

<template>
  <canvas ref="canvasRef" width="600" height="400"></canvas>
</template>

<style scoped lang="scss"></style>
