<template>
  <div class="three-container">
    <button @click="toggleFollow" class="follow-button">
      {{ followMode ? 'Stop Follow' : 'Follow Car' }}
    </button>
    <div ref="threeCanvas"></div>
    <div class="overlay-chart">
      <SpeedChart
        :speed="currentSpeed"
        :positionZ="currentPositionZ"
        :resetChart="resetChart"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import SpeedChart from './SpeedChart.vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const threeCanvas = ref(null)
const followMode = ref(false)
const currentSpeed = ref(0)
const currentPositionZ = ref(-500)
const resetChart = ref(false)

const toggleFollow = () => { followMode.value = !followMode.value }

onMounted(() => {
  // 브라우저 줌 방지 (Ctrl+스크롤)
  window.addEventListener('wheel', e => {
    if (e.ctrlKey) e.preventDefault()
  }, { passive: false })

  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0x87ceeb)

  const camera = new THREE.PerspectiveCamera(
    75, window.innerWidth / window.innerHeight, 0.1, 1000
  )
  camera.position.set(0, 5, 50)

  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(window.devicePixelRatio)
  threeCanvas.value.appendChild(renderer.domElement)

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })

  // 도로
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(20, 1000),
    new THREE.MeshPhongMaterial({ color: 0x444444 })
  )
  plane.rotation.x = -Math.PI / 2
  plane.position.y = 0.05
  scene.add(plane)

  // 중앙선
  const lineGeo = new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(0, 0.06, -500),
    new THREE.Vector3(0, 0.06, 500)
  ])
  scene.add(new THREE.Line(lineGeo, new THREE.LineBasicMaterial({ color: 0xffffff })))

  // 건물
  for (let i = -500; i <= 500; i += 100) {
    const geo = new THREE.BoxGeometry(10, 30, 10)
    const mat = new THREE.MeshPhongMaterial({ color: 0x999999 })
    const L = new THREE.Mesh(geo, mat); L.position.set(-30,15,i); scene.add(L)
    const R = new THREE.Mesh(geo, mat); R.position.set(30,15,i); scene.add(R)
  }

  scene.add(new THREE.AmbientLight(0x404040))
  const dl = new THREE.DirectionalLight(0xffffff,1)
  dl.position.set(10,10,10).normalize()
  scene.add(dl)

  let model = null, speed = Math.random()*2+1
  new GLTFLoader().load(
    '/Mercedes+Benz+GLS+580.glb',
    gltf => {
      model = gltf.scene
      model.position.set(-5,-0.4,-500)
      model.scale.set(1,1,1)
      scene.add(model)
    },
    undefined,
    console.error
  )

  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true

  function animate() {
    requestAnimationFrame(animate)
    if (model) {
      speed = Math.max(0.5, Math.min(speed + (Math.random()-0.5)*0.1, 5))
      model.position.z += speed

      // 실시간 데이터 업데이트 (반응성 보장)
      currentSpeed.value = parseFloat((speed * 10).toFixed(1)) // km/h 단위로 변환
      currentPositionZ.value = parseFloat(model.position.z.toFixed(1))

      if (model.position.z >= 500) {
        model.position.z = -500
        speed = Math.random()*2+1
        resetChart.value = true
        // 다음 프레임에서 resetChart를 false로 변경
        setTimeout(() => { resetChart.value = false }, 100)
      }

      if (followMode.value) {
        const tgt = new THREE.Vector3(model.position.x, model.position.y+5, model.position.z-20)
        camera.position.lerp(tgt, 0.1)
        camera.lookAt(model.position)
      }
    }

    if (!followMode.value) controls.update()
    renderer.render(scene, camera)
  }

  // 차트 데이터 초기화 (모델이 없어도 차트가 표시되도록)
  setTimeout(() => {
    currentSpeed.value = 0
    currentPositionZ.value = -500
  }, 100)

  animate()
})
</script>