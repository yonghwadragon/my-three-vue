<template>
  <div ref="threeCanvas"></div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const threeCanvas = ref(null)

onMounted(() => {
  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0x87ceeb)

  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.set(0, 5, 50)

  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  threeCanvas.value.appendChild(renderer.domElement)

  // ✅ 도로 (Z축 방향)
  const planeGeometry = new THREE.PlaneGeometry(20, 1000)
  const planeMaterial = new THREE.MeshPhongMaterial({ color: 0x444444 })
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2
  plane.position.y = 0.05
  scene.add(plane)

  // ✅ 중앙선 (Z축 방향)
  const lineMaterial = new THREE.LineBasicMaterial({ color: 0xffffff })
  const points = []
  points.push(new THREE.Vector3(0, 0.06, -500))
  points.push(new THREE.Vector3(0, 0.06, 500))
  const lineGeometry = new THREE.BufferGeometry().setFromPoints(points)
  const line = new THREE.Line(lineGeometry, lineMaterial)
  scene.add(line)

  // ✅ 건물 (Z축 방향, 좌우 배치)
  for (let i = -500; i <= 500; i += 100) {
    const boxGeometry = new THREE.BoxGeometry(10, 30, 10)
    const boxMaterial = new THREE.MeshPhongMaterial({ color: 0x999999 })

    const boxLeft = new THREE.Mesh(boxGeometry, boxMaterial)
    boxLeft.position.set(-30, 15, i)
    scene.add(boxLeft)

    const boxRight = new THREE.Mesh(boxGeometry, boxMaterial)
    boxRight.position.set(30, 15, i)
    scene.add(boxRight)
  }

  const ambientLight = new THREE.AmbientLight(0x404040)
  scene.add(ambientLight)

  const light = new THREE.DirectionalLight(0xffffff, 1)
  light.position.set(10, 10, 10).normalize()
  scene.add(light)

  // ✅ 자동차 위치 조정 (Z축 방향으로 뒤쪽으로 배치)
  const loader = new GLTFLoader()
  loader.load(
    '/Mercedes+Benz+GLS+580.glb',
    (gltf) => {
      const model = gltf.scene
      model.scale.set(1, 1, 1)
      model.position.set(-5, -0.4, -200) // ← 차량 위치 조정
      scene.add(model)
    },
    undefined,
    (error) => {
      console.error(error)
    }
  )

  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true

  function animate() {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
  }

  animate()
})
</script>