<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Wooly Warfare</title>

<style>
html,body{
  margin:0;
  overflow:hidden;
  touch-action:none;
  font-family:sans-serif;
}
#ui{
  position:absolute;
  top:10px;
  left:10px;
  background:#fff;
  padding:8px;
  border-radius:8px;
  z-index:10;
}
#menu{
  position:absolute;
  right:10px;
  top:50%;
  transform:translateY(-50%);
  display:flex;
  flex-direction:column;
  gap:6px;
  z-index:10;
}
.btn{
  background:#fff;
  border:2px solid #777;
  border-radius:8px;
  padding:6px;
  width:200px;
}
.btn.selected{
  border-color:orange;
  background:#fff3dd;
}
</style>
</head>

<body>

<div id="ui">
🧶 Wool: <span id="wool">50</span><br>
🔁 Round: <span id="round">0</span><br>
<button id="startRound">Start Round</button>
</div>

<div id="menu"></div>

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
 grassLight:new THREE.MeshBasicMaterial({color:0x4caf50}),
 path:new THREE.MeshBasicMaterial({color:0x8b6b3e}),
 wool:new THREE.MeshBasicMaterial({color:0xdddddd}),
 goat:new THREE.MeshBasicMaterial({color:0xc8b07a}),
 enemy:new THREE.MeshBasicMaterial({color:0xcc3333}),
 fast:new THREE.MeshBasicMaterial({color:0x3366ff}),
 tank:new THREE.MeshBasicMaterial({color:0x992222}),
 dark:new THREE.MeshBasicMaterial({color:0x444444}),
 horn:new THREE.MeshBasicMaterial({color:0x8b5a2b}),
 crown:new THREE.MeshBasicMaterial({color:0xffcc00})
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

for(let i=0;i<path.length-1;i++){
 const a=path[i],b=path[i+1];
 const len=a.distanceTo(b);
 for(let j=0;j<len/4;j++){
  const t=new THREE.Mesh(new THREE.BoxGeometry(4,0.2,4),M.path);
  t.position.copy(a.clone().lerp(b,j/(len/4)));
  t.position.y=0.1;
  scene.add(t);
 }
}

/* ================= BASE SHEEP ================= */
function baseSheep(mat){
 const g=new THREE.Group();

 const body=new THREE.Mesh(
  new THREE.SphereGeometry(1.3,8,8),mat
 );
 body.position.y=1.4;
 g.add(body);
 g.body=body;

 const face=new THREE.Mesh(
  new THREE.BoxGeometry(.6,.6,.6),M.dark
 );
 face.position.set(0,1.4,1.2);
 g.add(face);

 [-.5,.5].forEach(x=>{
  [-.5,.5].forEach(z=>{
   const leg=new THREE.Mesh(
    new THREE.CylinderGeometry(.12,.12,.8),M.dark
   );
   leg.position.set(x,.4,z);
   g.add(leg);
  });
 });

 g.attackAnim=0;
 return g;
}

/* ================= HORNS ================= */
function addRamHorns(g){
 for(let s of [-1,1]){
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(.6,.18,10,20),M.horn
  );
  h.rotation.y=Math.PI/2;
  h.position.set(.9*s,1.6,0);
  g.add(h);
 }
}
function addGoatHorns(g){
 for(let s of [-1,1]){
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(1.2,.3,12,24),M.horn
  );
  h.rotation.y=Math.PI/2;
  h.position.set(1.3*s,2.1,0);
  g.add(h);
 }
}

/* ================= TOWERS ================= */
function cube(){const g=baseSheep(M.wool);g.damage=5;g.range=12;return g;}
function ram(){const g=baseSheep(M.wool);g.damage=10;g.range=14;addRamHorns(g);return g;}
function goat(){
 const g=baseSheep(M.goat);
 g.damage=150;g.range=16;
 g.scale.set(1.2,1.2,1.2);
 addGoatHorns(g);
 return g;
}
function grazer(){
 const g=new THREE.Group();
 const c=new THREE.Mesh(new THREE.BoxGeometry(2.6,1.2,2.6),M.grassLight);
 c.position.y=.6;
 g.add(c);
 const s=baseSheep(M.wool);
 s.scale.set(.5,.5,.5);
 s.position.y=1.5;
 g.add(s);
 g.damage=8;g.range=14;
 return g;
}
function shearer(){const g=new THREE.Group();g.support=true;return g;}

const TOWERS={
 Cube:{cost:10,build:cube},
 Ram:{cost:25,build:ram},
 Grazer:{cost:100,build:grazer},
 Shearer:{cost:250,build:shearer},
 Goat:{cost:500,build:goat}
};

/* ================= UI ================= */
let wool=50,selected="Cube";
const menu=document.getElementById("menu");

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

/* ================= PLACEMENT ================= */
const ray=new THREE.Raycaster();
const pt=new THREE.Vector2();
let taps=0,timer;
const towers=[];

renderer.domElement.addEventListener("pointerdown",e=>{
 taps++;
 if(taps===1){
  timer=setTimeout(()=>taps=0,300);
  return;
 }
 clearTimeout(timer);taps=0;

 const rect=renderer.domElement.getBoundingClientRect();
 pt.x=((e.clientX-rect.left)/rect.width)*2-1;
 pt.y=-((e.clientY-rect.top)/rect.height)*2+1;

 ray.setFromCamera(pt,camera);
 const hit=ray.intersectObject(ground);
 if(!hit.length)return;

 const t=TOWERS[selected];
 if(wool<t.cost)return;

 wool-=t.cost;
 document.getElementById("wool").textContent=wool;

 const u=t.build();
 u.position.set(Math.round(hit[0].point.x),0,Math.round(hit[0].point.z));
 u.cooldown=0;
 towers.push(u);
 scene.add(u);
});

/* ================= ENEMIES ================= */
function makeEnemy(type){
 let mat=M.enemy,hp=80,speed=.045,woolGain=5;
 if(type==="fast"){mat=M.fast;hp=60;speed=.08;woolGain=6;}
 if(type==="tank"){mat=M.tank;hp=240;speed=.025;woolGain=15;}
 if(type==="boss"){hp=1200;speed=.02;woolGain=100;}

 const g=baseSheep(mat);
 g.hp=hp;g.max=hp;g.speed=speed;g.wool=woolGain;g.i=0;

 if(type==="boss"){
  const crown=new THREE.Mesh(new THREE.TorusGeometry(1,.3,8,16),M.crown);
  crown.position.y=2.8;
  g.add(crown);
  g.scale.set(2,2,2);
 }

 const bar=new THREE.Mesh(
  new THREE.PlaneGeometry(2.5,.3),
  new THREE.MeshBasicMaterial({color:0x00ff00})
 );
 bar.position.y=3;
 g.add(bar);
 g.bar=bar;
 return g;
}

const enemies=[];

/* ================= ROUNDS (UPDATED) ================= */
let round=0,queue=[],spawning=false,spawnTimer=0;

document.getElementById("startRound").onclick=()=>{
 if(spawning||enemies.length)return;
 round++;
 document.getElementById("round").textContent=round;

 queue=[...Array(6+round).fill("normal")];

 if(round>=5) queue.push(...Array(round-4).fill("fast"));
 if(round>=10) queue.push(...Array(round-9).fill("tank"));
 if(round>=20 && round%10===0) queue.push("boss");

 spawning=true;
};

/* ================= LOOP ================= */
function animate(){
 requestAnimationFrame(animate);

 if(spawning&&queue.length){
  spawnTimer++;
  if(spawnTimer>50){
   spawnTimer=0;
   const e=makeEnemy(queue.shift());
   e.position.copy(path[0]);
   enemies.push(e);
   scene.add(e);
  }
 }
 if(spawning&&!queue.length&&!enemies.length)spawning=false;

 enemies.forEach(e=>{
  const n=path[e.i+1];
  if(!n)return;
  e.lookAt(n);
  const d=n.clone().sub(e.position);
  if(d.length()<.5)e.i++;
  e.position.add(d.normalize().multiplyScalar(e.speed));
  e.bar.scale.x=e.hp/e.max;
 });

 towers.forEach(t=>{
  if(t.support)return;
  if(t.cooldown>0)t.cooldown--;
  if(t.attackAnim>0)t.attackAnim-=.15;

  const target=enemies.find(e=>e.hp>0&&e.position.distanceTo(t.position)<t.range);
  if(target&&t.cooldown===0){
   t.lookAt(target.position);
   target.hp-=t.damage;
   t.attackAnim=1;
   t.cooldown=40;
  }
  if(t.body){
   const a=Math.max(0,t.attackAnim);
   t.body.position.z=-a*.4;
  }
 });

 for(let i=enemies.length-1;i>=0;i--){
  if(enemies[i].hp<=0){
   wool+=enemies[i].wool;
   document.getElementById("wool").textContent=wool;
   scene.remove(enemies[i]);
   enemies.splice(i,1);
  }
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
