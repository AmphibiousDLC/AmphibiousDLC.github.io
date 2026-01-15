<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Cheese Triumph</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{margin:0;overflow:hidden;font-family:sans-serif;background:#ddd}

/* ================= START SCREEN ================= */
#startScreen{
  position:fixed;
  inset:0;
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
  letter-spacing:4px;
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

/* ================= GAME UI ================= */
#joystick{
  position:fixed;
  bottom:90px;
  left:30px;
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

#endings{
  position:fixed;
  bottom:10px;
  left:50%;
  transform:translateX(-50%);
  display:flex;
  gap:18px;
  display:none;
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

<!-- START SCREEN -->
<div id="startScreen">
  <div id="title">
    <span class="c">C</span><span>HEESE TRIUMPH</span>
  </div>
  <button id="startBtn">START</button>
</div>

<!-- GAME UI -->
<div id="joystick"><div id="stick"></div></div>

<div id="endings">
  <div id="end-good" class="ending"><div class="cheese good"></div></div>
  <div id="end-boss" class="ending"><div class="cheese mouldy"></div></div>
  <div id="end-walk" class="ending"><div class="cheese wander"></div></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.155.0/build/three.min.js"></script>
<script>
/* ================= START LOGIC ================= */
let started=false;
document.getElementById("startBtn").onclick=()=>{
  document.getElementById("startScreen").style.display="none";
  document.getElementById("joystick").style.display="block";
  document.getElementById("endings").style.display="flex";
  started=true;
};

/* ================= FLAGS ================= */
let gotGoodEnding=false;
let gotBossEnding=false;
let gotWalkEnding=false;

let walkTimer=0;
let touchedCheese=false;

/* ================= SCENE ================= */
const scene=new THREE.Scene();
scene.background=new THREE.Color(0xffffff);

const camera=new THREE.PerspectiveCamera(70,innerWidth/innerHeight,0.1,1000);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.outputColorSpace=THREE.SRGBColorSpace;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff,1));
const sun=new THREE.DirectionalLight(0xffffff,1.2);
sun.position.set(10,20,10);
scene.add(sun);

/* ================= GROUND ================= */
const groundMat=new THREE.MeshStandardMaterial({color:0xdddddd});
const ground=new THREE.Mesh(new THREE.PlaneGeometry(500,500),groundMat);
ground.rotation.x=-Math.PI/2;
scene.add(ground);

/* ================= PLAYER ================= */
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

// helmet grate
for(let i=-1;i<=1;i++){
  const slit=new THREE.Mesh(
    new THREE.BoxGeometry(.05,.35,.01),
    new THREE.MeshStandardMaterial({color:0x444444})
  );
  slit.position.set(i*.15,.65,.505);
  knight.add(slit);
}
scene.add(knight);

/* ================= WIZARD ================= */
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

const wizard=new THREE.Mesh(
  new THREE.ConeGeometry(1,2,32),
  wizardMaterial("#4b0082","#ffd700")
);
wizard.position.set(6,1,0);
scene.add(wizard);

/* ================= CHEESES ================= */
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

/* ================= RESET ================= */
function resetGame(){
  walkTimer=0;
  touchedCheese=false;

  knight.position.set(0,0,0);
  knight.rotation.y=0;
  wizard.position.set(6,1,0);

  goodCheese.position.set(-6,1,0);
}

/* ================= CONTROLS ================= */
let forward=0,turn=0;
const SPEED=.06,TURN=.04;

const joy=document.getElementById("joystick");
joy.ontouchend=()=>{forward=turn=0};
joy.ontouchmove=e=>{
  const r=joy.getBoundingClientRect();
  const t=e.touches[0];
  forward=-(t.clientY-r.top-60)/40;
  turn=(t.clientX-r.left-60)/40;
};

/* ================= LOOP ================= */
resetGame();
function animate(){
  requestAnimationFrame(animate);
  if(!started) return;

  goodCheese.rotation.y+=.04;

  knight.rotation.y-=turn*TURN;
  knight.position.x+=Math.sin(knight.rotation.y)*forward*SPEED;
  knight.position.z+=Math.cos(knight.rotation.y)*forward*SPEED;

  camera.position.set(
    knight.position.x-Math.sin(knight.rotation.y)*8,
    6,
    knight.position.z-Math.cos(knight.rotation.y)*8
  );
  camera.lookAt(knight.position.x,1,knight.position.z);

  if(!touchedCheese){
    walkTimer++;
    if(walkTimer>3600 && !gotWalkEnding){
      gotWalkEnding=true;
      document.getElementById("end-walk").classList.add("glow");
    }
  }

  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
