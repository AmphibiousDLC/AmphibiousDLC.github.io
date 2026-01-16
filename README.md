<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Wooly Warfare (Spherical Sheep)</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<style>
body { margin:0; overflow:hidden; background:#222; font-family:sans-serif; }
#ui {
  position:fixed; top:10px; left:10px;
  color:white; z-index:10;
}
#menu {
  position:fixed; right:0; top:0;
  width:170px; height:100%;
  background:#333; color:white;
  padding:8px; z-index:10;
}
button { width:100%; margin:6px 0; padding:6px; }
</style>
</head>
<body>

<div id="ui">
🧶 Wool: <span id="wool">50</span><br>
🔁 Round: <span id="round">1</span>
</div>

<div id="menu">
<b>Towers</b>
<button onclick="select('Cube')">Cube (50)</button>
<button onclick="select('Ram')">Ram (120)</button>
<button onclick="select('Grazer')">Grazer (150)</button>
<button onclick="select('Shearer')">Shearer (250)</button>
<button onclick="select('Goat')">Goat (500)</button>
<hr>
<button onclick="startRound()">▶ Start Round</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152/build/three.min.js"></script>
<script>
/* ================= SETUP ================= */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x6a8f4e);

const camera = new THREE.PerspectiveCamera(60, innerWidth/innerHeight, 0.1, 200);
camera.position.set(18,20,25);
camera.lookAt(0,0,0);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

window.addEventListener("resize",()=>{
  camera.aspect = innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});

/* ================= LIGHTING ================= */
scene.add(new THREE.AmbientLight(0xffffff,0.6));
const sun = new THREE.DirectionalLight(0xffffff,0.6);
sun.position.set(20,30,10);
scene.add(sun);

/* ================= GROUND ================= */
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(60,60),
  new THREE.MeshStandardMaterial({color:0x5f7f45})
);
ground.rotation.x = -Math.PI/2;
scene.add(ground);

/* ================= GLOBALS ================= */
let wool = 50;
let round = 1;
let selected = "Cube";
let placing = false;
let lastTap = 0;

const towers = [];
const enemies = [];

/* ================= PATH ================= */
const path = [
  new THREE.Vector3(-25,0,-20),
  new THREE.Vector3(-10,0,-20),
  new THREE.Vector3(-10,0,10),
  new THREE.Vector3(15,0,10),
  new THREE.Vector3(15,0,-15),
  new THREE.Vector3(25,0,-15)
];

/* ================= SPHERICAL SHEEP MODEL ================= */
function sphericalSheep(woolColor){
  const g = new THREE.Group();

  const wool = new THREE.Mesh(
    new THREE.SphereGeometry(0.9,16,16),
    new THREE.MeshStandardMaterial({color:woolColor})
  );
  wool.position.y = 1.1;
  g.add(wool);
  g.body = wool;

  const face = new THREE.Mesh(
    new THREE.SphereGeometry(0.35,12,12),
    new THREE.MeshStandardMaterial({color:0x444444})
  );
  face.position.set(0,1.05,0.75);
  g.add(face);

  for(let x of [-0.45,0.45]){
    for(let z of [-0.45,0.45]){
      const leg = new THREE.Mesh(
        new THREE.CylinderGeometry(0.12,0.12,0.6),
        new THREE.MeshStandardMaterial({color:0x444444})
      );
      leg.position.set(x,0.3,z);
      g.add(leg);
    }
  }
  return g;
}

/* ================= HORNS (RESTORED) ================= */
function addRamHorns(parent){
  const mat = new THREE.MeshStandardMaterial({color:0x8b5a2b});
  for(let s of [-1,1]){
    const horn = new THREE.Mesh(
      new THREE.TorusGeometry(0.5,0.15,10,20),
      mat
    );
    horn.rotation.y = Math.PI/2;
    horn.position.set(0.8*s,1.2,0);
    parent.add(horn);
  }
}

function addGoatHorns(parent){
  const mat = new THREE.MeshStandardMaterial({color:0x6e4b2a});
  for(let s of [-1,1]){
    const horn = new THREE.Mesh(
      new THREE.TorusGeometry(1.1,0.28,12,24),
      mat
    );
    horn.rotation.y = Math.PI/2;
    horn.position.set(1.2*s,1.5,0);
    parent.add(horn);
  }
}

/* ================= TOWERS ================= */
function select(t){ selected = t; placing = true; }

function createTower(type,pos){
  let cost=0, model;

  if(type==="Cube"){ cost=50; model=sphericalSheep(0xdddddd); }

  if(type==="Ram"){
    cost=120;
    model=sphericalSheep(0xdddddd);
    addRamHorns(model);
  }

  if(type==="Grazer"){
    cost=150;
    model=new THREE.Mesh(
      new THREE.BoxGeometry(2,1.5,2),
      new THREE.MeshStandardMaterial({color:0x4f7f3a})
    );
  }

  if(type==="Shearer"){
    cost=250;
    model=new THREE.Mesh(
      new THREE.CylinderGeometry(0.4,0.4,3),
      new THREE.MeshStandardMaterial({color:0x888888})
    );
  }

  if(type==="Goat"){
    cost=500;
    model=sphericalSheep(0xc9b28c);
    addGoatHorns(model);
  }

  if(wool < cost) return;
  wool -= cost;
  document.getElementById("wool").textContent = wool;

  model.position.copy(pos);
  towers.push(model);
  scene.add(model);
}

/* ================= ENEMIES (HEALTH ×4 ONLY) ================= */
function spawnEnemy(type){
  let color=0xcc3333, hp=20;

  if(type==="fast"){ color=0x3366ff; hp=12; }
  if(type==="tank"){ color=0x992222; hp=80; }
  if(type==="boss"){ color=0xcc0000; hp=500; }

  hp *= 4; // <<< ONLY CHANGE YOU ASKED FOR

  const e = sphericalSheep(color);
  e.hp = hp;
  e.max = hp;
  e.pathIndex = 0;
  e.speed = type==="fast"?0.12:type==="tank"?0.04:type==="boss"?0.03:0.06;
  e.position.copy(path[0]);
  enemies.push(e);
  scene.add(e);

  const bar = new THREE.Mesh(
    new THREE.PlaneGeometry(1.4,0.18),
    new THREE.MeshBasicMaterial({color:0x00ff00})
  );
  bar.position.set(0,2.2,0);
  e.add(bar);
  e.hpBar = bar;
}

/* ================= ROUNDS ================= */
function startRound(){
  let count = 3 + round;
  for(let i=0;i<count;i++){
    setTimeout(()=>spawnEnemy("normal"), i*800);
  }
  if(round>=5) spawnEnemy("fast");
  if(round>=10) spawnEnemy("tank");
  if(round>=20 && round%10===0) spawnEnemy("boss");

  document.getElementById("round").textContent = round;
  round++;
}

/* ================= PLACEMENT ================= */
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();

renderer.domElement.addEventListener("pointerdown",e=>{
  const now = Date.now();
  if(now - lastTap < 300 && placing){
    pointer.x = (e.clientX/innerWidth)*2 - 1;
    pointer.y = -(e.clientY/innerHeight)*2 + 1;
    raycaster.setFromCamera(pointer,camera);
    const hit = raycaster.intersectObject(ground);
    if(hit.length) createTower(selected, hit[0].point);
  }
  lastTap = now;
});

/* ================= LOOP ================= */
function animate(){
  requestAnimationFrame(animate);

  enemies.forEach(e=>{
    const target = path[e.pathIndex+1];
    if(!target) return;
    const dir = target.clone().sub(e.position);
    if(dir.length()<0.1) e.pathIndex++;
    else{
      dir.normalize();
      e.position.add(dir.multiplyScalar(e.speed));
      e.lookAt(target);
    }
    e.hpBar.scale.x = e.hp / e.max;
  });

  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
