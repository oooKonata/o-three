<script setup lang="ts">
  import * as THREE from 'three'
  import { GUI } from 'lil-gui'
  import { RGBELoader, GLTFLoader, OrbitControls } from 'three/examples/jsm/Addons.js'
  import { onMounted, ref } from 'vue'
  import hdr from './resources/blue_photo_studio_2k.hdr?url'
  import texture from './resources/checker.png'
  import parrotGlb from './resources/Parrot.glb?url'
  import flamingoGlb from './resources/Flamingo.glb?url'
  import storkGlb from './resources/Stork.glb?url'

  const canvasRef = ref()

  const gui = new GUI()
  const params = {
    color: '#ffffff',
    size: 10,
  }
  gui.addColor(params, 'color').name('LightColor')
  gui.add(params, 'size', 1, 50, 1).name('LightIntensity')

  onMounted(async () => {
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
    // 开启视图裁剪
    renderer.setScissorTest(true)
    // 相机
    const cameraA = new THREE.PerspectiveCamera(30, window.innerWidth / 2 / window.innerHeight, 0.1, 1000)
    cameraA.position.set(32, -32, 48)
    cameraA.up.set(0, 0, 1)
    cameraA.lookAt(0, 0, 0)
    // 相机2
    const cameraB = new THREE.PerspectiveCamera(30, window.innerWidth / 2 / window.innerHeight, 0.1, 1000)
    cameraB.position.set(-32, -32, 48)
    cameraB.up.set(0, 0, 1)
    cameraB.lookAt(0, 0, 0)
    // 相机视锥体辅助对象
    const cameraHelper = new THREE.CameraHelper(cameraA)
    scene.add(cameraHelper)
    // 控制器
    const controlsA = new OrbitControls(cameraA, document.getElementById('cameraA'))
    controlsA.target.set(0, 0, 0)
    const controlsB = new OrbitControls(cameraB, document.getElementById('cameraB'))
    controlsB.target.set(0, 0, 0)

    // controls.enablePan = false // 禁用平移
    // controls.enableZoom = false // 禁用缩放
    // controls.enableRotate = false // 禁用旋转
    // controls.listenToKeyEvents(window) // 箭头键平移相机
    // controls.autoRotate = true // 自动旋转，围绕target
    // controls.autoRotateSpeed = 1 // 自动旋转速度
    // controls.minAzimuthAngle = -(Math.PI * 1) / 2 // 水平旋转最小角度 [-PI,0]
    // controls.maxAzimuthAngle = (Math.PI * 1) / 2 // 水平旋转最大角度 [0,PI]
    // controls.minPolarAngle = (Math.PI * 1) / 4 // 垂直旋转最小角度 [0,PI/2]
    // controls.maxPolarAngle = (Math.PI * 1.5) / 4 // 垂直旋转最大角度 [PI/2,PI]
    controlsA.enableDamping = true // 启用阻尼
    controlsA.dampingFactor = 0.1 // 阻尼速度
    controlsB.enableDamping = true // 启用阻尼
    controlsB.dampingFactor = 0.1 // 阻尼速度
    // 坐标轴
    const axesHelper = new THREE.AxesHelper(20)
    scene.add(axesHelper)
    // 网格
    const gridHelper = new THREE.GridHelper(40, 40)
    gridHelper.rotation.x = Math.PI / 2
    scene.add(gridHelper)
    // 时钟
    const clock = new THREE.Clock()

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

    // // 节
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
    protoSphere.castShadow = true
    scene.add(group)
    for (let i = 0; i < 1; i += 0.05) {
      const cloneSphere = protoSphere.clone()
      cloneSphere.position.x = Math.cos(2 * Math.PI * i) * 12
      cloneSphere.position.y = Math.sin(2 * Math.PI * i) * 12
      cloneSphere.position.z = Math.sin(2 * Math.PI * 4 * i)
      group.add(cloneSphere)
    }
    group.position.set(0, 0, 2)

    // glTF格式加载3D模型
    const gltfLoader = new GLTFLoader()
    const parrotData = await gltfLoader.loadAsync(parrotGlb)
    const parrotModel = parrotData.scene.children[0]
    parrotModel.scale.set(0.04, 0.04, 0.04)
    parrotModel.position.set(0, -8, 4)
    parrotModel.rotation.set(Math.PI / 2, 0, 0)
    scene.add(parrotModel)
    const parrotMixer = new THREE.AnimationMixer(parrotModel)
    const parrotClip = parrotData.animations[0]
    const parrotAction = parrotMixer.clipAction(parrotClip)
    parrotAction.play()
    parrotModel.castShadow = true
    parrotModel.receiveShadow = true

    const flamingoData = await gltfLoader.loadAsync(flamingoGlb)
    const flamingoModel = flamingoData.scene.children[0]
    flamingoModel.scale.set(0.03, 0.03, 0.03)
    flamingoModel.position.set(-6, -4, 2)
    flamingoModel.rotation.set(Math.PI / 2, 0, 0)
    scene.add(flamingoModel)
    const flamingoMixer = new THREE.AnimationMixer(flamingoModel)
    const flamingoClip = flamingoData.animations[0]
    const flamingoAction = flamingoMixer.clipAction(flamingoClip)
    flamingoAction.play()
    flamingoModel.castShadow = true
    flamingoModel.receiveShadow = true

    const storkData = await gltfLoader.loadAsync(storkGlb)
    const storkModel = storkData.scene.children[0]
    storkModel.scale.set(0.03, 0.03, 0.03)
    storkModel.position.set(6, -4, 2)
    storkModel.rotation.set(Math.PI / 2, 0, 0)
    scene.add(storkModel)
    const storkMixer = new THREE.AnimationMixer(storkModel)
    const storkClip = storkData.animations[0]
    const storkAction = storkMixer.clipAction(storkClip)
    storkAction.play()
    storkModel.castShadow = true
    storkModel.receiveShadow = true

    // 背景
    const bg = new THREE.Mesh(
      new THREE.PlaneGeometry(40, 40),
      new THREE.MeshStandardMaterial({ color: '#C3D8FF', roughness: 0.5, metalness: 0.3, side: THREE.DoubleSide })
    )
    bg.receiveShadow = true
    scene.add(bg)

    // // 平行光
    const directionalLight = new THREE.DirectionalLight('#fff', 10)
    directionalLight.position.set(-10, -10, 20)
    // 调整阴影摄像机的参数，以适应物体缩放后的大小
    directionalLight.shadow.camera.left = -100
    directionalLight.shadow.camera.right = 100
    directionalLight.shadow.camera.top = 100
    directionalLight.shadow.camera.bottom = -100
    // 远近平面（控制阴影的深度范围）
    directionalLight.shadow.camera.near = 0.1
    directionalLight.shadow.camera.far = 1000
    directionalLight.castShadow = true
    directionalLight.shadow.mapSize.width = 2048 //阴影分辨率
    directionalLight.shadow.mapSize.height = 2048
    scene.add(directionalLight)
    // 环境光
    // const ambientLight = new THREE.AmbientLight('red', 0.3)
    // scene.add(ambientLight)
    // 天际线光
    // const hemisphereLight = new THREE.HemisphereLight('white', 'darkslategrey', 5)
    // scene.add(hemisphereLight)
    // 点光
    const pointLight = new THREE.PointLight('#fff', 1000)
    pointLight.position.set(3, -3, 8)
    pointLight.castShadow = true
    pointLight.shadow.mapSize.width = 2048 //阴影分辨率
    pointLight.shadow.mapSize.height = 2048
    scene.add(pointLight)

    // 响应式
    window.addEventListener('resize', () => {
      renderer.setSize(window.innerWidth, window.innerHeight)
      cameraA.aspect = (window.innerWidth * 0.5) / window.innerHeight
      cameraA.updateProjectionMatrix()
      cameraB.aspect = (window.innerWidth * 0.5) / window.innerHeight
      cameraB.updateProjectionMatrix()
    })

    function animete(time?: number) {
      requestAnimationFrame(animete)

      // 帧间隔（动态）(单位s)
      const deltaTime = clock.getDelta()

      torusKnot.rotation.z += Math.PI * deltaTime // 每秒旋转180度
      if (cube) {
        cube.rotation.z += -Math.PI * deltaTime * 0.5
      }
      group.rotation.z += Math.PI * deltaTime * 0.1 // 10s一圈

      group.children.forEach((sphere, index) => {
        const offset = index * 0.5
        sphere.position.z = Math.sin(Math.PI * (time! * 0.001 + offset)) * 1.2
      })

      parrotMixer.update(deltaTime)
      flamingoMixer.update(deltaTime)
      storkMixer.update(deltaTime)

      // cameraHelper.update()
      controlsA.update()
      controlsB.update()

      // 清除上一帧
      renderer.clear()

      renderer.setViewport(0, 0, window.innerWidth / 2, window.innerHeight)
      renderer.setScissor(0, 0, window.innerWidth / 2, window.innerHeight)
      renderer.render(scene, cameraA)

      renderer.setViewport(window.innerWidth / 2, 0, window.innerWidth / 2, window.innerHeight)
      renderer.setScissor(window.innerWidth / 2, 0, window.innerWidth / 2, window.innerHeight)
      renderer.render(scene, cameraB)
    }
    animete()
  })
</script>

<template>
  <canvas ref="canvasRef" width="600" height="400"></canvas>
  <div id="box">
    <div id="cameraA"></div>
    <div id="cameraB"></div>
  </div>
</template>

<style scoped lang="scss">
  #box {
    display: flex;
    width: 100vw;
    height: 100vh;
    position: absolute;
    left: 0;
    top: 0;
    #cameraA {
      flex: 1;
    }
    #cameraB {
      flex: 1;
    }
  }
</style>
