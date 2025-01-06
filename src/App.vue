<template>
  <button type="button" id="R1">R'</button>
  <button type="button" id="R2">R</button>
  <button type="button" id="L1">L'</button>
  <button type="button" id="L2">L</button>
  <button type="button" id="F1">F'</button>
  <button type="button" id="F2">F</button>
  <button type="button" id="B1">B'</button>
  <button type="button" id="B2">B</button>
  <button type="button" id="T1">T'</button>
  <button type="button" id="T2">T</button>
  <button type="button" id="D1">D'</button>
  <button type="button" id="D2">D</button>
</template>

<script setup>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { onMounted, ref, watch, useTemplateRef } from 'vue';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

let scene = null, camera = null, renderer = null, model = null, rotatorGroup = new THREE.Group(), controls;
let cubeSizes = [
  [
    ['YBO', 'BY', 'RBY'],
    ['BO', 'B', 'BR'],
    ['OBW', 'BW1', 'WBR'],
  ], 
  [
    ['YO', 'Y', 'RY1'],
    ['O', '', 'R'],
    ['OW', 'W', 'WR'],
  ], 
  [
    ['YOG', 'YG', 'RYG'],
    ['OG', 'G', 'RG'],
    ['OWG', 'WG', 'WRG'],
  ],
];
const occupied = ref(false);

const getSizeArr = size => {
  let outputArr = [];
  switch (size) {
    case 'F':
      outputArr = [].concat(cubeSizes[0][2], cubeSizes[1][2], cubeSizes[2][2]);
      break;
    case 'B':
      outputArr = [].concat(cubeSizes[0][0], cubeSizes[1][0], cubeSizes[2][0]);
      break;
    case 'R':
      for (let cubeSize of cubeSizes) {
        for (let cubeStr of cubeSize) {
          outputArr.push(cubeStr[2]);
        }
      }
      break;
    case 'L':
      for (let cubeSize of cubeSizes) {
        for (let cubeStr of cubeSize) {
          outputArr.push(cubeStr[0]);
        }
      }
      break;
    case 'T':
      outputArr = [].concat(cubeSizes[0][0], cubeSizes[0][1], cubeSizes[0][2]);
      break;
    case 'D':
      outputArr = [].concat(cubeSizes[2][0], cubeSizes[2][1], cubeSizes[2][2]);
      break;
  }
  return outputArr;
}

const getSizeModelsArr = sizeArr => {
  let outputArr = [];
  for (let size of sizeArr) {
    outputArr.push(model.getObjectByName(size));
  }
  return outputArr;
}

const rewriteCubeSizes = size => {
  switch (size) {
    case 'F':
      cubeSizes = [
        [
          [cubeSizes[0][0][0], cubeSizes[0][0][1], cubeSizes[0][0][2]],
          [cubeSizes[0][1][0], cubeSizes[0][1][1], cubeSizes[0][1][2]],
          [cubeSizes[2][2][0], cubeSizes[1][2][0], cubeSizes[0][2][0]],
        ], 
        [
          [cubeSizes[1][0][0], cubeSizes[1][0][1], cubeSizes[1][0][2]],
          [cubeSizes[1][1][0], cubeSizes[1][1][1], cubeSizes[1][1][2]],
          [cubeSizes[2][2][1], cubeSizes[1][2][1], cubeSizes[0][2][1]],
        ], 
        [
          [cubeSizes[2][0][0], cubeSizes[2][0][1], cubeSizes[2][0][2]],
          [cubeSizes[2][1][0], cubeSizes[2][1][1], cubeSizes[2][1][2]],
          [cubeSizes[2][2][2], cubeSizes[1][2][2], cubeSizes[0][2][2]],
        ],
      ];
      break;
    case 'B':
      cubeSizes = [
        [   
          [cubeSizes[0][0][2], cubeSizes[1][0][2], cubeSizes[2][0][2]],
          [cubeSizes[0][1][0], cubeSizes[0][1][1], cubeSizes[0][1][2]],
          [cubeSizes[0][2][0], cubeSizes[0][2][1], cubeSizes[0][2][2]],
        ], 
        [
          [cubeSizes[0][0][1], cubeSizes[1][0][1], cubeSizes[2][0][1]],
          [cubeSizes[1][1][0], cubeSizes[1][1][1], cubeSizes[1][1][2]],
          [cubeSizes[1][2][0], cubeSizes[1][2][1], cubeSizes[1][2][2]],
        ], 
        [
          [cubeSizes[0][0][0], cubeSizes[1][0][0], cubeSizes[2][0][0]],
          [cubeSizes[2][1][0], cubeSizes[2][1][1], cubeSizes[2][1][2]],
          [cubeSizes[2][2][0], cubeSizes[2][2][1], cubeSizes[2][2][2]],
        ],
      ];
      break;
    case 'R':
      cubeSizes = [
        [
          [cubeSizes[0][0][0], cubeSizes[0][0][1], cubeSizes[0][2][2]],
          [cubeSizes[0][1][0], cubeSizes[0][1][1], cubeSizes[1][2][2]],
          [cubeSizes[0][2][0], cubeSizes[0][2][1], cubeSizes[2][2][2]],
        ], 
        [
          [cubeSizes[1][0][0], cubeSizes[1][0][1], cubeSizes[0][1][2]],
          [cubeSizes[1][1][0], cubeSizes[1][1][1], cubeSizes[1][1][2]],
          [cubeSizes[1][2][0], cubeSizes[1][2][1], cubeSizes[2][1][2]],
        ], 
        [
          [cubeSizes[2][0][0], cubeSizes[2][0][1], cubeSizes[0][0][2]],
          [cubeSizes[2][1][0], cubeSizes[2][1][1], cubeSizes[1][0][2]],
          [cubeSizes[2][2][0], cubeSizes[2][2][1], cubeSizes[2][0][2]],
        ],
      ];
      break;
    case 'L':
      cubeSizes = [      
        [
          [cubeSizes[2][0][0], cubeSizes[0][0][1], cubeSizes[0][0][2]],
          [cubeSizes[1][0][0], cubeSizes[0][1][1], cubeSizes[0][1][2]],
          [cubeSizes[0][0][0], cubeSizes[0][2][1], cubeSizes[0][2][2]],
        ], 
        [
          [cubeSizes[2][1][0], cubeSizes[1][0][1], cubeSizes[1][0][2]],
          [cubeSizes[1][1][0], cubeSizes[1][1][1], cubeSizes[1][1][2]],
          [cubeSizes[0][1][0], cubeSizes[1][2][1], cubeSizes[1][2][2]],
        ], 
        [
          [cubeSizes[2][2][0], cubeSizes[2][0][1], cubeSizes[2][0][2]],
          [cubeSizes[1][2][0], cubeSizes[2][1][1], cubeSizes[2][1][2]],
          [cubeSizes[0][2][0], cubeSizes[2][2][1], cubeSizes[2][2][2]],
        ],
      ];
      break;
    case 'T':
      cubeSizes = [
        [
          [cubeSizes[0][2][0], cubeSizes[0][1][0], cubeSizes[0][0][0]],
          [cubeSizes[0][2][1], cubeSizes[0][1][1], cubeSizes[0][0][1]],
          [cubeSizes[0][2][2], cubeSizes[0][1][2], cubeSizes[0][0][2]],
        ], 
        [
          [cubeSizes[1][0][0], cubeSizes[1][0][1], cubeSizes[1][0][2]],
          [cubeSizes[1][1][0], cubeSizes[1][1][1], cubeSizes[1][1][2]],
          [cubeSizes[1][2][0], cubeSizes[1][2][1], cubeSizes[1][2][2]],
        ], 
        [
          [cubeSizes[2][0][0], cubeSizes[2][0][1], cubeSizes[2][0][2]],
          [cubeSizes[2][1][0], cubeSizes[2][1][1], cubeSizes[2][1][2]],
          [cubeSizes[2][2][0], cubeSizes[2][2][1], cubeSizes[2][2][2]],
        ],
      ];
      break;
    case 'D':
      cubeSizes = [
        [
          [cubeSizes[0][0][0], cubeSizes[0][0][1], cubeSizes[0][0][2]],
          [cubeSizes[0][1][0], cubeSizes[0][1][1], cubeSizes[0][1][2]],
          [cubeSizes[0][2][0], cubeSizes[0][2][1], cubeSizes[0][2][2]],
        ], 
        [
          [cubeSizes[1][0][0], cubeSizes[1][0][1], cubeSizes[1][0][2]],
          [cubeSizes[1][1][0], cubeSizes[1][1][1], cubeSizes[1][1][2]],
          [cubeSizes[1][2][0], cubeSizes[1][2][1], cubeSizes[1][2][2]],
        ], 
        [
          [cubeSizes[2][0][2], cubeSizes[2][1][2], cubeSizes[2][2][2]],
          [cubeSizes[2][0][1], cubeSizes[2][1][1], cubeSizes[2][2][1]],
          [cubeSizes[2][0][0], cubeSizes[2][1][0], cubeSizes[2][2][0]],
        ],
      ];
      break;
  }
}

const rotateSize = (way, size) => {
  if (!occupied.value) {
    for (let mesh of getSizeModelsArr(getSizeArr(size))) rotatorGroup.add(mesh);
    console.log(getSizeModelsArr(getSizeArr(size)));
    let axis = null;
    if (size == 'F' || size == 'B') axis = 'Z';
    else if (size == 'R' || size == 'L') axis = 'X';
    else if (size == 'T' || size == 'D') axis = 'Y';
    let deg = null;
    switch (size) {
      case 'F': case 'R': case 'T':
        deg = 90;
        break;
      case 'B': case 'L': case 'D':
        deg = -90;
        break;
    }
    if (way == 'left') {
      rotator(deg, size);
      for (let i = 0; i < 3; i++) rewriteCubeSizes(size);
    } else if (way == 'right') {
      rotator(-deg, size);
      rewriteCubeSizes(size);
    }
    console.log(cubeSizes);
  }
}

const clearRotatorGroup = size => {
  getSizeModelsArr(getSizeArr(size)).forEach((mesh) => {
    let position = new THREE.Vector3();
    let quaternion = new THREE.Quaternion();
    mesh.getWorldPosition(position);
    mesh.getWorldQuaternion(quaternion);
    rotatorGroup.remove(mesh);
    model.add(mesh);
    mesh.position.copy(position);
    mesh.quaternion.copy(quaternion);
  });
  rotatorGroup.quaternion.x = 0;
  rotatorGroup.quaternion.y = 0;
  rotatorGroup.quaternion.z = 0;
}

onMounted(() => {
  scene = new THREE.Scene();
  const fov = 50;
  const aspect = window.innerWidth / window.innerHeight;
  const near = 0.1;
  const far = 1000;

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
  camera.position.z = 5;
  camera.position.x = 5;
  camera.position.y = 5;

  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;

  addLights();
  loadModel();
  animate();

  R1.onclick = () => {
    rotateSize('left', 'R');
  }
  R2.onclick = () => {
    rotateSize('right', 'R');
  }
  L1.onclick = () => {
    rotateSize('left', 'L');
  }
  L2.onclick = () => {
    rotateSize('right', 'L');
  }
  F1.onclick = () => {
    rotateSize('left', 'F');
  }
  F2.onclick = () => {
    rotateSize('right', 'F');
  }
  B1.onclick = () => {
    rotateSize('left', 'B');
  }
  B2.onclick = () => {
    rotateSize('right', 'B');
  }
  T1.onclick = () => {
    rotateSize('left', 'T');
  }
  T2.onclick = () => {
    rotateSize('right', 'T');
  }
  D1.onclick = () => {
    rotateSize('left', 'D');
  }
  D2.onclick = () => {
    rotateSize('right', 'D');
  }
});

const loadModel = () => {
  const loader = new GLTFLoader();
  loader.load('/src/assets/models/cube.glb', (gltf) => {
    model = gltf.scene;
    scene.add(model);
    model.add(rotatorGroup);
    rotatorGroup.scale.set(1, 1, 1);


    // let watcher = watch(occupied, (old, neww) => {
    //   if (!old && neww) {
    //     console.log(123);
    //     rotator(360);
    //     watcher.stop();
    //   }
    // });

  }, undefined, (error) => {
    console.error(error);
  });
}

const addLights = () => {
  // Направленный свет
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.7);
  directionalLight.position.set(20, 30, 40);
  scene.add(directionalLight);

  // Окружающий свет
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.3);
  scene.add(ambientLight);
}

const animate = () => {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}

const degToRad = deg => {
  return Math.PI / 180 * deg;
}

const rotator = (deg, size, recurs = false) => {
  if (!occupied.value || recurs) {
    occupied.value = true;
    let axis = null;
    if (size == 'F' || size == 'B') axis = 'Z';
    else if (size == 'R' || size == 'L') axis = 'X';
    else if (size == 'T' || size == 'D') axis = 'Y';
    let currRotate = 1;
    if (deg < 0) currRotate = -1;
    else currRotate = 1;
    if (axis == 'X') rotatorGroup.rotation.x += degToRad(1) * currRotate;
    else if (axis == 'Y') rotatorGroup.rotation.y += degToRad(1) * currRotate;
    else if (axis == 'Z') rotatorGroup.rotation.z += degToRad(1) * currRotate;
    if (deg != -1 && deg != 1) {
      setTimeout(() => rotator(deg - currRotate, size, true), 1);
    } else {
      occupied.value = false;
      clearRotatorGroup(size);
    }
  }
}
</script>

<style scoped>
  button {
    padding: 20px;
  }
</style>