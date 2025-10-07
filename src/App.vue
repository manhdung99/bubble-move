<template>
  <div id="app" class="relative w-screen h-screen overflow-hidden">
    <canvas ref="space" class="absolute top-0 left-0 w-full h-full"></canvas>

    <img v-for="(bubble, index) in bubbles" :key="bubble.id" :src="bubble.img"
      class="absolute rounded-full shadow-lg avatar" :style="{
        left: bubble.x + 'vw',
        width: bubble.size + 'px',
        height: bubble.size + 'px',
        animationDuration: bubble.duration + 's',
        animationDelay: bubble.delay + 's',
      }" @animationend="resetBubble(index)" />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

const MAX_AVATARS = 80;
const bubbles = ref([]);
let idCounter = 0;

const avatarUrls = Array.from({ length: 40 }, (_, i) => `/avatar/${i + 1}.jpg`);

function createAvatar() {
  const { minSize, maxSize } = getConfigByDevice();
  return {
    id: idCounter++,
    x: Math.random() * 100,
    size: Math.round(minSize + Math.random() * (maxSize - minSize)),
    duration: 15 + Math.random() * 20,
    delay: Math.random() * 15,
    img: avatarUrls[Math.floor(Math.random() * avatarUrls.length)],
  };
}

function resetBubble(index) {
  bubbles.value[index] = createAvatar();
}

onMounted(() => {
  const canvas = document.querySelector("#app canvas");
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.set(0, 50, 150);

  const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setClearColor(0x000010);

  const ambient = new THREE.AmbientLight(0xffffff, 0.3);
  scene.add(ambient);

  const textureLoader = new THREE.TextureLoader();
  const sunTexture = textureLoader.load("/planet/2k_sun.jpg");
  const sunMaterial = new THREE.MeshBasicMaterial({ map: sunTexture });
  const sun = new THREE.Mesh(new THREE.SphereGeometry(10, 64, 64), sunMaterial);
  // scene.add(sun);

  // const sunLight = new THREE.PointLight(0xffcc33, 2.5, 500);
  // scene.add(sunLight);

  const planetData = [
    { name: "Mercury", size: 2, dist: 20, speed: 0.004, texture: "/planet/2k_mercury.jpg" },
    { name: "Venus", size: 3, dist: 30, speed: 0.003, texture: "/planet/2k_venus_surface.jpg" },
    { name: "Earth", size: 3.5, dist: 40, speed: 0.0025, texture: "/planet/2k_earth_daymap.jpg" },
    { name: "Mars", size: 3, dist: 50, speed: 0.002, texture: "/planet/2k_mars.jpg" },
    { name: "Jupiter", size: 7, dist: 70, speed: 0.0015, texture: "/planet/2k_jupiter.jpg" },
    { name: "Saturn", size: 6, dist: 90, speed: 0.0012, texture: "/planet/2k_saturn.jpg" },
    { name: "Uranus", size: 4, dist: 110, speed: 0.0009, texture: "/planet/2k_uranus.jpg" },
    { name: "Neptune", size: 4, dist: 130, speed: 0.0007, texture: "/planet/2k_neptune.jpg" },
  ];

  const planets = planetData.map((p) => {
    const tex = textureLoader.load(p.texture);
    const mat = new THREE.MeshStandardMaterial({ map: tex, roughness: 1 });
    const mesh = new THREE.Mesh(new THREE.SphereGeometry(p.size, 32, 32), mat);
    mesh.position.x = p.dist;
    // scene.add(mesh);
    return { mesh, ...p, angle: Math.random() * Math.PI * 2 };
  });

  const starGeometry = new THREE.BufferGeometry();
  const starVertices = [];
  for (let i = 0; i < 2000; i++) {
    const x = (Math.random() - 0.5) * 2000;
    const y = (Math.random() - 0.5) * 2000;
    const z = (Math.random() - 0.5) * 2000;
    starVertices.push(x, y, z);
  }
  starGeometry.setAttribute("position", new THREE.Float32BufferAttribute(starVertices, 3));
  const starMaterial = new THREE.PointsMaterial({ color: 0xffffff, size: 1 });
  const stars = new THREE.Points(starGeometry, starMaterial);
  scene.add(stars);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.autoRotate = true;
  controls.autoRotateSpeed = 0.5;

  window.addEventListener("resize", () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });

  function animate() {
    requestAnimationFrame(animate);
    planets.forEach((p) => {
      p.angle += p.speed;
      p.mesh.position.x = Math.cos(p.angle) * p.dist;
      p.mesh.position.z = Math.sin(p.angle) * p.dist;
      p.mesh.rotation.y += 0.01;
    });
    sun.rotation.y += 0.002;
    controls.update();
    renderer.render(scene, camera);
  }

  animate();

  for (let i = 0; i < MAX_AVATARS; i++) {
    bubbles.value.push(createAvatar());
  }
});

function getDeviceType() {
  const width = window.innerWidth;
  if (width < 768) return "mobile";
  if (width < 1024) return "tablet";
  return "desktop";
}

let deviceType = getDeviceType();
window.addEventListener("resize", () => {
  deviceType = getDeviceType();
});

function getConfigByDevice() {
  if (deviceType === "mobile") {
    return { max: 50, minSize: 20, maxSize: 60 };
  } else if (deviceType === "tablet") {
    return { max: 50, minSize: 40, maxSize: 80 };
  } else {
    return { max: 100, minSize: 40, maxSize: 120 };
  }
}

onBeforeUnmount(() => {
  bubbles.value = [];
});
</script>


<style scoped>
.avatar {
  position: absolute;
  bottom: -150px;
  object-fit: cover;
  border-radius: 50%;
  animation-name: rise;
  animation-timing-function: linear;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  will-change: transform, opacity;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.25);
}

@keyframes rise {
  0% {
    transform: translateY(-10vh);
    opacity: 0;
  }

  10% {
    opacity: 1;
  }

  90% {
    opacity: 1;
  }

  100% {
    transform: translateY(-100vh);
    opacity: 0;
  }
}
</style>
