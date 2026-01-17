<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no">
<title>Wooly Warfare</title>
<style>
html,body{margin:0;overflow:hidden;touch-action:none;font-family:sans-serif}
#ui{
 position:absolute;right:10px;top:50%;
 transform:translateY(-50%);
 background:#fff;padding:10px;border-radius:10px;z-index:10;
}
.btn{border:2px solid #777;border-radius:8px;padding:6px;margin:4px 0}
.btn.sel{border-color:orange;background:#ffe8c4}
</style>
</head>
<body>

<div id="ui">
🧶 Wool: <span id="wool">50</span><br>
🔁 Round: <span id="round">0</span><br>
<button id="start">Start Round</button>
<div id="menu"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160/build/three.min.js"></script>
<script>
/* ================= SETUP ================= */
const scene=new THREE.Scene();
scene.background=new THREE.Color(0x87ceeb);

const cam=new THREE.PerspectiveCamera(60,innerWidth/innerHeight,0.1,300);
cam.position.set(50,55,65);
cam.lookAt(0,0,0);

const r=new THREE.WebGLRenderer({antialias:true});
r.setSize(innerWidth,innerHeight);
document.body.appendChild(r.domElement);

scene.add(new THREE.AmbientLight(0xffffff,1));

/* ================= MATERIALS ================= */
const M={
 grass:new THREE.MeshBasicMaterial({color:0x2a6a32}),
 grassLight:new THREE.MeshBasicMaterial({color:0x4caf50}),
 path:new THREE.MeshBasicMaterial({color:0x8b6b3e}),
 wool:new THREE.MeshBasicMaterial({color:0xdddddd}),
 goat:new THREE.MeshBasicMaterial({color:0xc8b07a}),
 enemy:new THREE.MeshBasicMaterial({color:0xcc3333}),
 dark:new THREE.MeshBasicMaterial({color:0x444444}),
 horn:new THREE.MeshBasicMaterial({color:0x8b5a2b}),
 crown:new THREE.MeshBasicMaterial({color:0xffcc00}),
 pink:new THREE.MeshBasicMaterial({color:0xff66cc}),
 projectile:new THREE.MeshBasicMaterial({color:0x33ff33})
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

// visible path tiles
for(let i=0;i<path.length-1;i++){
 const a=path[i],b=path[i+1];
 const len=a.distanceTo(b);
 for(let j=0;j<len/4;j++){
  const tile=new THREE.Mesh(
   new THREE.BoxGeometry(4,0.2,4),M.path
  );
  tile.position.copy(a.clone().lerp(b,j/(len/4)));
  tile.position.y=0.1;
  scene.add(tile);
 }
}

/* ================= SHEEP BASE ================= */
function sheepBase(mat){
 const root=new THREE.Group();
 const pivot=new THREE.Group();
 root.add(pivot);
 root.pivot=pivot;

 const body=new THREE.Mesh(new THREE.SphereGeometry(1.3,10,10),mat);
 body.position.y=1.4;
 pivot.add(body);

 const face=new THREE.Mesh(new THREE.BoxGeometry(.6,.6,.6),M.dark);
 face.position.set(0,1.4,1.2);
 pivot.add(face);

 for(let x of[-.5,.5])for(let z of[-.5,.5]){
  const leg=new THREE.Mesh(
   new THREE.CylinderGeometry(.12,.12,.8),M.dark
  );
  leg.position.set(x,.4,z);
  pivot.add(leg);
 }

 root.body=body;
 root.attackTime=0;
 return root;
}

/* ================= HORNS ================= */
function ramHorns(g){
 for(let s of[-1,1]){
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(.6,.18,10,20),M.horn
  );
  h.rotation.y=Math.PI/2;
  h.position.set(.9*s,1.6,0);
  g.pivot.add(h);
 }
}
function goatHorns(g){
 for(let s of[-1,1]){
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(1.2,.3,12,24),M.horn
  );
  h.rotation.y=Math.PI/2;
  h.position.set(1.3*s,1.9,0);
  g.pivot.add(h);
 }
}

/* ================= TOWERS ================= */
function Cube(){const g=sheepBase(M.wool);g.damage=5;g.range=12;return g;}
function Ram(){const g=sheepBase(M.wool);ramHorns(g);g.damage=10;g.range=14;return g;}
function Goat(){const g=sheepBase(M.goat);goatHorns(g);g.damage=150;g.range=16;g.scale.set(1.2,1.2,1.2);return g;}

function Grazer(){
 const g=new THREE.Group();

 const cube=new THREE.Mesh(
  new THREE.BoxGeometry(2.6,1.2,2.6),M.grassLight
 );
 cube.position.y=.6;
 g.add(cube);

 const sheep=sheepBase(M.wool);
 sheep.scale.set(.5,.5,.5);
 sheep.position.y=1.5;
 g.add(sheep);

 g.damage=16;
 g.range=14;
 g.shoots=true;
 g.cool=0;
 return g;
}

function Shearer(){
 const g=new THREE.Group();
 const pillar=new THREE.Mesh(
  new THREE.CylinderGeometry(.4,.4,3),new THREE.MeshBasicMaterial({color:0x999999})
 );
 pillar.position.y=1.5;
 g.add(pillar);
 g.support=true;
 return g;
}

function Pink(){
 const g=sheepBase(M.pink);
 g.special=true;
 g.damage=300;
 return g;
}

const TOWERS={
 Cube:{cost:10,build:Cube},
 Ram:{cost:25,build:Ram},
 Grazer:{cost:100,build:Grazer},
 Shearer:{cost:250,build:Shearer},
 Goat:{cost:500,build:Goat},
 Pink:{cost:1000,build:Pink}
};

/* ================= UI ================= */
let wool=50,round=0,selected="Cube";
const menu=document.getElementById("menu");
for(const k in TOWERS){
 const b=document.createElement("div");
 b.className="btn"+(k===selected?" sel":"");
 b.textContent=`${k} (${TOWERS[k].cost})`;
 b.onclick=()=>{
  selected=k;
  document.querySelectorAll(".btn").forEach(x=>x.classList.remove("sel"));
  b.classList.add("sel");
 };
 menu.appendChild(b);
}

/* ================= PLACEMENT ================= */
const ray=new THREE.Raycaster();
const pt=new THREE.Vector2();
let taps=0,timer;
const towers=[];

r.domElement.addEventListener("pointerdown",e=>{
 taps++;
 if(taps===1) timer=setTimeout(()=>taps=0,300);
 else{
  clearTimeout(timer);taps=0;

  const p=e.touches?e.touches[0]:e;
  const rc=r.domElement.getBoundingClientRect();
  pt.x=((p.clientX-rc.left)/rc.width)*2-1;
  pt.y=-((p.clientY-rc.top)/rc.height)*2+1;
  ray.setFromCamera(pt,cam);

  const hit=ray.intersectObject(ground);
  if(!hit.length) return;

  const t=TOWERS[selected];
  if(wool<t.cost) return;

  wool-=t.cost;
  document.getElementById("wool").textContent=wool;

  const u=t.build();
  u.position.set(
   Math.round(hit[0].point.x),
   0,
   Math.round(hit[0].point.z)
  );
  u.cool=0;
  towers.push(u);
  scene.add(u);
 }
});

/* ================= ENEMIES ================= */
const enemies=[];
function makeEnemy(){
 const g=sheepBase(M.enemy);
 g.hp=80; g.max=80; g.speed=.045; g.i=0;

 const bar=new THREE.Mesh(
  new THREE.PlaneGeometry(2.5,.3),
  new THREE.MeshBasicMaterial({color:0x00ff00})
 );
 bar.position.y=3;
 g.add(bar);
 g.bar=bar;

 return g;
}

/* ================= ROUNDS ================= */
document.getElementById("start").onclick=()=>{
 if(enemies.length) return;
 round++;
 document.getElementById("round").textContent=round;
 for(let i=0;i<5+round;i++){
  setTimeout(()=>{
   const e=makeEnemy();
   e.position.copy(path[0]);
   enemies.push(e);
   scene.add(e);
  },i*600);
 }
};

/* ================= LOOP ================= */
function animate(){
 requestAnimationFrame(animate);

 enemies.forEach(e=>{
  const n=path[e.i+1];
  if(!n){
   alert("YOU LOSE");
   location.reload();
  }
  const d=n.clone().sub(e.position);
  if(d.length()<.5) e.i++;
  d.normalize();
  e.rotation.y=Math.atan2(d.x,d.z);
  e.position.add(d.multiplyScalar(e.speed));
  e.bar.scale.x=Math.max(e.hp/e.max,0);
 });

 towers.forEach(t=>{
  if(t.support) return;

  if(t.cool>0)t.cool--;

  let target=null,dist=Infinity;
  enemies.forEach(e=>{
   if(e.hp<=0)return;
   const d=t.position.distanceTo(e.position);
   if(d<t.range&&d<dist){target=e;dist=d;}
  });

  if(target&&t.cool===0){
   const dx=target.position.x-t.position.x;
   const dz=target.position.z-t.position.z;
   t.rotation.y=Math.atan2(dx,dz);

   target.hp-=t.damage;
   t.attackTime=8;
   t.cool=30;
  }

  if(t.attackTime>0){
   t.attackTime--;
   if(t.pivot) t.pivot.position.y=Math.sin(t.attackTime*.6)*.15;
  }else if(t.pivot){
   t.pivot.position.y*=.8;
  }
 });

 for(let i=enemies.length-1;i>=0;i--){
  if(enemies[i].hp<=0){
   wool+=5;
   document.getElementById("wool").textContent=wool;
   scene.remove(enemies[i]);
   enemies.splice(i,1);
  }
 }

 r.render(scene,cam);
}
animate();

window.addEventListener("resize",()=>{
 cam.aspect=innerWidth/innerHeight;
 cam.updateProjectionMatrix();
 r.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
