<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Cheese Triumph</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{margin:0;overflow:hidden;font-family:sans-serif;background:#ddd}

/* START SCREEN */
#startScreen{
  position:fixed; inset:0;
  background:#ddd;
  display:flex;
  flex-direction:column;
  align-items:center;
  justify-content:center;
  z-index:10;
}
#title{
  font-size:64px;
  font-weight:900;
  margin-bottom:40px;
  transform:perspective(600px) rotateX(15deg);
}
#title span{color:#ffd700}
#title .c{color:#7fbf3f}
#startBtn{
  padding:20px 50px;
  font-size:28px;
  font-weight:bold;
  color:white;
  background:#1e6bff;
  border:none;
  border-radius:14px;
  box-shadow:0 10px 0 #0d3da8;
}
#startBtn:active{
  box-shadow:0 4px 0 #0d3da8;
  transform:translateY(6px);
}

/* JOYSTICK */
#joystick{
  position:fixed;
  bottom:90px; left:30px;
  width:120px;height:120px;
  border-radius:50%;
  background:rgba(0,0,0,.25);
  display:none;
}
#stick{
  position:absolute;
  left:40px;top:40px;
  width:40px;height:40px;
  border-radius:50%;
  background:rgba(0,0,0,.6);
}

/* ENDINGS */
#endings{
  position:fixed;
  bottom:10px; left:50%;
  transform:translateX(-50%);
  display:none;
  gap:18px;
}
.ending{
  width:56px;height:56px;
  border-radius:50%;
  border:3px solid #444;
  background:#bbb;
  display:flex;
  align-items:center;
  justify-content:center;
  opacity:.35;
}
.ending.glow{
  opacity:1;
  box-shadow:0 0 14px 6px rgba(255,255,255,.8);
}
.cheese{
  width:26px;height:26px;
  clip-path:polygon(0 0,100% 50%,0 100%);
}
.good{background:#ffd700}
.mouldy{background:#7fbf3f}
.wander{background:#d0d0d0}
</style>
</head>
<body>

<div id="startScreen">
  <div id="title">
    <span class="c">C</span><span>HEESE TRIUMPH</span>
  </div>
  <button id="startBtn">START</button>
</div>

<div id="joystick"><div id="stick"></div></div>

<div id="endings">
  <div id="end-good" class="ending"><div class="cheese good"></div></div>
  <div id="end-mouldy" class="ending"><div class="cheese mouldy"></div></div>
  <div id="end-walk" class="ending"><div class="cheese wander"></div></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.155.0/build/three.min.js"></script>
<script>
/* DOM */
const startScreen=document.getElementById("startScreen");
const startBtn=document.getElementById("startBtn");
const joystick=document.getElementById("joystick");
const endings=document.getElementById("endings");
const endGood=document.getElementById("end-good");
const endMouldy=document.getElementById("end-mouldy");
const endWalk=document.getElementById("end-walk");

/* FLAGS */
let started=false;
let holding=null;
let touchedCheese=false;
let walkTimer=0;
let gotGood=false;
let gotMouldy=false;
let gotWalk=false;

/* START */
startBtn.onclick=()=>{
  startScreen.style.display="none";
  joystick.style.display="block";
  endings.style.display="flex";
  started=true;
};

/* SCENE */
const scene=new THREE.Scene();
scene.background=new THREE.Color(0xffffff);

const camera=new THREE.PerspectiveCamera(70,innerWidth/innerHeight,.1,500);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.outputColorSpace=THREE.SRGBColorSpace;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff,1));
const sun=new THREE.DirectionalLight(0xffffff,1.2);
sun.position.set(10,20,10);
scene.add(sun);

/* GROUND */
const ground=new THREE.Mesh(
  new THREE.PlaneGeometry(500,500),
  new THREE.MeshStandardMaterial({color:0xdddddd})
);
ground.rotation.x=-Math.PI/2;
scene.add(ground);

/* PLAYER */
const knight=new THREE.Group();
const body=new THREE.Mesh(
  new THREE.BoxGeometry(1,1,1),
  new THREE.MeshStandardMaterial({color:0xaaaaaa})
);
body.position.y=.5;
knight.add(body);

const band=new THREE.Mesh(
  new THREE.BoxGeometry(1.01,.25,1.01),
  new THREE.MeshStandardMaterial({color:0x444444})
);
band.position.y=.125;
knight.add(band);

/* HELMET GRATE */
for(let i=-1;i<=1;i++){
  const slit=new THREE.Mesh(
    new THREE.BoxGeometry(.05,.35,.01),
    new THREE.MeshStandardMaterial({color:0x444444})
  );
  slit.position.set(i*.15,.65,.505);
  knight.add(slit);
}
scene.add(knight);

/* WIZARD MATERIAL */
function wizardMaterial(base,spot){
  const c=document.createElement("canvas");
  c.width=c.height=512;
  const x=c.getContext("2d");
  x.fillStyle=base;
  x.fillRect(0,0,512,512);
  x.fillStyle=spot;
  for(let i=0;i<14;i++){
    x.beginPath();
    x.arc((i%7)*70+50,Math.floor(i/7)*140+80,28,0,Math.PI*2);
    x.fill();
  }
  return new THREE.MeshStandardMaterial({map:new THREE.CanvasTexture(c)});
}

/* WIZARD */
const wizard=new THREE.Mesh(
  new THREE.ConeGeometry(1,2,32),
  wizardMaterial("#4b0082","#ffd700")
);
wizard.position.set(6,1,0);
scene.add(wizard);

/* GOOD CHEESE */
const goodCheese=new THREE.Mesh(
  new THREE.ConeGeometry(.6,1,3),
  new THREE.MeshStandardMaterial({
    color:0xffd700,
    emissive:0xffd700,
    emissiveIntensity:.5
  })
);
goodCheese.rotation.x=Math.PI/2;
scene.add(goodCheese);

/* MOULDY CHEESE — NORMAL SIZE, FAR AWAY */
const mouldyCheese=new THREE.Mesh(
  new THREE.ConeGeometry(.6,1,3),
  new THREE.MeshStandardMaterial({
    color:0x7fbf3f,
    emissive:0x3b5f1f,
    emissiveIntensity:.4
  })
);
mouldyCheese.rotation.x=Math.PI/2;
scene.add(mouldyCheese);

/* CROWN */
let crown=null;
function giveCrown(){
  if(crown)return;
  crown=new THREE.Group();
  const ring=new THREE.Mesh(
    new THREE.TorusGeometry(.6,.15,12,24),
    new THREE.MeshStandardMaterial({color:0xffc400})
  );
  ring.rotation.x=Math.PI/2;
  crown.add(ring);
  for(let i=0;i<3;i++){
    const spike=new THREE.Mesh(
      new THREE.ConeGeometry(.15,.4,6),
      new THREE.MeshStandardMaterial({color:0xffc400})
    );
    spike.position.set(
      Math.cos(i*2*Math.PI/3)*.4,
      .35,
      Math.sin(i*2*Math.PI/3)*.4
    );
    crown.add(spike);
  }
  crown.position.y=1.6;
  knight.add(crown);
}

/* CONTROLS */
let forward=0,turn=0;
const SPEED=.06,TURN=.04;
joystick.ontouchend=()=>{forward=turn=0};
joystick.ontouchmove=e=>{
  const r=joystick.getBoundingClientRect();
  const t=e.touches[0];
  forward=-(t.clientY-r.top-60)/40;
  turn=(t.clientX-r.left-60)/40;
};

/* RESET */
function resetGame(){
  holding=null;
  touchedCheese=false;
  walkTimer=0;

  knight.position.set(0,0,0);
  knight.rotation.y=0;

  wizard.position.set(6,1,0);

  goodCheese.position.set(-6,1,0);
  mouldyCheese.position.set(80,1,80); // FAR → looks like a pixel

  if(crown){knight.remove(crown);crown=null;}
}

/* LOOP */
resetGame();
function animate(){
  requestAnimationFrame(animate);
  if(!started)return;

  goodCheese.rotation.y+=.04;
  mouldyCheese.rotation.y+=.02;

  knight.rotation.y-=turn*TURN;
  knight.position.x+=Math.sin(knight.rotation.y)*forward*SPEED;
  knight.position.z+=Math.cos(knight.rotation.y)*forward*SPEED;

  camera.position.set(
    knight.position.x-Math.sin(knight.rotation.y)*8,
    6,
    knight.position.z-Math.cos(knight.rotation.y)*8
  );
  camera.lookAt(knight.position.x,1,knight.position.z);

  /* WANDERER */
  if(!touchedCheese){
    walkTimer++;
    if(walkTimer>3600&&!gotWalk){
      gotWalk=true;
      endWalk.classList.add("glow");
    }
  }

  /* PICKUP */
  if(!holding){
    if(knight.position.distanceTo(goodCheese.position)<1.2){
      holding=goodCheese;
      touchedCheese=true;
    } else if(knight.position.distanceTo(mouldyCheese.position)<1.2){
      holding=mouldyCheese;
      touchedCheese=true;
    }
  }

  if(holding){
    holding.position.set(knight.position.x,1.6,knight.position.z);
  }

  /* GIVE */
  if(holding && knight.position.distanceTo(wizard.position)<1.8){
    if(holding===goodCheese){
      giveCrown();
      endGood.classList.add("glow");
      gotGood=true;
    }
    if(holding===mouldyCheese){
      endMouldy.classList.add("glow");
      gotMouldy=true;
    }
    holding.position.set(-999,-999,-999);
    holding=null;
    setTimeout(resetGame,5000);
  }

  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
