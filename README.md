<!DOCTYPE html>
<html lang="he">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>סרטון קרב תלת-ממדי</title>
<style>
body { margin:0; overflow:hidden; background:#111; }
canvas { display:block; }
</style>
</head>
<body>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r134/three.min.js"></script>
<script>

// --- סצנה ומצלמה ---
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x222222);
const camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(0,10,20);
camera.lookAt(0,2,0);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// --- תאורה ---
const ambient = new THREE.AmbientLight(0xffffff,0.8);
scene.add(ambient);
const dirLight = new THREE.DirectionalLight(0xffffff,1);
dirLight.position.set(10,20,10);
scene.add(dirLight);

// --- זירה ---
const ring = new THREE.Group();
scene.add(ring);
const base = new THREE.Mesh(new THREE.BoxGeometry(16,0.8,16), new THREE.MeshStandardMaterial({color:0x444466}));
base.position.y=0.4; ring.add(base);
const mat = new THREE.Mesh(new THREE.PlaneGeometry(14,14), new THREE.MeshStandardMaterial({color:0x2b2b7a}));
mat.rotation.x=-Math.PI/2; mat.position.y=0.81; ring.add(mat);

// --- קהל (אצטדיון) ---
for(let r=0;r<12;r++){
  for(let c=0;c<35;c++){
    const box = new THREE.Mesh(
      new THREE.BoxGeometry(0.5,0.8,0.3),
      new THREE.MeshStandardMaterial({color:new THREE.Color().setHSL(Math.random()*0.3+0.1,0.7,0.4)})
    );
    box.position.set((c-17.5)*0.5,0.4+r*0.5,-15-r*0.5);
    scene.add(box);
  }
}

// --- פונקציה ליצירת לוחם ---
function createFighter(x,color,hairColor,moustacheColor){
  const g = new THREE.Group();
  g.position.x=x;

  // גוף וראש
  const body = new THREE.Mesh(new THREE.CylinderGeometry(0.5,0.5,1.5,16), new THREE.MeshStandardMaterial({color}));
  body.position.y=1; g.add(body);
  const head = new THREE.Mesh(new THREE.SphereGeometry(0.38,16,12), new THREE.MeshStandardMaterial({color:0xf4d4b4}));
  head.position.y=2.05; g.add(head);

  // שיער ושפם
  const hair = new THREE.Mesh(new THREE.BoxGeometry(0.9,0.3,0.9), new THREE.MeshStandardMaterial({color:hairColor}));
  hair.position.y=2.35; g.add(hair);
  const moustache = new THREE.Mesh(new THREE.BoxGeometry(0.5,0.08,0.12), new THREE.MeshStandardMaterial({color:moustacheColor}));
  moustache.position.set(0,1.95,0.35); g.add(moustache);

  // ידיים עם אצבעות
  function makeArm(side){
    const armGroup = new THREE.Group();
    const upper = new THREE.Mesh(new THREE.CylinderGeometry(0.12,0.12,0.45,12), new THREE.MeshStandardMaterial({color}));
    upper.position.y=0.7; upper.rotation.z=side==='left'?Math.PI/8:-Math.PI/8; armGroup.add(upper);
    const fore = new THREE.Mesh(new THREE.CylinderGeometry(0.11,0.11,0.4,12), new THREE.MeshStandardMaterial({color}));
    fore.position.y=0.4; fore.position.z=side==='left'?0.15:-0.15; fore.rotation.z=side==='left'?Math.PI/4:-Math.PI/4; armGroup.add(fore);
    const hand = new THREE.Mesh(new THREE.SphereGeometry(0.12,8,8), new THREE.MeshStandardMaterial({color:0xf4d4b4}));
    hand.position.set(side==='left'?-0.42:0.42,0.1,0); armGroup.add(hand);
    for(let i=0;i<4;i++){
      const f=new THREE.Mesh(new THREE.CylinderGeometry(0.03,0.03,0.18,8),new THREE.MeshStandardMaterial({color:0xf4d4b4}));
      f.rotation.x=Math.PI/2; f.position.set((i-1.5)*0.04 + (side==='left'?-0.02:0.02),-0.04,0.12); hand.add(f);
    }
    armGroup.position.y=1.3; armGroup.position.x=side==='left'?-0.6:0.6;
    return armGroup;
  }
  g.leftArm=makeArm('left'); g.rightArm=makeArm('right');
  g.add(g.leftArm,g.rightArm);

  // רגליים
  const leftLeg = new THREE.Mesh(new THREE.CylinderGeometry(0.14,0.14,0.6,12), new THREE.MeshStandardMaterial({color}));
  leftLeg.position.set(-0.2,0.3,0); g.add(leftLeg);
  const rightLeg = leftLeg.clone(); rightLeg.position.x=0.2; g.add(rightLeg);

  return g;
}

// --- לוחמים ---
const fighter1=createFighter(-4,0xff3333,0x3a2b1f,0x000000);
const fighter2=createFighter(4,0x3355ff,0xeeee22,0x884400);
scene.add(fighter1,fighter2);

// --- סאונד ---
const listener = new THREE.AudioListener(); camera.add(listener);
const audioLoader = new THREE.AudioLoader();
const crowdSound = new THREE.Audio(listener);
audioLoader.load('https://www.soundjay.com/human/crowd-cheer-1.mp3',buffer=>{crowdSound.setBuffer(buffer);crowdSound.setLoop(true);crowdSound.setVolume(0.3);crowdSound.play();});
const punchSound = new THREE.Audio(listener);
audioLoader.load('https://www.soundjay.com/human/punch-1.mp3',buffer=>{punchSound.setBuffer(buffer);punchSound.setVolume(0.7);});
const heavyHit = new THREE.Audio(listener);
audioLoader.load('https://www.soundjay.com/mechanical/slam-1.mp3',buffer=>{heavyHit.setBuffer(buffer);heavyHit.setVolume(0.9);});

// --- פאואר פאצ' ---
const glow = new THREE.Mesh(new THREE.SphereGeometry(0.6,12,8), new THREE.MeshBasicMaterial({color:0xffdd44, transparent:true, opacity:0}));
glow.position.set(0,1.6,0); scene.add(glow);

// --- אנימציה קרב ---
const clock = new THREE.Clock();
let time=0;
function animate(){
  requestAnimationFrame(animate);
  const dt=clock.getDelta();
  time+=dt;

  // לוחמים מסתכלים אחד על השני
  fighter1.lookAt(fighter2.position.x,1.5,fighter2.position.z);
  fighter2.lookAt(fighter1.position.x,1.5,fighter1.position.z);

  // קרב
  if(time<3){
    fighter1.position.x+=0.02; fighter2.position.x-=0.02;
    fighter1.rightArm.rotation.z=Math.sin(time*5)*0.6-0.4;
    fighter2.leftArm.rotation.z=Math.sin(time*5)*0.6-0.4;
  } else if(time<6){
    fighter1.rightArm.rotation.z=-1.2; fighter2.position.z-=0.3;
    if(!punchSound.isPlaying)punchSound.play(); glow.material.opacity=0.7;
  } else if(time<9){
    fighter2.leftArm.rotation.z=1.2; fighter1.position.z+=0.3;
    if(!punchSound.isPlaying)punchSound.play(); glow.material.opacity=0.7;
  } else if(time<12){
    fighter1.rightArm.rotation.z=Math.sin(time*10)*0.5;
    fighter2.leftArm.rotation.z=-Math.sin(time*10)*0.5;
    if(!heavyHit.isPlaying)heavyHit.play(); glow.material.opacity=0.9;
  } else{
    time=0;
    fighter1.position.set(-4,0,0); fighter2.position.set(4,0,0);
    glow.material.opacity=0;
  }

  // מצלמה מסתובבת סביב הזירה
  const camAngle=Math.sin(clock.getElapsedTime()*0.25)*0.35;
  const camRadius=20;
  camera.position.x=Math.sin(camAngle)*camRadius;
  camera.position.z=Math.cos(camAngle)*camRadius;
  camera.position.y=10;
  camera.lookAt(0,2,0);

  renderer.render(scene,camera);
}
animate();

// --- התאמת מסך ---
window.addEventListener('resize',()=>{camera.aspect=window.innerWidth/window.innerHeight;camera.updateProjectionMatrix();renderer.setSize(window.innerWidth,window.innerHeight);});

</script>
</body>
</html>
