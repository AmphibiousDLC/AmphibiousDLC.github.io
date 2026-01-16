<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Wooly Warfare</title>
<style>
body { margin:0; overflow:hidden; font-family:sans-serif; touch-action:none; }
#ui {
  position:absolute; top:10px; left:10px;
  background:rgba(255,255,255,0.9);
  padding:10px; border-radius:8px;
}
#menu {
  position:absolute; right:10px; top:50%;
  transform:translateY(-50%);
  display:flex; flex-direction:column; gap:6px;
}
.btn {
  background:#fff; border:2px solid #888;
  border-radius:8px; padding:6px;
  width:170px; font-size:13px;
}
.btn.selected { border-color:#ff9900; background:#fff3dd; }
#startBtn {
  margin-top:6px;
  width:100%;
}
</style>
</head>
<body>

<div id="ui">
🧶 Wool: <span id="wool">50</span><br>
🔁 Round: <span id="round">1</span><br>
<button id="startBtn">Start Round</button>
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
document.body.appendChild(renderer.domElement);
scene.add(new THREE.AmbientLight(0xffffff,1));

/* ================= MATERIALS ================= */
const mats={
 grass:new THREE.MeshBasicMaterial({color:0x2b6f34}),
 path:new THREE.MeshBasicMaterial({color:0x8b6b3e}),
 wool:new THREE.MeshBasicMaterial({color:0xdddddd}),
 goat:new THREE.MeshBasicMaterial({color:0xc9b27c}),
 enemy:new THREE.MeshBasicMaterial({color:0xcc3333}),
 fast:new THREE.MeshBasicMaterial({color:0x3366cc}),
 tank:new THREE.MeshBasicMaterial({color:0x770000}),
 boss:new THREE.MeshBasicMaterial({color:0x990000}),
 dark:new THREE.MeshBasicMaterial({color:0x444444}),
 horn:new THREE.MeshBasicMaterial({color:0x8b5a2b}),
 crown:new THREE.MeshBasicMaterial({color:0xffd700}),
 shot:new THREE.MeshBasicMaterial({color:0x44cc44}),
 metal:new THREE.MeshBasicMaterial({color:0xcccccc})
};

/* ================= GROUND ================= */
const ground=new THREE.Mesh(
 new THREE.PlaneGeometry(200,200), mats.grass
);
ground.rotation.x=-Math.PI/2;
scene.add(ground);

/* ================= PATH ================= */
const path=[
 new THREE.Vector3(-50,0,-40),
 new THREE.Vector3(-50,0,20),
 new THREE.Vector3(40,0,20)
];
for(let i=0;i<path.length-1;i++){
 const a=path[i],b=path[i+1];
 const len=a.distanceTo(b);
 for(let j=0;j<=len/4;j++){
  const tile=new THREE.Mesh(
   new THREE.BoxGeometry(4,0.2,4), mats.path
  );
  tile.position.copy(a.clone().lerp(b,j/(len/4)));
  tile.position.y=0.1;
  scene.add(tile);
 }
}

/* ================= MODELS ================= */
function addHorns(g,size){
 [-1,1].forEach(side=>{
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(size,0.15,8,16), mats.horn
  );
  h.position.set(0.6*side,2,0.3);
  h.rotation.y=Math.PI/2;
  g.add(h);
 });
}
function addCrown(g){
 const c=new THREE.Mesh(
  new THREE.CylinderGeometry(0.7,0.7,0.4,8), mats.crown
 );
 c.position.y=3;
 g.add(c);
}
function makeSheep(mat,scale=1){
 const g=new THREE.Group();
 const body=new THREE.Mesh(
  new THREE.SphereGeometry(1.2,8,8), mat
 );
 body.position.y=1.4;
 body.scale.set(scale,scale,scale);
 g.add(body);

 const face=new THREE.Mesh(
  new THREE.BoxGeometry(0.6,0.6,0.6), mats.dark
 );
 face.position.set(0,1.4,1.1);
 g.add(face);

 [-0.4,0.4].forEach(x=>{
  [-0.4,0.4].forEach(z=>{
   const leg=new THREE.Mesh(
    new THREE.CylinderGeometry(0.12,0.12,0.8), mats.dark
   );
   leg.position.set(x,0.4,z);
   g.add(leg);
  });
 });

 g.body=body;
 return g;
}
function makeShearer(){
 const g=new THREE.Group();
 const p=new THREE.Mesh(
  new THREE.CylinderGeometry(0.6,0.6,3), mats.path
 );
 p.position.y=1.5;
 g.add(p);

 const b1=new THREE.Mesh(
  new THREE.BoxGeometry(0.2,1.2,0.05), mats.metal
 );
 b1.position.set(-0.25,3,0);
 b1.rotation.z=0.4;
 const b2=b1.clone();
 b2.position.x=0.25;
 b2.rotation.z=-0.4;
 g.add(b1); g.add(b2);
 return g;
}

/* ================= TOWERS ================= */
const TOWERS={
 cube:{cost:10,dmg:5,range:12,rate:60},
 ram:{cost:25,dmg:10,range:14,rate:50},
 grazer:{cost:100,dmg:8,range:18,rate:45},
 goat:{cost:500,dmg:150,range:16,rate:90},
 shearer:{cost:250,dmg:0,range:0,rate:0}
};

let wool=50, round=1, woolMult=1;
let selected="cube";
const towers=[], enemies=[], shots=[];

/* ================= MENU ================= */
const menu=document.getElementById("menu");
for(const k in TOWERS){
 const t=TOWERS[k];
 const b=document.createElement("div");
 b.className="btn"+(k===selected?" selected":"");
 b.innerHTML=`${k.toUpperCase()}<br>DMG:${t.dmg}<br>COST:${t.cost}`;
 b.onclick=()=>{
  selected=k;
  document.querySelectorAll(".btn").forEach(x=>x.classList.remove("selected"));
  b.classList.add("selected");
 };
 menu.appendChild(b);
}

/* ================= PLACEMENT (DOUBLE TAP) ================= */
let taps=0,timer=null;
const ray=new THREE.Raycaster(),mouse=new THREE.Vector2();
renderer.domElement.addEventListener("pointerdown",e=>{
 taps++;
 if(taps===1){
  timer=setTimeout(()=>taps=0,300);
 }else{
  clearTimeout(timer); taps=0;
  mouse.x=(e.clientX/innerWidth)*2-1;
  mouse.y=-(e.clientY/innerHeight)*2+1;
  ray.setFromCamera(mouse,camera);
  const hit=ray.intersectObject(ground);
  if(hit.length) placeTower(hit[0].point);
 }
});

function placeTower(p){
 const t=TOWERS[selected];
 if(wool<t.cost) return;
 wool-=t.cost;
 document.getElementById("wool").textContent=wool;

 let u;
 if(selected==="shearer"){
  u=makeShearer();
  woolMult=2;
 }else{
  u=makeSheep(
   selected==="goat"?mats.goat:mats.wool,
   selected==="goat"?1.3:1
  );
  if(selected==="ram") addHorns(u,0.6);
  if(selected==="goat") addHorns(u,1.8);
 }

 u.position.set(Math.round(p.x),0,Math.round(p.z));
 u.type=selected; u.cool=0;
 towers.push(u); scene.add(u);
}

/* ================= START ROUND ================= */
let wave=[], spawning=false, spawnIndex=0, spawnTimer=0;
document.getElementById("startBtn").onclick=()=>{
 if(!spawning && enemies.length===0){
  wave=[];
  for(let i=0;i<5+round;i++) wave.push("normal");
  if(round>=5) wave.push("fast");
  if(round>=10) wave.push("tank");
  if(round>=20 && round%10===0) wave.push("boss");
  spawning=true; spawnIndex=0;
 }
};

/* ================= LOOP ================= */
function animate(){
 requestAnimationFrame(animate);

 if(spawning){
  spawnTimer++;
  if(spawnTimer>40 && spawnIndex<wave.length){
   spawnTimer=0;
   spawnEnemy(wave[spawnIndex++]);
  }
  if(spawnIndex>=wave.length) spawning=false;
 }

 enemies.forEach(e=>{
  const t=path[e.pathIndex+1];
  if(!t) return;
  const dir=t.clone().sub(e.position);
  e.lookAt(t.x,1.4,t.z);
  if(dir.length()<0.5) e.pathIndex++;
  e.position.add(dir.normalize().multiplyScalar(e.speed));
  e.bar.scale.x=e.hp/e.max;
 });

 towers.forEach(t=>{
  if(t.type==="shearer") return;
  t.cool--;
  const d=TOWERS[t.type];
  if(t.cool>0) return;
  let target=null,dist=999;
  enemies.forEach(e=>{
   const dd=t.position.distanceTo(e.position);
   if(dd<d.range && dd<dist){dist=dd;target=e;}
  });
  if(target){
   t.lookAt(target.position.x,1.4,target.position.z);
   t.body.scale.y=0.8;
   setTimeout(()=>t.body.scale.y=1,100);
   if(t.type==="grazer") shoot(t,target,d.dmg);
   else target.hp-=d.dmg;
   t.cool=d.rate;
  }
 });

 shots.forEach((s,i)=>{
  const dir=s.target.position.clone().sub(s.position).normalize();
  s.position.add(dir.multiplyScalar(0.5));
  if(s.position.distanceTo(s.target.position)<1){
   s.target.hp-=s.dmg;
   scene.remove(s); shots.splice(i,1);
  }
 });

 enemies.forEach((e,i)=>{
  if(e.hp<=0){
   wool+=5*woolMult;
   document.getElementById("wool").textContent=wool;
   scene.remove(e); enemies.splice(i,1);
   if(enemies.length===0 && !spawning){
    round++;
    document.getElementById("round").textContent=round;
   }
  }
 });

 renderer.render(scene,camera);
}

function spawnEnemy(type){
 let mat=mats.enemy,hp=20,speed=0.07,scale=1;
 if(type==="fast"){mat=mats.fast;hp=15;speed=0.12;}
 if(type==="tank"){mat=mats.tank;hp=80;speed=0.04;}
 if(type==="boss"){mat=mats.boss;hp=500;speed=0.03;scale=2;}

 const e=makeSheep(mat,scale);
 e.hp=hp; e.max=hp; e.speed=speed; e.pathIndex=0;
 e.position.copy(path[0]);
 if(type==="boss") addCrown(e);

 e.bar=new THREE.Mesh(
  new THREE.PlaneGeometry(2.5,0.3),
  new THREE.MeshBasicMaterial({color:0x00ff00})
 );
 e.bar.position.y=3.5;
 e.add(e.bar);

 enemies.push(e); scene.add(e);
}

animate();
</script>
</body>
</html>
