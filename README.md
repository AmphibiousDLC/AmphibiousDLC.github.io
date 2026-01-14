<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>White World with Pink Throne & Harriet</title>
<style>
body{margin:0;overflow:hidden;font-family:Arial;}
#ui{position:absolute;top:10px;left:10px;background:rgba(0,0,0,.6);color:#fff;padding:10px;border-radius:6px;z-index:10}
#prayBtn{position:absolute;bottom:40px;right:40px;width:90px;height:90px;border-radius:50%;background:#8e44ad;color:white;border:none;font-size:16px;z-index:10}
#joystick{position:absolute;bottom:20px;left:20px;width:120px;height:120px;background:rgba(255,255,255,.2);border-radius:50%;z-index:10}
#stick{width:50px;height:50px;background:#fff;border-radius:50%;position:absolute;left:35px;top:35px}
</style>
</head>
<body>
<div id="ui">Use joystick to move</div>
<button id="prayBtn">PRAY</button>
<div id="joystick"><div id="stick"></div></div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>

// ================= SCENE =================
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xffffff);

const camera = new THREE.PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 1000);
camera.position.set(0, 3, 8);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

const ambient = new THREE.AmbientLight(0xffffff, 0.8);
scene.add(ambient);

const sun = new THREE.DirectionalLight(0xffffff, 0.5);
sun.position.set(10,10,10);
scene.add(sun);

// ================= PLAYER =================
const player = new THREE.Group();
player.position.set(0,0,5);

const skinMat = new THREE.MeshStandardMaterial({color:0xffd2b0});
const clothMat = new THREE.MeshStandardMaterial({color:0x3498db}); // blue
const body = new THREE.Mesh(new THREE.BoxGeometry(.8,1,.5), clothMat);
body.position.y=1; player.add(body);
const head = new THREE.Mesh(new THREE.BoxGeometry(.7,.7,.7), skinMat);
head.position.y=1.8; player.add(head);

// Face
const eyeMat = new THREE.MeshStandardMaterial({color:0x000000});
const leftEye = new THREE.Mesh(new THREE.BoxGeometry(.1,.1,.05), eyeMat); leftEye.position.set(-.15,1.85,.36);
const rightEye = new THREE.Mesh(new THREE.BoxGeometry(.1,.1,.05), eyeMat); rightEye.position.set(.15,1.85,.36);
const mouth = new THREE.Mesh(new THREE.BoxGeometry(.25,.05,.05), eyeMat); mouth.position.set(0,1.65,.36);
player.add(leftEye,rightEye,mouth);

// Limbs
function limb(x,y,colorMat){const l=new THREE.Mesh(new THREE.BoxGeometry(.2,.7,.2), colorMat);l.position.set(x,y,0);player.add(l);return l;}
const leftArm=limb(-.5,1.2,clothMat), rightArm=limb(.5,1.2,clothMat);
const leftLeg=limb(-.2,0.35,clothMat), rightLeg=limb(.2,0.35,clothMat);

scene.add(player);

// ================= THRONE =================
const throne = new THREE.Group();
const pinkMat = new THREE.MeshStandardMaterial({color:0xff69b4});
const goldMat = new THREE.MeshStandardMaterial({color:0xffd700});

// Base seat
const seat = new THREE.Mesh(new THREE.BoxGeometry(3,1,2), pinkMat);
seat.position.y=0.5; throne.add(seat);

// Backrest
const back = new THREE.Mesh(new THREE.BoxGeometry(3,4,.5), pinkMat);
back.position.set(0,2.5,-0.75); throne.add(back);

// Golden top
const goldTop = new THREE.Mesh(new THREE.BoxGeometry(3.2,.2,.7), goldMat);
goldTop.position.set(0,4.6,-0.75); throne.add(goldTop);

// Armrests
const armL = new THREE.Mesh(new THREE.BoxGeometry(.3,1,.5), pinkMat);
armL.position.set(-1.65,1,0); throne.add(armL);
const armR = new THREE.Mesh(new THREE.BoxGeometry(.3,1,.5), pinkMat);
armR.position.set(1.65,1,0); throne.add(armR);

// ================= HARriet =================
const harriet = new THREE.Group();

// Clothes: darker pink than throne
const hClothMat = new THREE.MeshStandardMaterial({color:0xff1493});
const hBody = new THREE.Mesh(new THREE.BoxGeometry(.6,1,.4), hClothMat);
hBody.position.y=0.5; harriet.add(hBody);
const hHead = new THREE.Mesh(new THREE.BoxGeometry(.5,.5,.5), skinMat);
hHead.position.y=1.05; harriet.add(hHead);

// Limbs
function hLimb(x,y){return new THREE.Mesh(new THREE.BoxGeometry(.15,.6,.15), hClothMat);}
const hLeftArm = hLimb(-0.35,0.8); hLeftArm.position.set(-0.35,0.8,0); harriet.add(hLeftArm);
const hRightArm = hLimb(0.35,0.8); hRightArm.position.set(0.35,0.8,0); harriet.add(hRightArm);
const hLeftLeg = hLimb(-0.15,0.0); hLeftLeg.position.set(-0.15,0,0); harriet.add(hLeftLeg);
const hRightLeg = hLimb(0.15,0.0); hRightLeg.position.set(0.15,0,0); harriet.add(hRightLeg);

harriet.position.set(0,1,0);
throne.add(harriet);

scene.add(throne);
throne.position.set(0,0,0);

// ================= PRAY ABILITY =================
let praying = false;
const prayBtn = document.getElementById("prayBtn");
prayBtn.onclick = () => {
    if(praying) return;
    praying = true;

    // Animate player head down
    let t=0;
    const prayAnim = setInterval(()=>{
        t+=0.05;
        player.rotation.x = Math.sin(t)*0.5;
        if(t>=Math.PI){
            clearInterval(prayAnim);
            player.rotation.x=0;
            praying=false;
        }
    },16);
};

// ================= JOYSTICK =================
let joyX=0, joyY=0, drag=false;
const joystick=document.getElementById("joystick");
const stick=document.getElementById("stick");

joystick.ontouchstart = ()=>drag=true;
joystick.ontouchend = ()=>{drag=false; joyX=0; joyY=0; stick.style.left="35px"; stick.style.top="35px";}
joystick.ontouchmove = e => {
  if(!drag) return;
  const r=joystick.getBoundingClientRect(), t=e.touches[0];
  let x = t.clientX - r.left - 60;
  let y = t.clientY - r.top - 60;
  const d = Math.min(40, Math.hypot(x,y)), a=Math.atan2(y,x);
  joyX = Math.cos(a)*(d/40); joyY = Math.sin(a)*(d/40);
  stick.style.left = 35 + joyX*40 + "px";
  stick.style.top = 35 + joyY*40 + "px";
};

// ================= ANIMATE =================
function animate(){
    requestAnimationFrame(animate);

    if(!praying){
        // Move player
        player.position.x += joyX*0.12;
        player.position.z += joyY*0.12;
        if(joyX!=0 || joyY!=0) player.rotation.y = Math.atan2(joyX, joyY);
    }

    // Limb animation
    const t=performance.now()*0.002;
    leftArm.rotation.x=Math.sin(t)*0.4; rightArm.rotation.x=-Math.sin(t)*0.4;
    leftLeg.rotation.x=-Math.sin(t)*0.4; rightLeg.rotation.x=Math.sin(t)*0.4;

    // Camera follows
    camera.position.x = player.position.x;
    camera.position.z = player.position.z + 8;
    camera.position.y = player.position.y + 3;
    camera.lookAt(player.position);

    renderer.render(scene,camera);
}
animate();

</script>
</body>
</html>
