<template>  
<div id="viewport">
  <div id="threejs-container"></div>
</div>
</template>

<script setup>
import { onMounted, watch } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { Rhino3dmLoader } from 'three/addons/loaders/3DMLoader.js'
import { runCompute, loadRhino } from '@/scripts/compute.js'

const loader = new Rhino3dmLoader()
loader.setLibraryPath('https://cdn.jsdelivr.net/npm/rhino3dm@8.0.0-beta2/')

const props = defineProps(['data', 'path', 'runCompute'])
const emits = defineEmits(['updateMetadata'])



let renderer, camera, scene, controls, container
let lettuceTexture, onionTexture, tomatoTexture, cheeseTexture, breadTexture, meatTexture

let solveCounter = 0

function init() {

  container = document.getElementById('threejs-container')
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(window.devicePixelRatio)
  container.appendChild(renderer.domElement)

  //create threejs scene
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#f5f6fa')

  //define camera
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(0, 40, 15)
  camera.up.set(0, 0, 1)



  //create a threejs renderer
  controls = new OrbitControls(camera, renderer.domElement)

  //create a threejs light
  scene.add(new THREE.AmbientLight(0xffffff, 1.2))
  const directionalLight = new THREE.DirectionalLight(0xffffff, 3)
  directionalLight.position.set(0, 1, 0)
  scene.add(directionalLight)


  //create threejs geometry
  const geometry = new THREE.BoxGeometry(20, 10, 10)
  const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 })
  const cube = new THREE.Mesh(geometry, material)
  cube.position.set(0, 0, 0)
  scene.add(cube)

  THREE.Object3D.DEFAULT_UP.set(0, 0, 1);


  animate()
}

function animate() {
  requestAnimationFrame(animate)
  controls.update()
  renderer.render(scene, camera)
}

async function compute() {
  const doc = await runCompute(props.data, props.path)

  if (doc.metadata) {
    emits('updateMetadata', doc.metadata)
  }

  scene.traverse((child) => {
    if (!child.isLight) scene.remove(child)
  })

  const buffer = new Uint8Array(doc.toByteArray()).buffer
  loader.parse(buffer, function (object) {
    object.traverse((child) => {
      if (child.isMesh && child.userData?.attributes?.userStrings?.length) {

        // DO something here

      }
    })

    scene.add(object)

    if (solveCounter === 0) {
      for (let child of scene.children) {
        if (child.type === 'Object3D') {
          zoomCameraToSelection(camera, controls, child.children)
        }
      }
    }

    solveCounter++
  })
}

function zoomCameraToSelection(camera, controls, selection, fitOffset = 1.1) {
  const box = new THREE.Box3()
  for (const object of selection) {
    if (!object.isLight) box.expandByObject(object)
  }

  const size = box.getSize(new THREE.Vector3())
  const center = box.getCenter(new THREE.Vector3())
  const maxSize = Math.max(size.x, size.y, size.z)
  const fitHeightDistance = maxSize / (2 * Math.atan(Math.PI * camera.fov / 360))
  const fitWidthDistance = fitHeightDistance / camera.aspect
  const distance = fitOffset * Math.max(fitHeightDistance, fitWidthDistance)

  const direction = controls.target.clone().sub(camera.position).normalize().multiplyScalar(distance)
  controls.maxDistance = distance * 10
  controls.target.copy(center)

  camera.near = distance / 100
  camera.far = distance * 100
  camera.updateProjectionMatrix()
  camera.position.copy(controls.target).sub(direction)
  controls.update()
}

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
})

  //VUE METHODS
  //this vue method watches for changes in the incoming data
  watch(
    () => props.data,
    (newValue) => {
      console.log(props.data);
      if (newValue) {
        // compute();  
      }
    },
    { deep: true }
  );

  //this vue method loads three and compute whenthe component is initialized 
  onMounted(async () => {
    init()
    // await loadRhino()
    // compute()
  
  })


</script>

<style>
#threejs-container {
  position: absolute;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100vw;
  z-index: 0;
  overflow: hidden;
}

#viewport {
  position: fixed;
}




</style>
