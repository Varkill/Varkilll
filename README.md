<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Stable Energy Sphere</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: black;
  }
  canvas {
    display: block;
  }
</style>
</head>
<body>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
const scene = new THREE.Scene();

const camera = new THREE.PerspectiveCamera(
  70,
  window.innerWidth / window.innerHeight,
  0.1,
  100
);
camera.position.z = 5.3;

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// ================= НАСТРОЙКИ =================
const COUNT = 6000;
const RADIUS = 2.3;
const INNER_MOVE = 0.12;   // СИЛА движения ВНУТРИ
// ============================================

const positions = new Float32Array(COUNT * 3);
const base = new Float32Array(COUNT * 3);
const colors = new Float32Array(COUNT * 3);

// Заполняем ОБЪЁМ шара
for (let i = 0; i < COUNT; i++) {
  const u = Math.random();
  const v = Math.random();
  const w = Math.random();

  const theta = 2 * Math.PI * u;
  const phi = Math.acos(2 * v - 1);
  const r = RADIUS * Math.cbrt(w);

  const x = r * Math.sin(phi) * Math.cos(theta);
  const y = r * Math.sin(phi) * Math.sin(theta);
  const z = r * Math.cos(phi);

  positions.set([x, y, z], i * 3);
  base.set([x, y, z], i * 3);

  if (Math.random() > 0.5) {
    colors.set([0.15, 1.0, 0.35], i * 3); // зелёный
  } else {
    colors.set([1.0, 0.15, 0.7], i * 3); // розовый
  }
}

const geometry = new THREE.BufferGeometry();
geometry.setAttribute("position", new THREE.BufferAttribute(positions, 3));
geometry.setAttribute("color", new THREE.BufferAttribute(colors, 3));

const material = new THREE.PointsMaterial({
  size: 0.018,
  vertexColors: true,
  transparent: true,
  opacity: 0.95
});

const sphere = new THREE.Points(geometry, material);
scene.add(sphere);

// ================= АНИМАЦИЯ =================
function animate(t) {
  requestAnimationFrame(animate);

  const time = t * 0.001;
  const pos = geometry.attributes.position.array;

  for (let i = 0; i < COUNT; i++) {
    const ix = i * 3;

    const bx = base[ix];
    const by = base[ix + 1];
    const bz = base[ix + 2];

    // направление от центра
    const len = Math.sqrt(bx*bx + by*by + bz*bz) || 1;
    const nx = bx / len;
    const ny = by / len;
    const nz = bz / len;

    // МЯГКОЕ внутреннее движение
    const flow =
      Math.sin(time + bx * 1.2) *
      Math.sin(time * 0.8 + by * 1.1) *
      Math.sin(time * 1.1 + bz * 0.9);

    pos[ix]     = bx + nx * flow * INNER_MOVE;
    pos[ix + 1] = by + ny * flow * INNER_MOVE;
    pos[ix + 2] = bz + nz * flow * INNER_MOVE;
  }

  geometry.attributes.position.needsUpdate = true;

  // Медленное вращение всей сферы
  sphere.rotation.y += 0.001;
  sphere.rotation.x += 0.0006;

  renderer.render(scene, camera);
}

animate();

// ================= РЕСАЙЗ =================
window.addEventListener("resize", () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>

</body>
</html>
