<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Space Arcade Mobile</title>
<style>
body{margin:0;background:black;overflow:hidden;touch-action:none}
canvas{display:block;margin:auto}
#menu{
  position:absolute;inset:0;
  display:flex;justify-content:center;align-items:center;
  z-index:5;
}
button.start{font-size:26px;padding:15px 50px;cursor:pointer}
#ui{
  position:absolute;top:10px;left:10px;
  color:white;font-family:Arial;font-size:14px;z-index:4
}

/* أزرار الموبايل */
#controls{
  position:absolute;bottom:15px;left:0;right:0;
  display:flex;justify-content:space-between;
  padding:0 20px;z-index:6
}
.ctrl{
  width:70px;height:70px;
  background:rgba(255,255,255,0.15);
  border-radius:50%;
  display:flex;align-items:center;justify-content:center;
  color:white;font-size:30px;
  user-select:none;
}
.fire{
  background:rgba(255,0,0,0.3);
}
</style>
</head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<body>

<div id="menu">
  <button class="start" onclick="startGame()">ابدأ</button>
</div>

<div id="ui"></div>
<canvas id="game" width="400" height="600"></canvas>

<div id="controls">
  <div class="ctrl" id="left">⬅️</div>
  <div class="ctrl fire" id="fire">🔴</div>
  <div class="ctrl" id="right">➡️</div>
</div>

<script>
const c=document.getElementById("game");
const x=c.getContext("2d");
const ui=document.getElementById("ui");
const menu=document.getElementById("menu");

let keys={},touch={left:false,right:false,fire:false};
let stars=[],aliens=[],lasers=[],explosions=[];
let score=0,highScore=localStorage.hs||0;
let speed=2,spawnRate=0.02;
let ammo=100,shield=2;
let lastAmmo=Date.now(),lastShield=Date.now();
let playing=false;

// مستويات لونية
const levels=[
  {bg:"#020b2d",star:"#6cf"},
  {bg:"#1b0033",star:"#d6a6ff"},
  {bg:"#2b0000",star:"#ff7b7b"},
  {bg:"#002b1f",star:"#7bffce"}
];

// صاروخ
const ship={x:180,y:500,w:40,h:60};

// كيبورد
addEventListener("keydown",e=>keys[e.key]=true);
addEventListener("keyup",e=>keys[e.key]=false);

// لمس
function bind(id,key){
  const el=document.getElementById(id);
  el.ontouchstart=()=>touch[key]=true;
  el.ontouchend=()=>touch[key]=false;
}
bind("left","left");
bind("right","right");
bind("fire","fire");

// نجوم
stars=Array.from({length:80},()=>({x:Math.random()*400,y:Math.random()*600,s:Math.random()*2+1}));

function spawnAlien(){
  let r=Math.random();
  aliens.push({
    x:Math.random()*360,y:-40,s:30,v:speed,
    type:r<0.33?"normal":r<0.66?"zig":"hunter",
    dir:Math.random()<0.5?-1:1
  });
}

function shoot(){
  if(ammo<=0) return;
  lasers.push({x:ship.x+20,y:ship.y});
  ammo--;
}

function explode(xp,yp){
  explosions.push({x:xp,y:yp,r:2});
}

function resetGame(){
  aliens=[];lasers=[];explosions=[];
  score=0; speed=2; spawnRate=0.02;
  ammo=100; shield=2;
  ship.x=180;
}

function startGame(){
  menu.style.display="none";
  playing=true;
  resetGame();
}

function update(){
  if(!playing) return;

  if((keys["ArrowLeft"]||touch.left)&&ship.x>0) ship.x-=4;
  if((keys["ArrowRight"]||touch.right)&&ship.x<360) ship.x+=4;
  if((keys[" "]||touch.fire)&&Math.random()<0.25) shoot();

  if(Math.random()<spawnRate) spawnAlien();

  aliens.forEach((a,i)=>{
    a.y+=a.v;
    if(a.type==="zig"){ a.x+=a.dir*2; if(Math.random()<0.02)a.dir*=-1; }
    if(a.type==="hunter"){ a.x+=Math.sign(ship.x-a.x)*1.5; }

    if(a.y>600){ aliens.splice(i,1); score++; }

    if(hit(ship,a)){
      shield--; explode(a.x,a.y); aliens.splice(i,1);
      if(shield<=0){
        highScore=Math.max(highScore,score);
        localStorage.hs=highScore;
        resetGame();
      }
    }
  });

  lasers.forEach((l,i)=>{
    l.y-=7;
    if(l.y<0) lasers.splice(i,1);
    aliens.forEach((a,j)=>{
      if(hitLaser(l,a)){
        explode(a.x,a.y);
        lasers.splice(i,1);
        aliens.splice(j,1);
        score++;
      }
    });
  });

  explosions.forEach((e,i)=>{
    e.r+=2;
    if(e.r>20) explosions.splice(i,1);
  });

  speed=2+score*0.002;
  spawnRate=0.02+score*0.00001;

  if(Date.now()-lastAmmo>300000){ ammo=100; lastAmmo=Date.now(); }
  if(Date.now()-lastShield>300000){ shield=2; lastShield=Date.now(); }
}

function drawRocket(){
  let g=x.createLinearGradient(ship.x,ship.y,ship.x,ship.y+60);
  g.addColorStop(0,"#eee"); g.addColorStop(1,"#999");
  x.fillStyle=g;
  x.fillRect(ship.x+12,ship.y+10,16,40);

  x.fillStyle="#ccc";
  x.beginPath();
  x.moveTo(ship.x+20,ship.y);
  x.lineTo(ship.x+5,ship.y+15);
  x.lineTo(ship.x+35,ship.y+15);
  x.fill();

  x.fillStyle="#b00";
  x.fillRect(ship.x,ship.y+35,10,15);
  x.fillRect(ship.x+30,ship.y+35,10,15);

  x.fillStyle="orange";
  x.beginPath();
  x.moveTo(ship.x+20,ship.y+60);
  x.lineTo(ship.x+15,ship.y+45);
  x.lineTo(ship.x+25,ship.y+45);
  x.fill();
}

function draw(){
  let lvl=levels[Math.floor(score/30)%levels.length];
  c.style.background=lvl.bg;
  x.clearRect(0,0,400,600);

  x.fillStyle=lvl.star;
  stars.forEach(s=>{
    s.y+=1;if(s.y>600)s.y=0;
    x.fillRect(s.x,s.y,s.s,s.s);
  });

  if(shield>0){
    x.strokeStyle="cyan";
    x.beginPath();
    x.arc(ship.x+20,ship.y+30,35,0,Math.PI*2);
    x.stroke();
  }

  drawRocket();

  aliens.forEach(a=>{
    x.fillStyle=a.type==="hunter"?"orange":a.type==="zig"?"lime":"green";
    x.beginPath();
    x.arc(a.x+15,a.y+15,15,0,Math.PI*2);
    x.fill();
  });

  x.fillStyle="red";
  lasers.forEach(l=>x.fillRect(l.x,l.y,4,12));

  explosions.forEach(e=>{
    x.strokeStyle="yellow";
    x.beginPath();
    x.arc(e.x+15,e.y+15,e.r,0,Math.PI*2);
    x.stroke();
  });

  ui.innerHTML=
    "النقاط: "+score+
    "<br>أعلى نتيجة: "+highScore+
    "<br>الدرع: "+shield+
    "<br>الذخيرة: "+ammo;
}

function hit(a,b){
  return a.x<b.x+b.s&&a.x+a.w>b.x&&a.y<b.y+b.s&&a.y+a.h>b.y;
}
function hitLaser(l,a){
  return l.x>a.x&&l.x<a.x+a.s&&l.y>a.y&&l.y<a.y+a.s;
}

function loop(){update();draw();requestAnimationFrame(loop);}
loop();
</script>

</body>
</html>
