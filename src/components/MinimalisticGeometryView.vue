<template>
  <div id="viewport">
    <!-- To this element we will append our 3d scene. -->
    <div id="threejs-container"></div>
  </div>
</template>

<script setup>
import { onMounted, watch } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { Rhino3dmLoader } from "three/addons/loaders/3DMLoader.js";
import { runCompute } from "@/scripts/compute.js";
import { loadRhino } from "@/scripts/compute.js";

import texturePath from "../assets/graphics/brick_texture.jpg"


const loader = new Rhino3dmLoader();
loader.setLibraryPath("https://cdn.jsdelivr.net/npm/rhino3dm@8.0.0-beta2/");

const props = defineProps(["data", "path", "runCompute"]);
const emits = defineEmits(["updateMetadata"]);

watch(
  () => props.data,
  (newValue) => {
    console.log(props.data);
    if (newValue) {
      compute();
    }
  },
  { deep: true }
);

// Three js objects
let renderer, camera, scene, controls, container;
let texture;

function init() {

  container = document.getElementById('threejs-container')
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  container.appendChild(renderer.domElement)

  // Set up camera - https://threejs.org/docs/#api/en/cameras/PerspectiveCamera
  camera = new THREE.PerspectiveCamera(
    60,
    window.innerWidth / window.innerHeight,
    1,
    1000
  );
  camera.position.set(30, 40, 20);
  camera.lookAt(0, 0, 0);
  camera.up.set(0, 0, 1);

  // scene
  scene = new THREE.Scene();
  scene.background = new THREE.Color("White");

  // orbit controls
  controls = new OrbitControls(camera, renderer.domElement);

  let lightX = 20;
  let lightY = 50;
  let lightZ = 50;

  // create a spotlight with shadow camera parameters
  const hemiLight = new THREE.HemisphereLight(0xffffff, 0x8d8d8d, 1.5);
  hemiLight.position.set(lightX, lightY, lightZ);
  scene.add(hemiLight);

  let dirLightX = 20;
  let dirLightY = 50;
  let dirLightZ = 50;
  const dirLight = new THREE.DirectionalLight(0xffdb83, 6);
  dirLight.position.set(dirLightX, dirLightY, dirLightZ);
  dirLight.lookAt(0, 0, 15);
  dirLight.castShadow = true;
  dirLight.shadow.camera.left = -35;
  dirLight.shadow.camera.right = 35;
  dirLight.shadow.camera.top = 80;

  dirLight.shadow.camera.near = 0.1;
  dirLight.shadow.camera.far = 500;

  // Increase shadow map size for better quality
  dirLight.shadow.mapSize.width = 2048;
  dirLight.shadow.mapSize.height = 2048;

  scene.add(dirLight);

  const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });

  const directLightgeometry = new THREE.SphereGeometry(1, 10, 10);
  const directLightSphere = new THREE.Mesh(directLightgeometry, material);
  directLightSphere.position.set(dirLightX, dirLightY, dirLightZ);
  scene.add(directLightSphere);

  // add light helper
  const lightHelper = new THREE.DirectionalLightHelper(dirLight, 5, 0x00ff00);
  // scene.add(lightHelper);

  // add shadow plane
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(1000, 50),
    new THREE.ShadowMaterial({
      color: "rgb(194, 191, 194)",
      transparent: true,
      opacity: 0.5,
      side: THREE.DoubleSide,
    })
  );
  plane.receiveShadow = true;
  scene.add(plane);

  texture = new THREE.TextureLoader().load(
    texturePath,
    function () {
      console.log("Texture loaded successfully");
    },
    undefined,
    function (err) {
      console.error("Error loading texture:", err);
    }
  );
  THREE.Object3D.DEFAULT_UP.set(0, 0, 1);
  animate();
}

// this function runs Compute
async function compute() {
  console.log("Runnning compute... \ndata sent: ", props.data);
  const doc = await runCompute(props.data, props.path);

  if (doc.metadata) {
    emits("updateMetadata", doc.metadata);
  }

  // clear objects from scene
  scene.traverse((child) => {
    if (!child.isLight && child.isCompute) {
      scene.remove(child);
    }
  });

  // add object graph from rhino model to three.js scene
  const buffer = new Uint8Array(doc.toByteArray()).buffer;
  loader.parse(buffer, function (object) {
    object.isCompute = true;
    object.castShadow = true;
    object.traverse((child) => {
      if (child.isMesh) {
        child.castShadow = true;
        child.receiveShadow = true;
        // Create a material with the texture
        const material = new THREE.MeshStandardMaterial({
          map: texture,
          side: THREE.DoubleSide,
        });

        // Apply the material to the child
        child.material = material;
      }
    });

    scene.add(object);
    console.log("Compute done");
  });
}

// for controls update
function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}

// This will be run whenever this component is instantiated
onMounted(async () => {
  init();
  await loadRhino();
  compute();
});
</script>

<style scoped>
  #viewport {
    position: fixed;
  }

#threejs-container {
    position: absolute;
    top: 0;
    left: 0;
    height: 100vh;
    width: 100vw;
    z-index: 0;
    overflow: hidden;
}
</style>