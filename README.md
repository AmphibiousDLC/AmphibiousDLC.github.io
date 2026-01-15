<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Knight & Cheese</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body { margin:0; overflow:hidden; background:white; touch-action:none; font-family:sans-serif; }

#joystick {
  position:fixed; bottom:30px; left:30px;
  width:120px; height:120px;
  border-radius:50%;
  background:rgba(0,0,0,0.25);
}
#stick {
  position:absolute; left:40px; top:40px;
  width:40px; height:40px;
  border-radius:50%;
  background:rgba(0,0,0,0.6);
}

#lookZone { position:fixed; right:0; top:0; width:50vw; height:100vh; }

#minimap {
  position:fixed; top:20px; right:20px;
  width:140px; height:140px;
  background:#eee;
  border:2px solid #999;
}

#fade, #flash {
  position:fixed; inset:0; pointer-events:none;
}
#fade { background:black; opacity:0; transition:opacity .6s; }
#flash { background:red; opacity:0; }

#msg {
  position:fixed;
  top:10px;
  width:100%;
  text-align:center;
  font-weight:bold;
  font-size:18px;
  color:black;
}
#msg span { font-size:14px; font-weight:normal; }
</style>
</head>
<body>

<div id="msg">
  GIVE THE GOOD CHEESE TO THE WIZARD!!!<br>
  <span>he likes them fresh</span>
</div>

<div id="joystick"><div id="stick"></div></div>
<div id="lookZone"></div>
<canvas id="minimap" width="140" height="140"></canvas>
<div id="fade"></div>
<div id="flash"></div>

<script src="https://cdn.jsdelivr.net/npm/three@0.155.0/build/three.min.js"></script>
<script>
/* ---------- CONSTANTS ---------- */
const MOVE_SPEED = 0.04;
const TURN_SPEED = 0.12;
const CAM_DIST = 8;
const MAP_SIZE = 50;

/* ---------- STATE ---------- */
let scene,camera,renderer;
let knight,wizard,cheese,mouldy,ground;
let lava,path;
let aura;
let moveForward=0,turnAmount=0,camYaw=0;
let hasCheese=false,hasMould=false,evil=false;
let lightningTimer;

/* ---------- MINIMAP ---------- */
const ctx=document.getElementById("minimap").getContext("2d");

/* ---------- INIT ---------- */
init(); animate();

function init(){
  scene=new THREE.Scene();
  scene.background=new THREE.Color(0xffffff);

  camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,0.1,1000);
  renderer=new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(innerWidth,innerHeight);
  document.body.appendChild(renderer.domElement);

  scene.add(new THREE.AmbientLight(0xffffff,0.7));
  const sun=new THREE.DirectionalLight(0xffffff,0.8);
  sun.position.set(10,20,10);
  scene.add(sun);

  ground=new THREE.Mesh(
    new THREE.PlaneGeometry(100,100),
    new THREE.MeshStandardMaterial({color:0xdddddd})
  );
  ground.rotation.x=-Math.PI/2;
  scene.add(ground);

  knight=new THREE.Mesh(
    new THREE.BoxGeometry(1,1,1),
    new THREE.MeshStandardMaterial({color:0x888888})
  );
  knight.position.y=0.5;
  scene.add(knight);

  wizard=new THREE.Mesh(
    new THREE.ConeGeometry(1,2,32),
    wizardMat("#4b0082","#FFD700")
  );
  wizard.position.set(10,1,10);
  scene.add(wizard);

  spawnCheese();
  spawnMouldy();
  setupJoystick();
  setupLook();
}

/* ---------- MATERIAL ---------- */
function wizardMat(base,spot){
  const c=document.createElement("canvas");
  c.width=c.height=512;
  const x=c.getContext("2d");
  x.fillStyle=base; x.fillRect(0,0,512,512);
  x.fillStyle=spot;
  for(let y=0;y<3;y++)for(let x2=0;x2<4;x2++){
    x.beginPath();
    x.arc((x2+.5)*128,(y+.5)*170,26,0,Math.PI*2);
    x.fill();
  }
  return new THREE.MeshStandardMaterial({map:new THREE.CanvasTexture(c)});
}

/* ---------- CHEESE ---------- */
function spawnCheese(){
  cheese=new THREE.Mesh(
    new THREE.ConeGeometry(0.5,1,3),
    new THREE.MeshStandardMaterial({color:0xffd700,emissive:0xffff66})
  );
  cheese.rotation.x=Math.PI;
  cheese.position.set(-8,1,-8);
  scene.add(cheese);

  aura=new THREE.Mesh(
    new THREE.SphereGeometry(0.9,16,16),
    new THREE.MeshBasicMaterial({color:0xffff66,transparent:true,opacity:.3})
  );
  cheese.add(aura);
}

/* ---------- MOULDY ---------- */
function spawnMouldy(){
  mouldy=new THREE.Mesh(
    new THREE.ConeGeometry(0.3,0.6,3),
    new THREE.MeshStandardMaterial({color:0xc2a000})
  );
  mouldy.rotation.x=Math.PI;
  mouldy.position.set(-23,1,-23);
  scene.add(mouldy);
}

/* ---------- CONTROLS ---------- */
function setupJoystick(){
  const joy=document.getElementById("joystick");
  const stick=document.getElementById("stick");
  let drag=false;
  joy.ontouchstart=()=>drag=true;
  joy.ontouchend=()=>{drag=false;moveForward=turnAmount=0;stick.style.left="40px";stick.style.top="40px";}
  joy.ontouchmove=e=>{
    if(!drag)return;
    const r=joy.getBoundingClientRect(),t=e.touches[0];
    const x=t.clientX-r.left-60, y=t.clientY-r.top-60;
    stick.style.left=60+x*.5-20+"px";
    stick.style.top=60+y*.5-20+"px";
    moveForward=-y/40; turnAmount=x/40;
  };
}

function setupLook(){
  const z=document.getElementById("lookZone");
  let last=null;
  z.ontouchstart=e=>last=e.touches[0].clientX;
  z.ontouchmove=e=>{
    camYaw-=(e.touches[0].clientX-last)*0.005;
    last=e.touches[0].clientX;
  };
}

/* ---------- LOOP ---------- */
function animate(){
  requestAnimationFrame(animate);

  cheese.rotation.y+=0.05;
  aura.material.opacity=.3+Math.sin(Date.now()*0.004)*.1;

  knight.rotation.y-=turnAmount*TURN_SPEED;
  knight.position.x+=Math.sin(knight.rotation.y)*moveForward*MOVE_SPEED;
  knight.position.z+=Math.cos(knight.rotation.y)*moveForward*MOVE_SPEED;

  camera.position.set(
    knight.position.x-Math.sin(knight.rotation.y+camYaw)*8,
    6,
    knight.position.z-Math.cos(knight.rotation.y+camYaw)*8
  );
  camera.lookAt(knight.position);

  if(!hasCheese && dist(knight,cheese)<1.2) hasCheese=true;
  if(!hasMould && dist(knight,mouldy)<1.2) hasMould=true;

  if(hasCheese) cheese.position.copy(knight.position).add(new THREE.Vector3(0,1.5,0));
  if(hasMould) mouldy.position.copy(knight.position).add(new THREE.Vector3(0,1.5,0));

  if(hasMould && !evil && dist(knight,wizard)<1.5) evilEvent();
  if(hasCheese && evil && dist(knight,wizard)<1.5) location.reload();

  drawMinimap();
  renderer.render(scene,camera);
}

/* ---------- EVIL MODE ---------- */
function evilEvent(){
  evil=true;
  document.getElementById("msg").style.display="none";
  document.getElementById("fade").style.opacity=1;

  setTimeout(()=>{
    document.getElementById("flash").style.opacity=.8;
    setTimeout(()=>document.getElementById("flash").style.opacity=0,200);
    document.getElementById("fade").style.opacity=0;

    scene.background.set(0x333333);
    ground.material.color.set(0x550000);
    wizard.scale.set(3,3,3);
    wizard.material=wizardMat("#8b0000","#00008b");
    scene.remove(mouldy);
    spawnLavaPath();
    lightningTimer=setInterval(spawnLightning,3500);
  },800);
}

/* ---------- LAVA PATH ---------- */
function spawnLavaPath(){
  lava=new THREE.Mesh(
    new THREE.PlaneGeometry(12,12),
    new THREE.MeshStandardMaterial({color:0x990000,emissive:0xff3300})
  );
  lava.rotation.x=-Math.PI/2;
  lava.position.set(-8,0.02,-8);
  scene.add(lava);

  path=new THREE.Mesh(
    new THREE.BoxGeometry(1,0.2,10),
    new THREE.MeshStandardMaterial({color:0x777777})
  );
  path.position.set(-8,0.2,-8);
  scene.add(path);
}

/* ---------- LIGHTNING ---------- */
function spawnLightning(){
  const bolt=new THREE.Mesh(
    new THREE.BoxGeometry(0.6,12,0.6),
    new THREE.MeshBasicMaterial({color:0xffff00})
  );
  bolt.position.set(
    knight.position.x+(Math.random()-0.5)*4,
    8,
    knight.position.z+(Math.random()-0.5)*4
  );
  scene.add(bolt);

  let t=0;
  const drop=setInterval(()=>{
    bolt.position.y-=0.8;
    bolt.material.color.setHex(0xffff00+Math.random()*0x1111);
    if(++t>15){
      if(dist(knight,bolt)<1){
        alert("You were struck by lightning!");
        location.reload();
      }
      scene.remove(bolt);
      clearInterval(drop);
    }
  },50);
}

/* ---------- HELPERS ---------- */
function dist(a,b){
  return Math.hypot(a.position.x-b.position.x,a.position.z-b.position.z);
}
function drawMinimap(){
  ctx.clearRect(0,0,140,140);
  dot(knight,"#888");
  dot(wizard,"#4b0082");
  if(!hasCheese) dot(cheese,"#ffd700");
}
function dot(o,c){
  ctx.fillStyle=c;
  ctx.beginPath();
  ctx.arc((o.position.x/MAP_SIZE+.5)*140,(o.position.z/MAP_SIZE+.5)*140,5,0,Math.PI*2);
  ctx.fill();
}
</script>
</body>
</html>
