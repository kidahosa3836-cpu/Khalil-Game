# Khalil-Game
Games for cells
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Ultimate Bioverse 3D RPG</title>
<style>
  body { margin: 0; overflow: hidden; background: black; }
  canvas { display: block; }
  #ui { position: absolute; top: 10px; width: 100%; text-align: center; color: white; font-family: Arial; }
  #feedback { color: yellow; font-weight: bold; }
  #gameOverMessage { color: red; font-weight: bold; font-size: 24px; }
  button { padding:5px; margin:2px; cursor:pointer; }
  .selected { box-shadow: 0 0 15px gold; transform: scale(1.05); }
</style>
</head>
<body>

<div id="ui">
Rank: <span id="rank">Novice</span> | Level: <span id="level">1</span> | XP: <span id="xp">0</span> | Streak: <span id="streak">0</span>
<br>
Domain:
<button onclick="selectDomain('Bacteria')">Bacteria</button>
<button onclick="selectDomain('Archaea')">Archaea</button>
<button onclick="selectDomain('Eukarya')">Eukarya</button>
<br>
Kingdom:
<button onclick="selectKingdom('Eubacteria')">Eubacteria</button>
<button onclick="selectKingdom('Archaebacteria')">Archaebacteria</button>
<button onclick="selectKingdom('Protista')">Protista</button>
<button onclick="selectKingdom('Fungi')">Fungi</button>
<button onclick="selectKingdom('Plantae')">Plantae</button>
<button onclick="selectKingdom('Animalia')">Animalia</button>
<br>
<button id="fireBtn" onclick="attemptFire()" disabled>⚡ FIRE LASER</button>
<p id="feedback"></p>
<p id="gameOverMessage"></p>
Question: <span id="questionText"></span>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.155.0/build/three.min.js"></script>
<script>

// --- THREE.JS 3D SETUP ---
let scene = new THREE.Scene();
let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 5000);
let renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Lighting
let light = new THREE.PointLight(0xffffff, 1);
light.position.set(0,0,500);
scene.add(light);

// Player ship
let playerGeo = new THREE.ConeGeometry(5, 20, 8);
let playerMat = new THREE.MeshBasicMaterial({color: 0x00ffff});
let player = new THREE.Mesh(playerGeo, playerMat);
player.rotation.x = Math.PI/2;
scene.add(player);
player.position.z = 0;

// Enemy ship
let enemyGeo = new THREE.BoxGeometry(15,10,10);
let enemyMat = new THREE.MeshBasicMaterial({color:0xff0000});
let enemy = new THREE.Mesh(enemyGeo, enemyMat);
scene.add(enemy);
enemy.position.set(0,0,200);

// Stars background
let starGeo = new THREE.BufferGeometry();
let starCount = 500;
let starVertices = [];
for(let i=0;i<starCount;i++){
  starVertices.push((Math.random()-0.5)*2000);
  starVertices.push((Math.random()-0.5)*2000);
  starVertices.push((Math.random()-0.5)*2000);
}
starGeo.setAttribute('position', new THREE.Float32BufferAttribute(starVertices,3));
let starMat = new THREE.PointsMaterial({color:0xffffff,size:2});
let stars = new THREE.Points(starGeo, starMat);
scene.add(stars);

// --- GAME LOGIC ---
let keys = {};
document.addEventListener('keydown', e=>keys[e.key]=true);
document.addEventListener('keyup', e=>keys[e.key]=false);

let lasers = [], enemyLasers = [], particles = [];

let xp=0, level=1, streak=0;
let selectedDomain=null, selectedKingdom=null;
let currentQuestion=null;
let gameOver=false;

// Organisms
const organisms=[
{domain:"Eukarya",kingdom:"Animalia",traits:"Multicellular, consumes prey"},
{domain:"Eukarya",kingdom:"Plantae",traits:"Multicellular, photosynthesizes"},
{domain:"Bacteria",kingdom:"Eubacteria",traits:"Unicellular, soil bacteria"},
{domain:"Archaea",kingdom:"Archaebacteria",traits:"Unicellular, extreme environments"},
{domain:"Eukarya",kingdom:"Fungi",traits:"Multicellular, absorbs nutrients"},
{domain:"Eukarya",kingdom:"Protista",traits:"Unicellular eukaryote"}
];

function loadQuestion(){
  currentQuestion=organisms[Math.floor(Math.random()*organisms.length)];
  document.getElementById("questionText").innerText=currentQuestion.traits;
  selectedDomain=null;
  selectedKingdom=null;
  document.querySelectorAll("#ui button").forEach(b=>b.classList.remove("selected"));
  document.getElementById("fireBtn").disabled=true;
  document.getElementById("feedback").innerText="";
}

// Select Domain/Kingdom
function selectDomain(d){selectedDomain=d; highlightSelected(); checkReady();}
function selectKingdom(k){selectedKingdom=k; highlightSelected(); checkReady();}
function highlightSelected(){
  document.querySelectorAll("#ui button").forEach(b=>b.classList.remove("selected"));
  document.querySelectorAll("#ui button").forEach(b=>{
    if(b.innerText===selectedDomain||b.innerText===selectedKingdom) b.classList.add("selected");
  });
}
function checkReady(){document.getElementById("fireBtn").disabled=!(selectedDomain&&selectedKingdom);}

// Fire laser
function attemptFire(){
  if(gameOver) return;
  if(selectedDomain===currentQuestion.domain && selectedKingdom===currentQuestion.kingdom){
    let laserGeo = new THREE.CylinderGeometry(0.5,0.5,20);
    let laserMat = new THREE.MeshBasicMaterial({color:0xffff00});
    let laser = new THREE.Mesh(laserGeo, laserMat);
    laser.rotation.x=Math.PI/2;
    laser.position.set(player.position.x,player.position.y,player.position.z+10);
    lasers.push(laser);
    scene.add(laser);
    enemy.position.z -= 20; // hit moves enemy closer
    streak++;
    xp+=10;
    document.getElementById("feedback").innerText="✅ Correct! Laser fired!";
  } else {
    let laserGeo = new THREE.CylinderGeometry(0.5,0.5,20);
    let laserMat = new THREE.MeshBasicMaterial({color:0xffa500});
    let laser = new THREE.Mesh(laserGeo, laserMat);
    laser.rotation.x=Math.PI/2;
    laser.position.set(enemy.position.x,enemy.position.y,enemy.position.z-10);
    enemyLasers.push(laser);
    scene.add(laser);
    player.position.z -= 10; // wrong answer hurts
    streak=0;
    document.getElementById("feedback").innerText="❌ Wrong! Enemy fires!";
  }
  updateUI();
  loadQuestion();
  document.getElementById("fireBtn").disabled=true;
}

// Update UI
function updateUI(){
  document.getElementById("xp").innerText = xp;
  document.getElementById("level").innerText = level;
  document.getElementById("streak").innerText = streak;
  document.getElementById("rank").innerText = (level>=10)?"Master":(level>=5)?"Expert":(level>=3)?"Researcher":"Novice";
}

// Restart
function restartGame(){
  gameOver=false;
  player.position.set(0,0,0);
  enemy.position.set(0,0,200);
  lasers.forEach(l=>scene.remove(l));
  enemyLasers.forEach(l=>scene.remove(l));
  lasers=[]; enemyLasers=[];
  xp=0; level=1; streak=0;
  document.getElementById("gameOverMessage").innerText="";
  loadQuestion();
}

// --- GAME LOOP ---
function animate(){
  if(gameOver){requestAnimationFrame(animate); return;}
  requestAnimationFrame(animate);

  // Player movement
  if(keys["ArrowUp"]||keys["w"]) player.position.y += 1;
  if(keys["ArrowDown"]||keys["s"]) player.position.y -= 1;
  if(keys["ArrowLeft"]||keys["a"]) player.position.x -= 1;
  if(keys["ArrowRight"]||keys["d"]) player.position.x += 1;

  // Camera follows player
  camera.position.set(player.position.x, player.position.y + 30, player.position.z - 50);
  camera.lookAt(player.position);

  // Move lasers
  lasers.forEach((l,i)=>{
    l.position.z += 5;
    if(l.position.z > 500){scene.remove(l); lasers.splice(i,1);}
  });
  enemyLasers.forEach((l,i)=>{
    l.position.z -= 5;
    if(l.position.z < -50){scene.remove(l); enemyLasers.splice(i,1);}
  });

  // Enemy simple AI
  enemy.position.x += Math.sin(Date.now()/500)*0.5;

  // Check collision (player hit)
  enemyLasers.forEach((l,i)=>{
    if(l.position.distanceTo(player.position)<5){
      scene.remove(l);
      enemyLasers.splice(i,1);
      player.position.z -= 5;
      if(player.position.z < -50){
        gameOver=true;
        document.getElementById("gameOverMessage").innerText = "💀 GAME OVER! Press F5 to Restart!";
      }
    }
  });

  renderer.render(scene,camera);
}

loadQuestion();
animate();

</script>
</body>
</html>
