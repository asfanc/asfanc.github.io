<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<title>Frikik Oyunu</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{margin:0; font-family:Arial, sans-serif; user-select:none; touch-action:none;}
#menu{
  position:absolute;width:100%;height:100%;
  background: url('https://i.ibb.co/W5zqs2S/stadium.jpg') center/cover no-repeat;
  display:flex;flex-direction:column;justify-content:center;align-items:center;
}
#menu h1{color:white;font-size:48px;text-shadow:0 0 10px black;}
#menu button{
  padding:12px 24px;margin:8px;border:none;border-radius:10px;background:rgba(255,255,255,0.9);
  font-size:22px;font-weight:bold;cursor:pointer;
}
canvas{display:none; background:green;}
#controls{
  position:absolute;bottom:10px;left:50%;transform:translateX(-50%);
  display:none;gap:8px;
}
#controls button{
  padding:12px 18px;font-size:18px;border-radius:10px;border:none;cursor:pointer;
}
</style>
</head>
<body>

<div id="menu">
<h1>Frikik Ustası</h1>
<button onclick="selectPlayer('ronaldo')">Ronaldo</button>
<button onclick="selectPlayer('talisca')">Talisca</button>
<button onclick="selectPlayer('rice')">Declan Rice</button>
<button onclick="selectPlayer('roberto')">Roberto Carlos</button>
<button onclick="startGame()">A Başlat</button>
</div>

<canvas id="game" width="900" height="500"></canvas>
<div id="controls">
  <button onclick="power+=1">+ Güç</button>
  <button onclick="power-=1">- Güç</button>
  <button onclick="curve-=0.2">← Falso</button>
  <button onclick="curve+=0.2">Falso →</button>
  <button onclick="shoot()">⚽ Şut</button>
</div>

<script>
const canvas=document.getElementById("game"),ctx=canvas.getContext("2d");
const menu=document.getElementById("menu"),controls=document.getElementById("controls");

let gameStarted=false;
let player='ronaldo';
let power=20, curve=0;
let ball={x:200,y:250,z:0,vx:0,vy:0,vz:0,moving:false};
let freeKickPos=null;
let wall=[], wallJumping=false, wallJumpStart=0;

// Oyuncular
const players={
  ronaldo:{name:'Ronaldo',num:7,skin:'#ffe0cc',hair:'#222',form:'#fff'},
  talisca:{name:'Talisca',num:94,skin:'#2b2b2b',hair:'#fff',form:'#fff'},
  rice:{name:'Declan Rice',num:41,skin:'#ffe8d6',hair:'#2e2e2e',form:'#fff'},
  roberto:{name:'Roberto Carlos',num:3,skin:'#ffe0cc',hair:null,form:'#fff'}
};

// Baraj renk
const wallColor='#f1c40f', wallAltColor='#e74c3c';

// Kaleci
let keeper={x:450,y:80,w:40,h:60,targetX:450,diving:false,diveTime:0};

// Seçim
function selectPlayer(p){player=p;console.log("Seçilen:",p);}
function startGame(){menu.style.display='none';canvas.style.display='block';controls.style.display='flex';gameStarted=true;draw();}
canvas.addEventListener('click',e=>{
  const r=canvas.getBoundingClientRect();
  freeKickPos={x:e.clientX-r.left,y:e.clientY-r.top};
  computeWall();
});
function clamp(a,b,c){return Math.max(b,Math.min(c,a));}
function computeWall(){
  wall=[];
  if(!freeKickPos) return;
  const goal={x:450,y:40};
  const dx=goal.x-freeKickPos.x,dy=goal.y-freeKickPos.y;
  const dist=Math.hypot(dx,dy);
  let count=dist>420?2:dist>300?3:dist>180?4:5;
  for(let i=0;i<count;i++){
    let x=freeKickPos.x+(i-(count-1)/2)*30;
    let y=freeKickPos.y-130;
    wall.push({x,y,h:40,w:20});
  }
  scheduleWallJump();
}
function scheduleWallJump(){wallJumping=true; wallJumpStart=performance.now();}
function drawField(){
  ctx.fillStyle='green'; ctx.fillRect(0,0,canvas.width,canvas.height);
  // orta çizgi
  ctx.strokeStyle='white'; ctx.lineWidth=2;
  ctx.beginPath(); ctx.moveTo(0,canvas.height/2); ctx.lineTo(canvas.width,canvas.height/2); ctx.stroke();
  // kale
  ctx.strokeRect(200,0,500,60);
}
function drawPlayers(){
  // Bizim oyuncu
  if(freeKickPos){
    const p=players[player];
    ctx.fillStyle=p.form;
    ctx.fillRect(freeKickPos.x-14,freeKickPos.y+10,28,24);
    // baş
    ctx.fillStyle=p.skin;
    ctx.beginPath();ctx.arc(freeKickPos.x,freeKickPos.y-10,10,0,Math.PI*2);ctx.fill();
    if(p.hair){ctx.fillStyle=p.hair;ctx.beginPath();ctx.ellipse(freeKickPos.x,freeKickPos.y-14,8,5,0,0,Math.PI*2);ctx.fill();}
    // Numara/isim
    ctx.fillStyle='black'; ctx.font='bold 12px Arial'; ctx.textAlign='center';
    ctx.fillText(p.num,freeKickPos.x,freeKickPos.y-22);
    ctx.font='10px Arial'; ctx.fillText(p.name,freeKickPos.x,freeKickPos.y+30);
  }
  // Roberto Carlos örnek sabit frikik noktası
  const rc=players['roberto'];
  const rcPos={x:300,y:400};
  ctx.fillStyle=rc.form; ctx.fillRect(rcPos.x-14,rcPos.y+10,28,24);
  ctx.fillStyle=rc.skin; ctx.beginPath();ctx.arc(rcPos.x,rcPos.y-10,10,0,Math.PI*2);ctx.fill();
  if(rc.hair){ctx.fillStyle=rc.hair;ctx.beginPath();ctx.ellipse(rcPos.x,rcPos.y-14,8,5,0,0,Math.PI*2);ctx.fill();}
  ctx.fillStyle='black'; ctx.font='bold 12px Arial'; ctx.textAlign='center';
  ctx.fillText(rc.num,rcPos.x,rcPos.y-22); ctx.font='10px Arial'; ctx.fillText(rc.name,rcPos.x,rcPos.y+30);
}
function drawWall(){
  wall.forEach(wp=>{
    let yshift=0;
    if(wallJumping){
      const jt=clamp((performance.now()-wallJumpStart)/500,0,1);
      yshift=Math.sin(jt*Math.PI)*-30;
    }
    ctx.fillStyle=wallColor; ctx.fillRect(wp.x-10,wp.y+yshift,wp.w,wp.h);
    ctx.fillStyle=wallAltColor; ctx.font='10px Arial'; ctx.textAlign='center';
    ctx.fillText('BARAJ',wp.x,wp.y+yshift-5);
  });
}
function drawBall(){
  if(ball.moving){
    ball.x+=ball.vx; ball.y+=ball.vy; ball.z+=ball.vz;
    ball.vz-=1; if(ball.z<0){ball.z=0;ball.moving=false;}
    ctx.fillStyle='white'; ctx.beginPath();ctx.arc(ball.x,ball.y-ball.z,8,0,Math.PI*2);ctx.fill();
  }
}
function shoot(){
  if(!freeKickPos) return;
  ball={x:freeKickPos.x,y:freeKickPos.y,z:5,vx:2+power/10,vy:-4, vz:power/10, moving:true};
}
function loop(){
  ctx.clearRect(0,0,canvas.width,canvas.height);
  drawField(); drawWall(); drawPlayers(); drawBall();
  requestAnimationFrame(loop);
}
loop();
</script>

</body>
</html>
