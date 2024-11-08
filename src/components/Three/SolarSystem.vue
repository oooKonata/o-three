<script setup lang="ts">
  import * as THREE from 'three'
  import { onMounted, ref } from 'vue'

  const canvasRef = ref()

  onMounted(() => {
    const canvas = canvasRef.value

    const renderer = new THREE.WebGLRenderer({ canvas, antialias: true })
    const scene = new THREE.Scene()
    renderer.setSize(canvas.clientWidth, canvas.clientHeight)

    let fov = 30
    let aspect = canvas.clientWidth / canvas.clientHeight
    let near = 0.5
    let far = 1000
    const camera = new THREE.PerspectiveCamera(fov, aspect, near, far)
    camera.position.set(0, 0, 32)

    const sunGeometry = new THREE.IcosahedronGeometry(2, 2)
    const sunMaterial = new THREE.MeshNormalMaterial({ flatShading: true })
    const sun = new THREE.Mesh(sunGeometry, sunMaterial)
    scene.add(sun)

    const earthGeometry = new THREE.SphereGeometry(1, 16, 16)
    const earthMaterial = new THREE.MeshPhongMaterial({ color: 0xb1cbff, emissive: 0xb1cbff })
    const earth = new THREE.Mesh(earthGeometry, earthMaterial)

    const moonGeometry = new THREE.SphereGeometry(0.4, 8, 8)
    const moonMaterial = new THREE.MeshPhongMaterial({ color: 0xffbbad, emissive: 0xffbbad })
    const moon = new THREE.Mesh(moonGeometry, moonMaterial)

    // 层级关系
    const earthOrbit = new THREE.Object3D()
    earthOrbit.add(moon)
    moon.position.set(2, 0, 0)
    scene.add(earthOrbit)

    const solarOrbit = new THREE.Object3D()
    solarOrbit.add(earth)
    earth.position.set(-6, 0, 0)
    scene.add(solarOrbit)

    const bgGeometry = new THREE.PlaneGeometry(1000, 1000, 8, 8)
    const bgMaterial = new THREE.MeshBasicMaterial({ color: 0x333333 })
    const bg = new THREE.Mesh(bgGeometry, bgMaterial)
    bg.position.set(0, 0, -6)
    scene.add(bg)

    const light = new THREE.DirectionalLight(0xffffff, 3)
    light.position.set(-8, 8, 8)
    scene.add(light)

    const pathPoints: THREE.Vector3[] = []
    const maxPathLength = 80
    let lastTime: number = 0
    let lineGeometry = new THREE.BufferGeometry()
    let line = new THREE.Line(lineGeometry, new THREE.LineBasicMaterial({ color: 0xff5838 }))
    scene.add(line)

    function animate(time?: number) {
      requestAnimationFrame(animate)

      // 月球轨道中心点 = 地球位置
      const earthMeshWorldPosition = new THREE.Vector3()
      earth.getWorldPosition(earthMeshWorldPosition)
      earthOrbit.position.x = earthMeshWorldPosition.x
      earthOrbit.position.y = earthMeshWorldPosition.y
      earthOrbit.position.z = earthMeshWorldPosition.z

      // const deltaTime = time! - lastTime
      // console.log('11', deltaTime, time, lastTime)
      // lastTime = time!
      // const radians = (cycleTime: number) => THREE.MathUtils.degToRad(((deltaTime * 0.001) / cycleTime) * 360)
      const a = 0.0005

      sun.rotation.z += 2.0 * a
      solarOrbit.rotation.z += 30 * a
      earth.rotation.z += 0.0826 * a
      earthOrbit.rotation.z += 2.24 * a * 100
      moon.rotation.z += 2.24 * a

      const moonWorldPosition = new THREE.Vector3()
      moon.getWorldPosition(moonWorldPosition)
      pathPoints.push(moonWorldPosition.clone())
      if (pathPoints.length > maxPathLength) {
        pathPoints.shift()
      }

      lineGeometry.dispose()
      lineGeometry = new THREE.BufferGeometry().setFromPoints(pathPoints)
      line.geometry = lineGeometry
      line.computeLineDistances()

      // 响应式
      // camera.aspect = canvas.clientWidth / canvas.clientHeight
      // camera.updateProjectionMatrix()
      // renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.render(scene, camera)
    }
    animate()
  })
</script>

<template>
  <canvas ref="canvasRef" width="400" height="400"></canvas>
</template>

<style scoped lang="scss"></style>
