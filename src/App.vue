<script setup lang="ts">
import * as THREE from 'three';
import { onMounted, onUnmounted, ref } from 'vue';
import RepoButton from './components/RepoButton.vue';
import TitleBanner from './components/TitleBanner.vue';

// Déclarer les variables
const threeMount = ref(null);
let scene: any, camera: any, renderer: any;

let planets: any[] = [];
let sun;

const planetData = [
  { name: "Mercury", distance: 0.39, speed: 4.74, texture: '/textures/8k_mercury.jpg', size: 0.38 },
  { name: "Venus", distance: 0.72, speed: 3.5, texture: '/textures/4k_venus_atmosphere.jpg', size: 0.95 },
  { name: "Earth", distance: 1, speed: 2.98, texture: '/textures/8k_earth_daymap.jpg', size: 1.00 },
  { name: "Mars", distance: 1.52, speed: 2.41, texture: '/textures/8k_mars.jpg', size: 0.53 },
  { name: "Jupiter", distance: 5.20, speed: 1.31, texture: '/textures/8k_jupiter.jpg', size: 11.21 },
  { name: "Uranus", distance: 19.22, speed: 0.43, texture: '/textures/2k_uranus.jpg', size: 4.01 },
  { name: "Neptune", distance: 30.05, speed: 0.34, texture: '/textures/2k_neptune.jpg', size: 3.88 }
];

onMounted(() => {
  initThree();
  animate();
  window.addEventListener('resize', onWindowResize, false); // Ajouter l'écouteur après l'initialisation
})

onUnmounted(() => {
  window.removeEventListener('resize', onWindowResize); // Nettoyer l'écouteur
  if (renderer) {
    renderer.dispose();
  }
});

function initThree() {
  // Créer la scène
  scene = new THREE.Scene();

  // Configurer le fond avec une image
  const loader = new THREE.TextureLoader();
  loader.load('/textures/8k_stars.jpg', function(texture: any) {
    const backgroundAspect = texture.image.width / texture.image.height;
    const windowAspect = window.innerWidth / window.innerHeight;
    texture.offset.x = backgroundAspect > windowAspect ? (1 - windowAspect / backgroundAspect) / 2 : 0;
    texture.repeat.x = backgroundAspect > windowAspect ? windowAspect / backgroundAspect : 1;
    scene.background = texture;
  })

  // Ajouter une caméra
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  // camera.position.z = 400;
  camera.position.set(0, 30, 50); // Ajustez selon le meilleur angle visuel pour votre scène
  camera.lookAt(0, 0, 0); // Oriente la caméra vers le centre de la scène

  // Configurer le rendu
  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  threeMount.value.appendChild(renderer.domElement);

  // Création du soleil
  const sunGeometry = new THREE.SphereGeometry(2, 32, 32);
  const sunMaterial = new THREE.MeshBasicMaterial({ map: new THREE.TextureLoader().load('/textures/8k_sun.jpg') });
  sun = new THREE.Mesh(sunGeometry, sunMaterial);
  scene.add(sun);

  // Initialiser les planètes
  planetData.forEach(p => createPlanet(p.size, p.texture, p.distance * 8, p.speed));

  // Redimensionner le canva lorsque la fenêtre change de taille
  window.addEventListener('resize', onWindowResize, false);
}

function createPlanet(size, texturePath, distance, speed) {
  const geometry = new THREE.SphereGeometry(size, 32, 32);
  const texture = new THREE.TextureLoader().load(texturePath);
  const material = new THREE.MeshBasicMaterial({ map: texture });
  const planet = new THREE.Mesh(geometry, material);
  planet.userData = { distance, speed, angle: 0 };  // angle pour gérer la rotation orbitale
  planet.position.x = distance;
  scene.add(planet);
  planets.push(planet);
}

function animate() {
  requestAnimationFrame(animate);

  // Rotation du soleil et des planètes
  sun.rotation.y += 0.004;

  // Mouvement orbital des planètes
  planets.forEach(planet => {
    planet.userData.angle += planet.userData.speed / 1000;  // La vitesse orbitale
    planet.position.x = planet.userData.distance * Math.cos(planet.userData.angle);
    planet.position.z = planet.userData.distance * Math.sin(planet.userData.angle);
    planet.rotation.y += 0.01;  // Rotation de la planète sur elle-même
  });

  renderer.render(scene, camera);
}

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);

  // Ajout pour le fond
  if (scene.background && scene.background.image) {
    const backgroundAspect = scene.background.image.width / scene.background.image.height;
    const windowAspect = window.innerWidth / window.innerHeight;
    scene.background.offset.x = backgroundAspect > windowAspect ? (1 - windowAspect / backgroundAspect) / 2 : 0;
    scene.background.repeat.x = backgroundAspect > windowAspect ? windowAspect / backgroundAspect : 1;
  }
}

function zoomIn() {
  camera.fov = Math.max(camera.fov - 5, 10); // Diminue le fov pour zoomer, limite à 10
  camera.updateProjectionMatrix();
}

function zoomOut() {
  camera.fov = Math.min(camera.fov + 5, 100); // Augmente le fov pour dézoomer, limite à 100
  camera.updateProjectionMatrix();
}
</script>

<template>
  <div id="main" ref="threeMount"></div>

  <div id="btn">
    <button class="btn" @click="zoomIn">+</button>
    <button class="btn" @click="zoomOut">-</button>
  </div>

  <TitleBanner />

  <RepoButton />

</template>

<style scoped>
#main {
  width: 100%;
  height: 100vh;
}

#btn {
  position: absolute;
  bottom: 2%;
  right: 2%;
  display: flex;
  flex-direction: column;
}

.btn {
  padding: 10px 20px;
  text-transform: uppercase;
  border-radius: 8px;
  font-size: 17px;
  font-weight: 500;
  color: #ffffff80;
  text-shadow: none;
  background: transparent;
  cursor: pointer;
  box-shadow: transparent;
  border: 1px solid #ffffff80;
  transition: 0.5s ease;
  user-select: none;
}

.btn:first-child {
  margin-bottom: 15%;
}

.btn:hover,
:focus {
  color: #ffffff;
  background: #008cff;
  border: 1px solid #008cff;
  text-shadow: 0 0 5px #ffffff, 0 0 10px #ffffff, 0 0 20px #ffffff;
  box-shadow: 0 0 5px #008cff, 0 0 20px #008cff, 0 0 50px #008cff,
    0 0 100px #008cff;
}
</style>