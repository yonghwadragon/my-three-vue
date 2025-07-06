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
  scene.background = new THREE.Color(0xffffff)

  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.set(0, 1, 5)

  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  threeCanvas.value.appendChild(renderer.domElement)

  const loader = new GLTFLoader()
  loader.load(
    '/Mercedes+Benz+GLS+580.glb',
    (gltf) => {
      const model = gltf.scene
      model.scale.set(1, 1, 1) // 필요 시 크기 조절
      scene.add(model)
    },
    undefined,
    (error) => {
      console.error(error)
    }
  )

  const light = new THREE.DirectionalLight(0xffffff, 1)
  light.position.set(10, 10, 10).normalize()
  scene.add(light)

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