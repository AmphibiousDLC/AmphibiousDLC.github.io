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
#menu{
  position:absolute;
  right:10px;
  top:10px;
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
  width:220px;
}
.btn.selected{
  border-color:orange;
  background:#fff3dd;
}
#ui{
  position:absolute;
  right:10px;
  bottom:10px;
  background:#fff;
  padding:8px;
  border-radius:8px;
  z-index:10;
}
</style>
</head>
<body>

<div id="menu">
 <div class="btn selected" data-t="Woole">Woole 🧶 10</div>
 <div class="btn" data-t="Ram">Ram 🧶 25</div>
 <div class="btn" data-t="Grazer">Grazer 🧶 100</div>
 <div class="btn" data-t="WoolPen">Wool Pen 🧶 80</div>
 <div class="btn" data-t="Goat">Goat 🧶 500</div>
 <div class="btn" data-t="Pink">Pink Sheep 🧶 1000</div>
</div>

<div id="ui">
 🧶 Wool: <span id="wool">50</span><br>
 🔁 Round: <span id="round">0</span><br>
 <button id="start">Start Round</button>
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
 grass:new THREE.MeshBasicMaterial({color:0x2f7a3a}),
 path:new THREE.MeshBasicMaterial({color:0x8b6b3e}),
 wool:new THREE.MeshBasicMaterial({color:0xdddddd}),
 goat:new THREE.MeshBasicMaterial({color:0xc8b07a}),
 dark:new THREE.MeshBasicMaterial({color:0x444444}),
 enemy:new THREE.MeshBasicMaterial({color:0xcc3333}),
 fast:new THREE.MeshBasicMaterial({color:0x3366ff}),
 tank:new THREE.MeshBasicMaterial({color:0x992222}),
 pink:new THREE.MeshBasicMaterial({color:0xff66cc}),
 orb:new THREE.MeshBasicMaterial({color:0x33ff33}),
 pen:new THREE.MeshBasicMaterial({color:0x8b5a2b}),
 horn:new THREE.MeshBasicMaterial({color:0x8b5a2b}),      // ★ ADDED
 crown:new THREE.MeshBasicMaterial({color:0xffb300})   // ★ ADDED
};

/* ================= MAP ================= */
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

const pathTiles=[];
for(let i=0;i<path.length-1;i++){
 const a=path[i],b=path[i+1];
 const len=a.distanceTo(b);
 for(let j=0;j<len/4;j++){
  const t=new THREE.Mesh(new THREE.BoxGeometry(4,0.2,4),M.path);
  t.position.copy(a.clone().lerp(b,j/(len/4)));
  t.position.y=0.1;
  scene.add(t);
  pathTiles.push(t);
 }
}

/* ================= BASE SHEEP ================= */
function baseSheep(mat){
 const g=new THREE.Group();

 const body=new THREE.Mesh(
  new THREE.SphereGeometry(1.3,12,12),mat
 );
 body.position.y=1.4;
 g.add(body);

 const face=new THREE.Mesh(
  new THREE.BoxGeometry(.6,.6,.6),M.dark
 );
 face.position.set(0,1.4,1.2);
 g.add(face);

 for(let x of[-.5,.5])for(let z of[-.5,.5]){
  const leg=new THREE.Mesh(
    new THREE.CylinderGeometry(.12,.12,.8),M.dark
  );
  leg.position.set(x,.4,z);
  g.add(leg);
 }

 g.body=body;
 return g;
}

/* ================= HORNS / CROWN ================= */
// ★ ADDED — does NOT affect anything else
function addRamHorns(g){
 [-1,1].forEach(s=>{
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(.55,.18,10,20),M.horn
  );
  h.rotation.y=Math.PI/2;
  h.position.set(.9*s,2,0);
  g.add(h);
 });
}

function addGoatHorns(g){
 [-1,1].forEach(s=>{
  const h=new THREE.Mesh(
   new THREE.TorusGeometry(1,.28,12,24),M.horn
  );
  h.rotation.y=Math.PI/2;
  h.position.set(1.2*s,2.4,0);
  g.add(h);
 });
}

function addBossCrown(g){
 const ring=new THREE.Mesh(
  new THREE.TorusGeometry(1.4,.2,10,20),M.crown
 );
 ring.rotation.x=Math.PI/2;
 ring.position.y=2.8;
 g.add(ring);

 for(let i=0;i<3;i++){
  const spike=new THREE.Mesh(
   new THREE.ConeGeometry(.25,.6,6),M.crown
  );
  spike.position.y=3.2;
  spike.rotation.y=i*(Math.PI*2/3);
  g.add(spike);
 }
}

/* ================= TOWERS ================= */
const limits={Woole:10,Ram:4,Grazer:3,WoolPen:5,Goat:2,Pink:1};
const placed={Woole:0,Ram:0,Grazer:0,WoolPen:0,Goat:0,Pink:0};

function Woole(){
 const g=baseSheep(M.wool);
 g.damage=6;g.range=12;
 return g;
}

function Ram(){
 const g=baseSheep(M.wool);
 addRamHorns(g);               // ★ ADDED
 g.damage=12;g.range=14;
 return g;
}

function Goat(){
 const g=baseSheep(M.goat);
 addGoatHorns(g);              // ★ ADDED
 g.damage=150;g.range=16;
 g.scale.set(1.2,1.2,1.2);
 return g;
}

function Grazer(){
 const g=new THREE.Group();
 const block=new THREE.Mesh(
  new THREE.BoxGeometry(2.6,1.2,2.6),
  new THREE.MeshBasicMaterial({color:0x66cc66})
 );
 block.position.y=.6;g.add(block);

 const s=baseSheep(M.wool);
 s.scale.set(.5,.5,.5);
 s.position.y=1.6;
 g.add(s);

 g.damage=12;g.range=36;g.shoots=true;
 return g;
}

function WoolPen(){
 const g=new THREE.Group();
 const wall=new THREE.Mesh(
  new THREE.CylinderGeometry(1.6,1.6,.8,12),M.pen
 );
 wall.position.y=.4;g.add(wall);

 const s=baseSheep(M.wool);
 s.scale.set(.4,.4,.4);
 s.position.y=1;
 g.add(s);

 g.support=true;g.timer=0;
 return g;
}

function Pink(){
 const g=baseSheep(M.pink);
 g.isPink=true;
 g.timer=0;
 return g;
}

const TOWERS={
 Woole:{cost:10,build:Woole},
 Ram:{cost:25,build:Ram},
 Grazer:{cost:100,build:Grazer},
 WoolPen:{cost:80,build:WoolPen},
 Goat:{cost:500,build:Goat},
 Pink:{cost:1000,build:Pink}
};

/* ================= UI ================= */
let selected="Woole";
document.querySelectorAll(".btn").forEach(b=>{
 b.onclick=()=>{
  document.querySelectorAll(".btn").forEach(x=>x.classList.remove("selected"));
  b.classList.add("selected");
  selected=b.dataset.t;
 };
});

/* ================= PLACEMENT ================= */
let wool=50;
const towers=[],enemies=[],projectiles=[];
const ray=new THREE.Raycaster(),mouse=new THREE.Vector2();
let taps=0,timer;

renderer.domElement.addEventListener("pointerdown",e=>{
 taps++;
 if(taps===1){timer=setTimeout(()=>taps=0,300);}
 else{
  clearTimeout(timer);taps=0;
  if(placed[selected]>=limits[selected])return;
  if(wool<TOWERS[selected].cost)return;

  const r=renderer.domElement.getBoundingClientRect();
  mouse.x=((e.clientX-r.left)/r.width)*2-1;
  mouse.y=-((e.clientY-r.top)/r.height)*2+1;
  ray.setFromCamera(mouse,camera);
  const hit=ray.intersectObject(ground);
  if(!hit.length)return;

  for(const p of pathTiles)
   if(p.position.distanceTo(hit[0].point)<3)return;

  wool-=TOWERS[selected].cost;
  document.getElementById("wool").textContent=wool;

  const t=TOWERS[selected].build();
  t.position.set(Math.round(hit[0].point.x),0,Math.round(hit[0].point.z));
  t.cooldown=0;
  placed[selected]++;
  towers.push(t);
  scene.add(t);
 }
});

/* ================= ENEMIES ================= */
function makeEnemy(type){
 let mat=M.enemy,hp=80,speed=.045;
 if(type==="fast"){mat=M.fast;hp=60;speed=.08}
 if(type==="tank"){mat=M.tank;hp=240;speed=.025}

 const g=baseSheep(mat);
 g.hp=hp;g.max=hp;g.speed=speed;g.i=0;

 if(type==="tank") addBossCrown(g);   // ★ ADDED

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
let round=0,queue=[],spawning=false,spawnTimer=0;
document.getElementById("start").onclick=()=>{
 if(spawning||enemies.length)return;
 round++;document.getElementById("round").textContent=round;
 queue=[];
 queue.push(...Array(6+round).fill("normal"));
 if(round>=5)queue.splice(Math.random()*queue.length|0,0,"fast");
 if(round>=10)queue.splice(Math.random()*queue.length|0,0,"tank");
 spawning=true;
};

/* ================= LOOP ================= */
function animate(){
 requestAnimationFrame(animate);

 if(spawning&&queue.length){
  spawnTimer++;
  if(spawnTimer>40){
   spawnTimer=0;
   const e=makeEnemy(queue.shift());
   e.position.copy(path[0]);
   enemies.push(e);
   scene.add(e);
  }
 }

 enemies.forEach(e=>{
  const n=path[e.i+1];
  if(!n){alert("You lost!");location.reload();}
  e.lookAt(n.x,1.4,n.z);
  const d=n.clone().sub(e.position);
  if(d.length()<.5)e.i++;
  e.position.add(d.normalize().multiplyScalar(e.speed));
  e.bar.scale.x=e.hp/e.max;
 });

 towers.forEach(t=>{
  if(t.support){
   t.timer++;
   if(t.timer>900){
    t.timer=0;
    wool+=20;
    document.getElementById("wool").textContent=wool;
   }
   return;
  }
  if(t.isPink){
   t.timer++;
   if(t.timer>1800){
    t.timer=0;
    const b=makeEnemy("tank");
    b.hp=500;b.max=500;
    b.position.copy(path[path.length-1]);
    b.i=path.length-2;
    enemies.push(b);scene.add(b);
   }
   return;
  }
  if(t.cooldown>0)t.cooldown--;
  const target=enemies.find(e=>e.position.distanceTo(t.position)<t.range);
  if(target&&t.cooldown===0){
   t.lookAt(target.position);
   if(t.shoots){
    const o=new THREE.Mesh(
      new THREE.SphereGeometry(.25,8,8),M.orb
    );
    o.position.copy(t.position);o.position.y=1.5;
    o.target=target;o.damage=t.damage;
    projectiles.push(o);scene.add(o);
   }else target.hp-=t.damage;
   t.cooldown=40;
  }
 });

 projectiles.forEach((p,i)=>{
  if(!p.target||p.target.hp<=0){
    scene.remove(p);projectiles.splice(i,1);return;
  }
  const d=p.target.position.clone().sub(p.position);
  if(d.length()<.5){
    p.target.hp-=p.damage;
    scene.remove(p);projectiles.splice(i,1);
  }else p.position.add(d.normalize().multiplyScalar(.6));
 });

 for(let i=enemies.length-1;i>=0;i--){
  if(enemies[i].hp<=0){
   wool+=5;
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
