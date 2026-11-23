<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Typhoon 3D</title>

<style>
*{margin:0;padding:0;box-sizing:border-box}
body{overflow:hidden;background:#87ceeb;font-family:Arial}
canvas{display:block}

#hud{
position:absolute;
top:15px;
left:15px;
color:white;
background:#0009;
padding:12px;
border-radius:10px;
font-weight:bold;
}

#controls{
position:absolute;
bottom:25px;
width:100%;
display:flex;
justify-content:space-between;
padding:0 25px;
}

.group{display:flex;gap:10px;align-items:end}

button{
width:65px;
height:65px;
border:2px solid white;
border-radius:50%;
background:#111b;
color:white;
font-size:23px;
touch-action:none;
}

#enter{
position:absolute;
bottom:110px;
right:30px;
width:85px;
height:55px;
border-radius:12px;
font-size:13px;
}
</style>
</head>

<body>

<div id="hud">
TYPHOON 3D 🌴<br>
MIAMI CITY<br>
<span id="status">Explore the city</span>
</div>

<button id="enter">ENTER CAR</button>

<div id="controls">
<div class="group">
<button id="left">◀</button>
<button id="right">▶</button>
</div>

<div class="group">
<button id="forward">▲</button>
<button id="back">▼</button>
</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);
scene.fog = new THREE.Fog(0x87ceeb,100,500);

const camera = new THREE.PerspectiveCamera(
70,window.innerWidth/window.innerHeight,0.1,1000
);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth,window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio,2));
renderer.shadowMap.enabled=true;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x777777,2));

const sun = new THREE.DirectionalLight(0xffffff,2);
sun.position.set(100,150,100);
scene.add(sun);

function box(x,y,z,w,h,d,color){
const mesh=new THREE.Mesh(
new THREE.BoxGeometry(w,h,d),
new THREE.MeshStandardMaterial({color})
);
mesh.position.set(x,y,z);
mesh.castShadow=true;
mesh.receiveShadow=true;
scene.add(mesh);
return mesh;
}

// CITY GROUND
box(0,-1,0,500,2,500,0x36a85a);

// OCEAN
const ocean=box(0,-0.7,-190,500,1,120,0x087fbd);

// BEACH
box(0,-0.3,-120,500,1,35,0xf4d58d);

// ROADS
box(0,0.05,0,18,0.2,500,0x333333);
box(0,0.05,0,500,0.2,18,0x333333);

// ROAD LINES
for(let i=-230;i<230;i+=15){
box(0,0.17,i,0.4,0.03,7,0xffffff);
box(i,0.17,0,7,0.03,0.4,0xffffff);
}

// BUILDINGS
const colors=[
0xf5a6a6,0xf1d38b,0xd5e9ee,
0xe5b5d5,0xb4d9e9,0xf6f0d7
];

for(let x=-100;x<=100;x+=30){
for(let z=-100;z<=100;z+=30){

if(Math.abs(x)<15 || Math.abs(z)<15)continue;

let height=15+Math.random()*55;

box(x,height/2,z,18,height,18,
colors[Math.floor(Math.random()*colors.length)]);

// WINDOWS
for(let y=5;y<height-3;y+=6){
box(x,y,z+9.1,10,2,0.2,0x79d8ef);
}

}
}

// PALM TREES
function palm(x,z){

const trunk=new THREE.Mesh(
new THREE.CylinderGeometry(0.5,0.8,12,8),
new THREE.MeshStandardMaterial({color:0x80552e})
);

trunk.position.set(x,6,z);
scene.add(trunk);

for(let i=0;i<7;i++){
let leaf=new THREE.Mesh(
new THREE.ConeGeometry(1.5,9,5),
new THREE.MeshStandardMaterial({color:0x138c3a})
);

leaf.position.set(x,12,z);
leaf.rotation.z=Math.PI/2;
leaf.rotation.y=i*Math.PI/3.5;
scene.add(leaf);
}
}

for(let i=-180;i<=180;i+=20){
palm(i,-105);
palm(i,105);
}

// PLAYER
const player=new THREE.Group();

const body=new THREE.Mesh(
new THREE.CapsuleGeometry(0.7,1.2,4,8),
new THREE.MeshStandardMaterial({color:0x1748df})
);
body.position.y=1.4;
player.add(body);

const head=new THREE.Mesh(
new THREE.SphereGeometry(0.5,16,16),
new THREE.MeshStandardMaterial({color:0xd69b72})
);
head.position.y=2.6;
player.add(head);

scene.add(player);
player.position.set(5,0,10);

// SPORTS CAR
const car=new THREE.Group();

const carBody=new THREE.Mesh(
new THREE.BoxGeometry(3,1,5),
new THREE.MeshStandardMaterial({color:0xe51b23})
);
carBody.position.y=1;
car.add(carBody);

const roof=new THREE.Mesh(
new THREE.BoxGeometry(2.4,1,2.4),
new THREE.MeshStandardMaterial({color:0x222b36})
);
roof.position.set(0,2,0);
car.add(roof);

for(let x of [-1.6,1.6]){
for(let z of [-1.6,1.6]){
let wheel=new THREE.Mesh(
new THREE.CylinderGeometry(0.55,0.55,0.4,16),
new THREE.MeshStandardMaterial({color:0x111111})
);
wheel.rotation.z=Math.PI/2;
wheel.position.set(x,0.55,z);
car.add(wheel);
}
}

scene.add(car);
car.position.set(10,0,5);

let driving=false;
let speed=0;
let rotation=0;

const keys={};

window.addEventListener("keydown",e=>{
keys[e.key.toLowerCase()]=true;
if(e.key.toLowerCase()==="e")toggleCar();
});

window.addEventListener("keyup",e=>{
keys[e.key.toLowerCase()]=false;
});

// MOBILE CONTROLS
function hold(id,key){
const btn=document.getElementById(id);

btn.addEventListener("pointerdown",e=>{
e.preventDefault();
keys[key]=true;
});

btn.addEventListener("pointerup",()=>keys[key]=false);
btn.addEventListener("pointerleave",()=>keys[key]=false);
btn.addEventListener("pointercancel",()=>keys[key]=false);
}

hold("forward","w");
hold("back","s");
hold("left","a");
hold("right","d");

function toggleCar(){
if(!driving){
if(player.position.distanceTo(car.position)<9){
driving=true;
player.visible=false;
document.getElementById("status").innerText="Driving sports car";
}
}else{
driving=false;
player.visible=true;
player.position.copy(car.position);
player.position.x+=4;
speed=0;
document.getElementById("status").innerText="Exploring Miami";
}
}

document.getElementById("enter").onclick=toggleCar;

// GAME LOOP
function animate(){
requestAnimationFrame(animate);

if(driving){

if(keys.w)speed=Math.min(speed+0.012,0.7);
if(keys.s)speed=Math.max(speed-0.02,-0.35);
if(!keys.w&&!keys.s)speed*=0.97;

if(keys.a)rotation+=0.025;
if(keys.d)rotation-=0.025;

car.rotation.y=rotation;

car.position.x+=Math.sin(rotation)*speed;
car.position.z+=Math.cos(rotation)*speed;

}else{

let movement=0;

if(keys.w)movement=0.25;
if(keys.s)movement=-0.25;

if(keys.a)player.rotation.y+=0.04;
if(keys.d)player.rotation.y-=0.04;

player.position.x+=Math.sin(player.rotation.y)*movement;
player.position.z+=Math.cos(player.rotation.y)*movement;
}

// CAMERA
let target=driving?car:player;

let distance=driving?12:9;

let camX=target.position.x-Math.sin(target.rotation.y)*distance;
let camZ=target.position.z-Math.cos(target.rotation.y)*distance;

camera.position.lerp(
new THREE.Vector3(camX,target.position.y+6,camZ),
0.08
);

camera.lookAt(
target.position.x,
target.position.y+2,
target.position.z
);

renderer.render(scene,camera);
}

animate();

window.addEventListener("resize",()=>{
camera.aspect=window.innerWidth/window.innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(window.innerWidth,window.innerHeight);
});

</script>
</body>
</html>
