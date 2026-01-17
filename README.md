<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Wooly Warfare</title>

<style>
html,body{margin:0;overflow:hidden;font-family:sans-serif}
#menu{
 position:absolute;right:10px;top:50%;
 transform:translateY(-50%);
 display:flex;flex-direction:column;gap:8px;z-index:10
}
.btn{
 background:#fff;border:2px solid #777;border-radius:8px;
 padding:6px;width:200px;cursor:pointer
}
.btn.selected{border-color:orange;background:#fff3dd}
#info{background:#fff;border-radius:8px;padding:8px;border:2px solid #999}
canvas{display:block}
</style>
</head>

<body>

<div id="menu">
 <div id="towerButtons"></div>
 <div id="info">
 🧶 Wool: <span id="wool">50</span><br>
 🔁 Round: <span id="round">0</span><br><br>
 <button id="startRound" class="btn">Start Round</button>
 </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
/* ================= SCENE ================= */
const scene=new THREE.Scene();
scene.background=new THREE.Color(0x87ceeb);

const camera=new THREE.PerspectiveCamera(60,innerWidth/innerHeight,0.1,500);
camera.position.set(45,55,65);
camera.lookAt(0,0,0);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(devicePixelRatio);
document.body.appendChild(renderer.domElement);
scene.add(new THREE.AmbientLight(0xffffff,1));

/* ================= MATERIALS ================= */
const M={
 grass:new THREE.MeshBasicMaterial({color:0x2a6a32}),
 path:new THREE.MeshBasicMaterial({color:0x8b6b3e}),
 wool:new THREE.MeshBasicMaterial({color:0xdddddd}),
 goat:new THREE.MeshBasicMaterial({color:0xc8b07a}),
 enemy:new THREE.MeshBasicMaterial({color:0xcc3333}),
 fast:new THREE.MeshBasicMaterial({color:0x3366ff}),
 tank:new THREE.MeshBasicMaterial({color:0x992222}),
 dark:new THREE.MeshBasicMaterial({color:0x444444}),
 horn:new THREE.MeshBasicMaterial({color:0x8b5a2b})
};

/* ================= GROUND ================= */
const ground=new THREE.Mesh(new THREE.PlaneGeometry(200,200),M.grass);
ground.rotation.x=-Math.PI/2;
scene.add(ground);

/* ================= PATH ================= */
const path=[
 new THREE.Vector3(-70,0,-40),
 new THREE.Vector3(-70,0,20),
 new THREE.Vector3(40,0,20),
 new THREE.Vector3(40,0,60)
];
const pathBoxes=[];
for(let i=0;i<path.length-1;i++){
 const a=path[i],b=path[i+1];
 const len=a.distanceTo(b);
 for(let j=0;j<len/4;j++){
  const t=new THREE.Mesh(new THREE.BoxGeometry(4,0.2,4),M.path);
  t.position.copy(a.clone().lerp(b,j/(len/4)));
  t.position.y=0.1;
  scene.add(t);
  pathBoxes.push(t);
 }
}

/* ================= BASE SHEEP ================= */
function baseSheep(mat){
 const g=new THREE.Group();
 const body=new THREE.Mesh(new THREE.SphereGeometry(1.3,10,10),mat);
 body.position.y=1.4;
 g.add(body);g.body=body;

 const face=new THREE.Mesh(new THREE.BoxGeometry(.6,.6,.6),M.dark);
 face.position.set(0,1.4,1.2);
 g.add(face);

 return g;
}

/* ================= HORNS ================= */
function addRamHorns(g){
 [-1,1].forEach(s=>{
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(0.6,0.18,10,20),
   M.horn
  );
  h.rotation.y=Math.PI/2;
  h.position.set(0.9*s,1.6,0);
  g.add(h);
 });
}
function addGoatHorns(g){
 [-1,1].forEach(s=>{
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(1.2,0.3,12,24),
   M.horn
  );
  h.rotation.y=Math.PI/2;
  h.position.set(1.3*s,1.9,0);
  g.add(h);
 });
}

/* ================= TOWERS ================= */
function cube(){const g=baseSheep(M.wool);g.damage=5;g.range=12;return g;}
function ram(){const g=baseSheep(M.wool);addRamHorns(g);g.damage=10;g.range=14;return g;}
function goat(){
 const g=baseSheep(M.goat);
 addGoatHorns(g);
 g.damage=150;g.range=16;
 g.scale.set(1.2,1.2,1.2);
 return g;
}
function grazer(){
 const g=baseSheep(M.wool);
 g.damage=16;g.range=14;
 g.shoots=true;
 return g;
}
function shearer(){
 const g=new THREE.Group();
 g.support=true;
 return g;
}

const TOWERS={
 Cube:{cost:10,build:cube,limit:20,count:0},
 Ram:{cost:25,build:ram,limit:5,count:0},
 Grazer:{cost:100,build:grazer,limit:5,count:0},
 Shearer:{cost:250,build:shearer,limit:3,count:0},
 Goat:{cost:500,build:goat,limit:2,count:0}
};

/* ================= UI ================= */
let wool=50,selected="Cube";
const towers=[],menu=document.getElementById("towerButtons");

for(const k in TOWERS){
 const b=document.createElement("div");
 b.className="btn"+(k===selected?" selected":"");
 b.textContent=`${k} (${TOWERS[k].cost})`;
 b.onclick=()=>{
  selected=k;
  document.querySelectorAll(".btn").forEach(x=>x.classList.remove("selected"));
  b.classList.add("selected");
 };
 menu.appendChild(b);
}

/* ================= DOUBLE TAP PLACEMENT ================= */
const ray=new THREE.Raycaster(),pt=new THREE.Vector2();
let lastTap=0;

renderer.domElement.addEventListener("pointerdown",e=>{
 const now=Date.now();
 if(now-lastTap<300){
  const r=renderer.domElement.getBoundingClientRect();
  pt.x=((e.clientX-r.left)/r.width)*2-1;
  pt.y=-((e.clientY-r.top)/r.height)*2+1;
  ray.setFromCamera(pt,camera);
  const hit=ray.intersectObject(ground);
  if(!hit.length)return;

  if(pathBoxes.some(p=>p.position.distanceTo(hit[0].point)<3))return;

  const t=TOWERS[selected];
  if(wool<t.cost||t.count>=t.limit)return;

  wool-=t.cost;document.getElementById("wool").textContent=wool;
  t.count++;

  const u=t.build();
  u.position.set(Math.round(hit[0].point.x),0,Math.round(hit[0].point.z));
  u.cooldown=0;u.anim=0;
  towers.push(u);scene.add(u);
 }
 lastTap=now;
});

/* ================= ENEMIES ================= */
const enemies=[];
function makeEnemy(type){
 let mat=M.enemy,hp=40,speed=.045,woolGain=5;
 if(type==="fast"){mat=M.fast;hp=30;speed=.08;woolGain=6;}
 if(type==="tank"){mat=M.tank;hp=120;speed=.025;woolGain=15;}

 const g=baseSheep(mat);
 g.hp=hp;g.max=hp;g.speed=speed;g.wool=woolGain;g.i=0;

 const bar=new THREE.Mesh(
  new THREE.PlaneGeometry(2.5,.25),
  new THREE.MeshBasicMaterial({color:0x00ff00})
 );
 bar.position.y=3;g.add(bar);g.bar=bar;

 return g;
}

/* ================= ROUNDS ================= */
let round=0,spawning=false,queue=[],spawnTimer=0;
const startBtn=document.getElementById("startRound");

startBtn.onclick=()=>{
 if(spawning)return;
 round++;document.getElementById("round").textContent=round;
 queue=[];
 queue.push(...Array(6+round).fill("normal"));
 if(round>=5)queue.push(...Array(round-4).fill("fast"));
 if(round>=10)queue.push(...Array(round-9).fill("tank"));
 spawning=true;spawnTimer=0;startBtn.disabled=true;
};

/* ================= LOOP ================= */
function animate(){
 requestAnimationFrame(animate);

 if(spawning&&queue.length&&++spawnTimer>50){
  spawnTimer=0;
  const e=makeEnemy(queue.shift());
  e.position.copy(path[0]);
  enemies.push(e);scene.add(e);
 }

 enemies.forEach(e=>{
  const n=path[e.i+1];
  if(!n)return;
  e.lookAt(n.x,1.4,n.z);
  const d=n.clone().sub(e.position);
  if(d.length()<.5)e.i++;
  e.position.add(d.normalize().multiplyScalar(e.speed));
  e.body.position.y=1.4+Math.sin(Date.now()*0.005)*0.1;
  e.bar.scale.x=e.hp/e.max;
 });

 towers.forEach(t=>{
  if(t.support)return;
  if(t.cooldown>0)t.cooldown--;
  const target=enemies.find(e=>e.hp>0&&e.position.distanceTo(t.position)<t.range);
  if(target&&t.cooldown===0){
   t.lookAt(target.position);
   t.anim=6;
   target.hp-=t.damage;
   t.cooldown=40;
  }
  if(t.anim>0){t.anim--;t.position.y=0.2;}
  else t.position.y=0;
 });

 for(let i=enemies.length-1;i>=0;i--){
  if(enemies[i].hp<=0){
   let gain=enemies[i].wool;
   if(towers.some(t=>t.support))gain*=2;
   wool+=gain;document.getElementById("wool").textContent=wool;
   scene.remove(enemies[i]);enemies.splice(i,1);
  }
 }

 if(spawning&&!queue.length&&!enemies.length){
  spawning=false;startBtn.disabled=false;
 }

 renderer.render(scene,camera);
}
animate();

window.addEventListener("resize",()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
