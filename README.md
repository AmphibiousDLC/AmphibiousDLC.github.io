<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Wooly Warfare</title>

<style>
html,body{margin:0;overflow:hidden;touch-action:none;font-family:sans-serif}
#ui{position:absolute;top:10px;left:10px;background:#fff;padding:8px;border-radius:8px;z-index:5}
#menu{position:absolute;right:10px;top:50%;transform:translateY(-50%);display:flex;flex-direction:column;gap:6px}
.btn{background:#fff;border:2px solid #777;border-radius:8px;padding:6px;width:180px}
.btn.selected{border-color:orange;background:#fff3dd}
</style>
</head>

<body>
<div id="ui">🧶 Wool: <span id="wool">500</span></div>
<div id="menu"></div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
/* ================= SETUP ================= */
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
 dark:new THREE.MeshBasicMaterial({color:0x444444}),
 horn:new THREE.MeshBasicMaterial({color:0x8b5a2b}),
 metal:new THREE.MeshBasicMaterial({color:0xb0b0b0}),
 wood:new THREE.MeshBasicMaterial({color:0x8b5a2b}),
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
 const body=new THREE.Mesh(new THREE.SphereGeometry(1.3,8,8),mat);
 body.position.y=1.4; g.add(body); g.body=body;

 const face=new THREE.Mesh(new THREE.BoxGeometry(.6,.6,.6),M.dark);
 face.position.set(0,1.4,1.2); g.add(face);

 [-.5,.5].forEach(x=>{
  [-.5,.5].forEach(z=>{
   const leg=new THREE.Mesh(new THREE.CylinderGeometry(.12,.12,.8),M.dark);
   leg.position.set(x,.4,z); g.add(leg);
  });
 });
 return g;
}

/* ================= TOWERS ================= */
function cube(){const g=baseSheep(M.wool);g.damage=5;g.range=12;return g;}
function ram(){
 const g=baseSheep(M.wool);g.damage=10;g.range=14;
 [-1,1].forEach(s=>{
  const h=new THREE.Mesh(new THREE.TorusGeometry(.5,.15,8,16),M.horn);
  h.rotation.y=Math.PI/2;h.position.set(.7*s,1.8,.4);g.add(h);
 });
 return g;
}
function goat(){
 const g=baseSheep(M.goat);g.scale.set(1.2,1.2,1.2);
 g.damage=150;g.range=16;
 [-1,1].forEach(s=>{
  const h=new THREE.Mesh(new THREE.TorusGeometry(1.2,.18,8,20),M.horn);
  h.rotation.y=Math.PI/2;h.position.set(1.1*s,2.2,.3);g.add(h);
 });
 return g;
}
function grazer(){
 const g=new THREE.Group();
 const cube=new THREE.Mesh(new THREE.BoxGeometry(2.6,1.2,2.6),M.grassLight);
 cube.position.y=.6;g.add(cube);
 const s=baseSheep(M.wool);s.scale.set(.5,.5,.5);s.position.y=1.5;g.add(s);
 g.damage=8;g.range=14;g.shoots=true;
 return g;
}
function shearer(){
 const g=new THREE.Group();
 const p=new THREE.Mesh(new THREE.CylinderGeometry(.6,.6,3),M.metal);
 p.position.y=1.5;g.add(p);
 g.support=true;return g;
}

/* ================= TOWER DATA ================= */
const TOWERS={
 Cube:{cost:10,build:cube},
 Ram:{cost:25,build:ram},
 Grazer:{cost:100,build:grazer},
 Shearer:{cost:250,build:shearer},
 Goat:{cost:500,build:goat}
};

let wool=500,selected="Cube";
const menu=document.getElementById("menu");
for(const k in TOWERS){
 const b=document.createElement("div");
 b.className="btn"+(k===selected?" selected":"");
 b.textContent=`${k} (${TOWERS[k].cost})`;
 b.onclick=()=>{selected=k;document.querySelectorAll(".btn").forEach(x=>x.classList.remove("selected"));b.classList.add("selected")};
 menu.appendChild(b);
}

/* ================= PLACEMENT ================= */
const ray=new THREE.Raycaster(),pt=new THREE.Vector2();
let taps=0,timer;
renderer.domElement.addEventListener("pointerdown",e=>{
 taps++; if(taps===1){timer=setTimeout(()=>taps=0,300);}
 else{
  clearTimeout(timer);taps=0;
  const p=e.touches?e.touches[0]:e;
  const r=renderer.domElement.getBoundingClientRect();
  pt.x=((p.clientX-r.left)/r.width)*2-1;
  pt.y=-((p.clientY-r.top)/r.height)*2+1;
  ray.setFromCamera(pt,camera);
  const hit=ray.intersectObject(ground);
  if(hit.length) place(hit[0].point);
 }
});
const towers=[];
function place(pos){
 const t=TOWERS[selected]; if(wool<t.cost)return;
 wool-=t.cost; document.getElementById("wool").textContent=wool;
 const u=t.build();
 u.position.set(Math.round(pos.x),0,Math.round(pos.z));
 u.cooldown=0; towers.push(u); scene.add(u);
}

/* ================= ENEMIES ================= */
function enemy(){
 const g=baseSheep(M.enemy);
 g.hp=20; g.max=20; g.i=0;
 const bar=new THREE.Mesh(new THREE.PlaneGeometry(2.5,.3),new THREE.MeshBasicMaterial({color:0x00ff00}));
 bar.position.y=3; g.add(bar); g.bar=bar;
 return g;
}
const enemies=[],projectiles=[];
let spawn=0;
function spawnEnemy(){
 const e=enemy(); e.position.copy(path[0]); enemies.push(e); scene.add(e);
}

/* ================= COMBAT ================= */
function attack(t,e){
 t.lookAt(e.position.x,1.4,e.position.z);
 t.body.rotation.x=Math.sin(Date.now()*0.02)*0.4;
 if(t.shoots){
  const p=new THREE.Mesh(new THREE.SphereGeometry(.25),M.projectile);
  p.position.copy(t.position).add(new THREE.Vector3(0,2,0));
  p.target=e; p.damage=t.damage;
  projectiles.push(p); scene.add(p);
 }else{
  e.hp-=t.damage;
 }
 t.cooldown=40;
}

/* ================= LOOP ================= */
function animate(){
 requestAnimationFrame(animate);

 spawn++; if(spawn>120){spawn=0;spawnEnemy();}

 enemies.forEach(e=>{
  const n=path[e.i+1]; if(!n)return;
  e.lookAt(n.x,1.4,n.z);
  const d=n.clone().sub(e.position);
  if(d.length()<.5)e.i++;
  e.position.add(d.normalize().multiplyScalar(.05));
  e.bar.scale.x=e.hp/e.max;
 });

 towers.forEach(t=>{
  if(t.cooldown>0)t.cooldown--;
  if(t.support)return;
  const target=enemies.find(e=>e.position.distanceTo(t.position)<t.range&&e.hp>0);
  if(target&&t.cooldown===0)attack(t,target);
 });

 projectiles.forEach((p,i)=>{
  if(!p.target||p.target.hp<=0){scene.remove(p);projectiles.splice(i,1);return;}
  const d=p.target.position.clone().add(new THREE.Vector3(0,1.5,0)).sub(p.position);
  if(d.length()<.5){
   p.target.hp-=p.damage;
   scene.remove(p);projectiles.splice(i,1);
  }else p.position.add(d.normalize().multiplyScalar(.6));
 });

 for(let i=enemies.length-1;i>=0;i--){
  if(enemies[i].hp<=0){scene.remove(enemies[i]);enemies.splice(i,1);wool+=5;document.getElementById("wool").textContent=wool;}
 }

 renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
