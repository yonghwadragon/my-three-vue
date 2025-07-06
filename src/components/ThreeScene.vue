<template>
  <div>
    <button @click="toggleFollow" style="position: absolute; top: 10px; left: 10px; z-index: 10;">
      {{ followMode ? 'Stop Follow' : 'Follow Car' }}
    </button>
    <div ref="threeCanvas"></div>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const threeCanvas = ref(null)
const state = reactive({
  followMode: false
})

const toggleFollow = () => {
  state.followMode = !state.followMode
}

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

  // Resize 대응
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })

  // 도로
  const planeGeometry = new THREE.PlaneGeometry(20, 1000)
  const planeMaterial = new THREE.MeshPhongMaterial({ color: 0x444444 })
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2
  plane.position.y = 0.05
  scene.add(plane)

  // 중앙선
  const lineMaterial = new THREE.LineBasicMaterial({ color: 0xffffff })
  const points = []
  points.push(new THREE.Vector3(0, 0.06, -500))
  points.push(new THREE.Vector3(0, 0.06, 500))
  const lineGeometry = new THREE.BufferGeometry().setFromPoints(points)
  const line = new THREE.Line(lineGeometry, lineMaterial)
  scene.add(line)

  // 건물
  for (let i = -500; i <= 500; i += 100) {
    const boxGeometry = new THREE.BoxGeometry(10, 30, 10)
    const boxMaterial = new THREE.MeshPhongMaterial({ color: 0x999999 })

    const boxLeft = new THREE.Mesh(boxGeometry, boxMaterial)
    boxLeft.position.set(-30, 15, i)
    scene.add(boxLeft)
  }

  const ambientLight = new THREE.AmbientLight(0x404040)
  scene.add(ambientLight)

  const light = new THREE.DirectionalLight(0xffffff, 1)
  light.position.set(10, 10, 10).normalize()
  scene.add(light)

  let model = null
  let speed = Math.random() * 2 + 1

  const loader = new GLTFLoader()
  loader.load(
    '/Mercedes+Benz+GLS+580.glb',
    (gltf) => {
      model = gltf.scene
      model.scale.set(1, 1, 1)
      model.position.set(-5, -0.4, -500)
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

    if (model) {
      // 랜덤 속도 변동
      speed += (Math.random() - 0.5) * 0.1
      speed = Math.max(0.5, Math.min(speed, 5))

      model.position.z += speed

      if (model.position.z >= 500) {
        model.position.z = -500
        speed = Math.random() * 2 + 1
      }

      if (state.followMode) {
        // 카메라가 차량 따라가도록 설정
        const targetPos = new THREE.Vector3(
          model.position.x,
          model.position.y + 5,
          model.position.z - 20
        )
        camera.position.lerp(targetPos, 0.1)
        camera.lookAt(model.position)
      }
    }

    if (!state.followMode) {
      controls.update()
    }

    renderer.render(scene, camera)
  }

  animate()
})
</script>