<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Cheese Triumph</title>
<style>
html,body{margin:0;overflow:hidden;background:#ddd;font-family:sans-serif}
#ui{position:fixed;top:0;left:0;width:100%;pointer-events:none}
#msg{position:absolute;top:10px;width:100%;text-align:center;font-weight:bold}
#timer{position:absolute;top:40px;width:100%;text-align:center;color:red}
#minimap{
  position:fixed;top:10px;right:10px;
  width:120px;height:120px;
  background:#222;border-radius:8px
}
#joystick{
  position:fixed;bottom:30px;left:30px;
  width:120px;height:120px;border-radius:50%;
  background:rgba(0,0,0,.3);display:none;pointer-events:auto
}
#stick{
  position:absolute;left:40px;top:40px;
  width:40px;height:40px;border-radius:50%;
  background:#aaa
}
#startScreen{
  position:fixed;inset:0;
  background:#ddd;display:flex;
  flex-direction:column;align-items:center;justify-content:center
}
#startBtn{
  background:#2979ff;color:white;
  padding:20px 40px;border-radius:12px;
  font-size:24px;cursor:pointer
}
h1{color:yellow;font-size:48px}
h1 span{color:#5a8f5a}
#endings{
  position:fixed;bottom:10px;left:50%;
  transform:translateX(-50%);
  display:flex;gap:20px;pointer-events:none
}
.orb{
  width:40px;height:40px;border-radius:50%;
  background:#444;opacity:.3
}
.orb.on{opacity:1;box-shadow:0 0 10px yellow}
</style>
</head>
<body>

<div id="startScreen">
  <h1><span>C</span>HEESE TRIUMPH</h1>
  <div id="startBtn">START</div>
</div>

<div id="ui">
  <div id="msg">GIVE THE GOOD CHEESE TO THE WIZARD!!! he likes them fresh</div>
  <div id="timer"></div>
</div>

<div id="minimap"></div>

<div id="joystick"><div id="stick"></div></div>

<div id="endings">
  <div id="orbGood" class="orb"></div>
  <div id="orbMouldy" class="orb"></div>
  <div id="orbWander" class="orb"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
/* ---------- DOM ---------- */
const startScreen = document.getElementById("startScreen");
const startBtn = document.getElementById("startBtn");
const joystick = document.getElementById("joystick");
const stick = document.getElementById("stick");
const msg = document.getElementById("msg");
const timerUI = document.getElementById("timer");
const minimap = document.getElementById("minimap");
const orbGood = document.getElementById("orbGood");
const orbMouldy = document.getElementById("orbMouldy");
const orbWander = document.getElementById("orbWander");

/* ---------- THREE ---------- */
const scene = new THREE.Scene();
scene.background = new THREE.Color("#ffffff");

const camera = new THREE.PerspectiveCamera(60,innerWidth/innerHeight,.1,500);
camera.position.set(0,5,10);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
document.body.appendChild(renderer.domElement);

/* ---------- WORLD ---------- */
const groundMat = new THREE.MeshLambertMaterial({color:"#ccc"});
const ground = new THREE.Mesh(new THREE.PlaneGeometry(500,500),groundMat);
ground.rotation.x = -Math.PI/2;
scene.add(ground);

const light = new THREE.DirectionalLight(0xffffff,1);
light.position.set(10,20,10);
scene.add(light);
scene.add(new THREE.AmbientLight(0xffffff,.5));

/* ---------- PLAYER ---------- */
const player = new THREE.Mesh(
  new THREE.BoxGeometry(1,1,1),
  new THREE.MeshLambertMaterial({color:"#bfbfbf"})
);
player.position.y=.5;
scene.add(player);

/* helmet base */
const base = new THREE.Mesh(
  new THREE.BoxGeometry(1,.3,1),
  new THREE.MeshLambertMaterial({color:"#555"})
);
base.position.y=-.35;
player.add(base);

/* face grate */
const grate = new THREE.Mesh(
  new THREE.PlaneGeometry(.6,.6),
  new THREE.MeshBasicMaterial({color:"#666"})
);
grate.position.z=.51;
player.add(grate);

/* ---------- WIZARD ---------- */
const wizard = new THREE.Mesh(
  new THREE.ConeGeometry(1,2,8),
  new THREE.MeshLambertMaterial({color:"indigo"})
);
wizard.position.set(10,1,10);
scene.add(wizard);

/* ---------- CHEESES ---------- */
function makeCheese(color){
  const g = new THREE.ConeGeometry(.5,1,3);
  const m = new THREE.MeshLambertMaterial({color});
  const c = new THREE.Mesh(g,m);
  c.rotation.x=Math.PI/2;
  return c;
}

const goodCheese = makeCheese("yellow");
goodCheese.position.set(-10,1,-10);
scene.add(goodCheese);

const mouldyCheese = makeCheese("#9acd32");
mouldyCheese.position.set(45,1,45);
scene.add(mouldyCheese);

/* ---------- GAME STATE ---------- */
let started=false;
let carrying=null;
let boss=false;
let bossTimer=0;
let immune=0;

/* ---------- START ---------- */
startBtn.onclick=()=>{
  startScreen.style.display="none";
  joystick.style.display="block";
  started=true;
};

/* ---------- INPUT ---------- */
let joy={x:0,y:0};
joystick.ontouchmove=e=>{
  const r=joystick.getBoundingClientRect();
  const t=e.touches[0];
  joy.x=(t.clientX-r.left-60)/40;
  joy.y=(t.clientY-r.top-60)/40;
};

/* ---------- LOOP ---------- */
function resetGame(){
  boss=false;
  bossTimer=0;
  immune=0;
  carrying=null;
  player.position.set(0,.5,0);
  wizard.position.set(10,1,10);
  goodCheese.position.set(-10,1,-10);
  mouldyCheese.position.set(45,1,45);
  scene.background=new THREE.Color("#fff");
  ground.material.color.set("#ccc");
  msg.textContent="GIVE THE GOOD CHEESE TO THE WIZARD!!! he likes them fresh";
  timerUI.textContent="";
}

function animate(){
  requestAnimationFrame(animate);
  if(!started) return;

  const speed = boss ? .045 : .05;
  player.position.x += joy.x*speed;
  player.position.z += joy.y*speed;

  if(joy.x||joy.y){
    player.rotation.y=Math.atan2(joy.x,joy.y);
    camera.position.lerp(
      new THREE.Vector3(
        player.position.x-joy.x*6,
        5,
        player.position.z-joy.y*6
      ),.1
    );
    camera.lookAt(player.position);
  }

  if(!carrying && player.position.distanceTo(goodCheese.position)<1){
    carrying="good";
  }
  if(!carrying && player.position.distanceTo(mouldyCheese.position)<1){
    carrying="mouldy";
    boss=true;
    immune=5;
    scene.background=new THREE.Color("#550000");
    ground.material.color.set("#444");
    msg.textContent="";
  }

  if(carrying){
    const target = carrying==="good"?wizard:player;
    (carrying==="good"?goodCheese:mouldyCheese)
      .position.lerp(target.position.clone().add(new THREE.Vector3(0,2,0)),.2);
  }

  if(boss){
    immune=Math.max(0,immune-.016);
    bossTimer+=.016;
    timerUI.textContent=(30-bossTimer).toFixed(1);
    wizard.position.lerp(player.position,.01);
    if(bossTimer>30){
      orbMouldy.classList.add("on");
      resetGame();
    }
  }

  renderer.render(scene,camera);
}
resetGame();
animate();
</script>
</body>
</html>
