<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Wooly Warfare</title>

<style>
html,body{margin:0;overflow:hidden;font-family:sans-serif}
#ui{
 position:absolute;right:10px;top:10px;
 background:#fff;padding:8px;border-radius:8px;z-index:10
}
#menu{
 position:absolute;right:10px;bottom:10px;
 display:flex;flex-direction:column;gap:6px;z-index:10
}
.btn{
 background:#fff;border:2px solid #777;
 border-radius:8px;padding:6px;width:200px
}
.btn.selected{border-color:orange;background:#fff3dd}
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
 path:new THREE.MeshBasicMaterial({color:0x8b6b3e}),
 wool:new THREE.MeshBasicMaterial({color:0xdddddd}),
 goat:new THREE.MeshBasicMaterial({color:0xc8b07a}),
 enemy:new THREE.MeshBasicMaterial({color:0xcc3333}),
 fast:new THREE.MeshBasicMaterial({color:0x3366ff}),
 tank:new THREE.MeshBasicMaterial({color:0x992222}),
 dark:new THREE.MeshBasicMaterial({color:0x444444}),
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
  const p=new THREE.Mesh(new THREE.BoxGeometry(4,0.2,4),M.path);
  p.position.copy(a.clone().lerp(b,j/(len/4)));
  p.position.y=0.1;
  scene.add(p);
 }
}

/* ================= SHEEP MODEL ================= */
function sheep(mat){
 const g=new THREE.Group();
 const body=new THREE.Mesh(new THREE.SphereGeometry(1.3,10,10),mat);
 body.position.y=1.4;g.add(body);g.body=body;

 const face=new THREE.Mesh(new THREE.BoxGeometry(.6,.6,.6),M.dark);
 face.position.set(0,1.4,1.2);g.add(face);

 [-.5,.5].forEach(x=>[-.5,.5].forEach(z=>{
  const leg=new THREE.Mesh(new THREE.CylinderGeometry(.12,.12,.8),M.dark);
  leg.position.set(x,.4,z);g.add(leg);
 }));
 return g;
}

/* ================= TOWERS ================= */
function cube(){const g=sheep(M.wool);g.damage=5;g.range=12;return g}
function ram(){const g=sheep(M.wool);g.damage=10;g.range=14;return g}
function goat(){const g=sheep(M.goat);g.damage=150;g.range=16;g.scale.set(1.2,1.2,1.2);return g}

const TOWERS={
 Cube:{cost:10,build:cube},
 Ram:{cost:25,build:ram},
 Goat:{cost:500,build:goat}
};

/* ================= MENU ================= */
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
const ray=new THREE.Raycaster(),pt=new THREE.Vector2();
let taps=0,timer;
const towers=[];

renderer.domElement.addEventListener("pointerdown",e=>{
 taps++;
 if(taps===1){
  timer=setTimeout(()=>taps=0,300);
 }else{
  clearTimeout(timer);taps=0;

  const r=renderer.domElement.getBoundingClientRect();
  pt.x=((e.clientX-r.left)/r.width)*2-1;
  pt.y=-((e.clientY-r.top)/r.height)*2+1;

  ray.setFromCamera(pt,camera);
  const hit=ray.intersectObject(ground);
  if(hit.length){
   const t=TOWERS[selected];
   if(wool<t.cost)return;
   wool-=t.cost;
   document.getElementById("wool").textContent=wool;
   const u=t.build();
   u.position.copy(hit[0].point).setY(0);
   u.cooldown=0;
   towers.push(u);
   scene.add(u);
  }
 }
});

/* ================= ENEMIES ================= */
function enemy(type){
 let mat=M.enemy,hp=40,speed=.045,woolGain=5;
 if(type==="fast"){mat=M.fast;hp=30;speed=.08}
 if(type==="tank"){mat=M.tank;hp=120;speed=.025}
 if(type==="boss"){hp=1000;speed=.02}

 const g=sheep(mat);
 g.hp=hp;g.max=hp;g.speed=speed;g.i=0;g.wool=woolGain;

 if(type==="boss"){
  const c=new THREE.Mesh(new THREE.TorusGeometry(1,.3,8,16),M.crown);
  c.position.y=2.8;g.add(c);g.scale.set(2,2,2);
 }

 const bar=new THREE.Mesh(
  new THREE.PlaneGeometry(2.5,.3),
  new THREE.MeshBasicMaterial({color:0x00ff00})
 );
 bar.position.y=3;g.add(bar);g.bar=bar;

 return g;
}

const enemies=[];

/* ================= ROUND LOGIC ================= */
let round=0,queue=[],spawnTimer=0,spawning=false;

document.getElementById("startRound").onclick=()=>{
 if(spawning||enemies.length)return;
 round++;document.getElementById("round").textContent=round;

 queue=[];
 const normals=6+round;
 const fasts=Math.max(0,round-4);
 const tanks=Math.max(0,round-9);
 const boss=(round>=20&&round%10===0);

 queue.push(...Array(normals).fill("normal"));

 function insert(type,count){
  for(let i=0;i<count;i++){
   const idx=Math.floor(Math.random()*queue.length);
   queue.splice(idx,0,type);
  }
 }

 insert("fast",fasts);
 insert("tank",tanks);
 if(boss) queue.push("boss");

 spawning=true;
};

/* ================= LOOP ================= */
function animate(){
 requestAnimationFrame(animate);

 if(spawning&&queue.length){
  spawnTimer++;
  if(spawnTimer>50){
   spawnTimer=0;
   const e=enemy(queue.shift());
   e.position.copy(path[0]);
   enemies.push(e);scene.add(e);
  }
 }
 if(spawning&&!queue.length&&!enemies.length)spawning=false;

 enemies.forEach(e=>{
  const n=path[e.i+1]; if(!n)return;
  e.lookAt(n.x,1.4,n.z);
  const d=n.clone().sub(e.position);
  if(d.length()<.5)e.i++;
  e.position.add(d.normalize().multiplyScalar(e.speed));
  e.bar.scale.x=e.hp/e.max;
 });

 towers.forEach(t=>{
  if(t.cooldown>0)t.cooldown--;
  const target=enemies.find(e=>e.hp>0&&e.position.distanceTo(t.position)<t.range);
  if(target&&t.cooldown===0){
   target.hp-=t.damage;
   t.cooldown=40;
  }
 });

 for(let i=enemies.length-1;i>=0;i--){
  if(enemies[i].hp<=0){
   wool+=enemies[i].wool;
   document.getElementById("wool").textContent=wool;
   scene.remove(enemies[i]);enemies.splice(i,1);
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
