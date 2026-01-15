<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Plez Chez</title>
<style>
  body { margin: 0; overflow: hidden; background: white; font-family: sans-serif; }
  #dialogue {
    position: absolute;
    bottom: 20%;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0,0,0,0.7);
    color: white;
    padding: 12px 20px;
    border-radius: 10px;
    display: none;
    font-size: 18px;
  }
  #joystick {
    position: absolute;
    bottom: 40px;
    left: 40px;
    width: 120px;
    height: 120px;
    background: rgba(0,0,0,0.2);
    border-radius: 50%;
    touch-action: none;
  }
  #stick {
    width: 50px;
    height: 50px;
    background: rgba(0,0,0,0.5);
    border-radius: 50%;
    position: absolute;
    top: 35px;
    left: 35px;
  }
</style>
</head>
<body>

<div id="dialogue"></div>

<div id="joystick">
  <div id="stick"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.158/build/three.min.js"></script>
<script>
/* ---------------- SCENE ---------------- */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xffffff);

const camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

/* ---------------- LIGHT ---------------- */
scene.add(new THREE.HemisphereLight(0xffffff, 0xaaaaaa, 1));
const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
dirLight.position.set(5,10,5);
scene.add(dirLight);

/* ---------------- GROUND ---------------- */
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(200, 200),
  new THREE.MeshStandardMaterial({ color: 0xcccccc })
);
ground.rotation.x = -Math.PI / 2;
scene.add(ground);

/* ---------------- PLAYER ---------------- */
const player = new THREE.Group();

// body
const body = new THREE.Mesh(
  new THREE.BoxGeometry(1,1,1),
  new THREE.MeshStandardMaterial({ color: 0x999999 })
);
player.add(body);

// dark stripe
const stripe = new THREE.Mesh(
  new THREE.BoxGeometry(1.02,0.3,1.02),
  new THREE.MeshStandardMaterial({ color: 0x444444 })
);
stripe.position.y = -0.35;
player.add(stripe);

// red feather
const feather = new THREE.Mesh(
  new THREE.ConeGeometry(0.1,0.5,8),
  new THREE.MeshStandardMaterial({ color: 0xff0000 })
);
feather.position.set(0,0.7,-0.3);
feather.rotation.x = Math.PI / 6;
player.add(feather);

player.position.set(0,0.5,0);
scene.add(player);

/* ---------------- WIZARD ---------------- */
const wizard = new THREE.Group();

const wizBody = new THREE.Mesh(
  new THREE.ConeGeometry(1,2,24),
  new THREE.MeshStandardMaterial({ color: 0x4b0082 })
);
wizard.add(wizBody);

// few yellow circles
const rings = 2;
const dotsPerRing = 6;

for(let r=0; r<rings; r++){
  const h = 0.4 + r * 0.6;
  const radius = 1 - h / 2;

  for(let i=0; i<dotsPerRing; i++){
    const angle = (i / dotsPerRing) * Math.PI * 2;
    const dot = new THREE.Mesh(
      new THREE.SphereGeometry(0.14,8,8),
      new THREE.MeshStandardMaterial({ color: 0xffff00 })
    );
    dot.position.set(
      Math.cos(angle) * radius,
      h - 1,
      Math.sin(angle) * radius
    );
    wizard.add(dot);
  }
}

wizard.position.set(5,1,0);
scene.add(wizard);

/* ---------------- CHEESE ---------------- */
const cheese = new THREE.Mesh(
  new THREE.ConeGeometry(0.6,0.8,4),
  new THREE.MeshStandardMaterial({ color: 0xffdd55 })
);
cheese.rotation.z = Math.PI / 2;
cheese.position.set(-6,1,0);
scene.add(cheese);

/* ---------------- STATE ---------------- */
let hasCheese = false;
let questComplete = false;

/* ---------------- DIALOGUE ---------------- */
const dialogue = document.getElementById("dialogue");
function showDialogue(text, time=1500){
  dialogue.textContent = text;
  dialogue.style.display = "block";
  setTimeout(()=>dialogue.style.display="none", time);
}

/* ---------------- JOYSTICK ---------------- */
let moveX = 0;
let moveZ = 0;
let dragging = false;

const joystick = document.getElementById("joystick");
const stick = document.getElementById("stick");

joystick.addEventListener("pointerdown", e=>{
  dragging = true;
  joystick.setPointerCapture(e.pointerId);
});

joystick.addEventListener("pointermove", e=>{
  if(!dragging) return;

  const rect = joystick.getBoundingClientRect();
  const x = e.clientX - rect.left - 60;
  const y = e.clientY - rect.top - 60;
  const max = 40;

  moveX = THREE.MathUtils.clamp(x / max, -1, 1);
  moveZ = THREE.MathUtils.clamp(-y / max, -1, 1); // UP = forward

  stick.style.left = `${35 + moveX * 40}px`;
  stick.style.top  = `${35 - moveZ * 40}px`;
});

joystick.addEventListener("pointerup", ()=>{
  dragging = false;
  moveX = 0;
  moveZ = 0;
  stick.style.left = "35px";
  stick.style.top = "35px";
});

/* ---------------- GAME LOOP ---------------- */
function animate(){
  requestAnimationFrame(animate);

  const moveSpeed = 0.12;
  const turnSpeed = 0.04;

  // turn player
  player.rotation.y -= moveX * turnSpeed;

  // move forward based on facing direction
  if(moveZ !== 0){
    const forward = new THREE.Vector3(0,0,1);
    forward.applyAxisAngle(new THREE.Vector3(0,1,0), player.rotation.y);
    player.position.addScaledVector(forward, moveZ * moveSpeed);
  }

  // camera behind player
  const camOffset = new THREE.Vector3(0,5,-8);
  camOffset.applyAxisAngle(new THREE.Vector3(0,1,0), player.rotation.y);
  camera.position.lerp(player.position.clone().add(camOffset), 0.1);
  camera.lookAt(player.position);

  // wizard interaction
  if(player.position.distanceTo(wizard.position) < 1.8){
    if(!hasCheese){
      showDialogue("Plez Chez");
    } else if(!questComplete){
      showDialogue("Yaey! Chez!");
      questComplete = true;

      const crown = new THREE.Mesh(
        new THREE.TorusGeometry(0.35,0.1,8,16),
        new THREE.MeshStandardMaterial({ color: 0xffd700 })
      );
      crown.position.y = 0.7;
      player.add(crown);
    }
  }

  // cheese pickup
  if(!hasCheese && player.position.distanceTo(cheese.position) < 1.5){
    hasCheese = true;
    scene.remove(cheese);
    showDialogue("Picked up cheese!");
  }

  renderer.render(scene, camera);
}

animate();

window.addEventListener("resize", ()=>{
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>
</body>
</html>
