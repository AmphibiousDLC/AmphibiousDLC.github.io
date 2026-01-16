<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport"
  content="width=device-width, height=device-height, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Wooly Warfare</title>

<style>
html, body {
  margin: 0;
  padding: 0;
  overflow: hidden;
  touch-action: none;
}
#ui {
  position: absolute;
  top: 10px;
  left: 10px;
  background: rgba(255,255,255,0.9);
  padding: 10px;
  border-radius: 8px;
  z-index: 10;
}
#menu {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  gap: 6px;
  z-index: 10;
}
.btn {
  background: #fff;
  border: 2px solid #888;
  border-radius: 8px;
  padding: 6px;
  width: 170px;
}
.btn.selected {
  border-color: orange;
}
</style>
</head>

<body>

<div id="ui">
🧶 Wool: <span id="wool">50</span>
</div>

<div id="menu"></div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
/* ===== SCENE ===== */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(60, innerWidth / innerHeight, 0.1, 500);
camera.position.set(40, 50, 60);
camera.lookAt(0, 0, 0);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 1));

/* ===== GROUND ===== */
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(200, 200),
  new THREE.MeshBasicMaterial({ color: 0x2b6f34 })
);
ground.rotation.x = -Math.PI / 2;
scene.add(ground);

/* ===== SHEEP ===== */
function makeSheep() {
  const g = new THREE.Group();
  const body = new THREE.Mesh(
    new THREE.SphereGeometry(1.2, 8, 8),
    new THREE.MeshBasicMaterial({ color: 0xdddddd })
  );
  body.position.y = 1.4;
  g.add(body);
  return g;
}

/* ===== MENU ===== */
let wool = 50;
let selected = "sheep";
const menu = document.getElementById("menu");

const btn = document.createElement("div");
btn.className = "btn selected";
btn.textContent = "SHEEP (10)";
menu.appendChild(btn);

/* ===== RAYCAST (FINAL FIX) ===== */
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();

let tapCount = 0;
let tapTimeout = null;

function getPointer(event) {
  if (event.touches && event.touches.length > 0) {
    return event.touches[0];
  }
  return event;
}

renderer.domElement.addEventListener("pointerdown", e => {
  tapCount++;
  if (tapCount === 1) {
    tapTimeout = setTimeout(() => tapCount = 0, 300);
  } else {
    clearTimeout(tapTimeout);
    tapCount = 0;

    const p = getPointer(e);
    const rect = renderer.domElement.getBoundingClientRect();

    pointer.x = ((p.clientX - rect.left) / rect.width) * 2 - 1;
    pointer.y = -((p.clientY - rect.top) / rect.height) * 2 + 1;

    raycaster.setFromCamera(pointer, camera);
    const hits = raycaster.intersectObject(ground);

    if (hits.length > 0) {
      placeSheep(hits[0].point);
    }
  }
});

function placeSheep(pos) {
  if (wool < 10) return;
  wool -= 10;
  document.getElementById("wool").textContent = wool;

  const s = makeSheep();
  s.position.set(
    Math.round(pos.x),
    0,
    Math.round(pos.z)
  );
  scene.add(s);
}

/* ===== LOOP ===== */
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
animate();

window.addEventListener("resize", () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
</script>
</body>
</html>
