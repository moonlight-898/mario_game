[Mario – jednoduchá plošinovka.html](https://github.com/user-attachments/files/33241152/Mario.jednoducha.plosinovka.html)
<!DOCTYPE html>
<html lang="cs">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Skokan</title>
<style>
:root{--bg:#1d2b53;--ink:#fff7e6;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#1d2b53;--ink:#fff7e6}}
:root[data-theme="dark"]{--bg:#1d2b53;--ink:#fff7e6}
html,body{min-height:100%;margin:0}
body{background:var(--bg);color:var(--ink);font-family:"Trebuchet MS",Verdana,sans-serif;display:flex;flex-direction:column;align-items:center;gap:14px;padding:16px 12px}
canvas{display:block;width:100%;height:auto;image-rendering:pixelated;border:4px solid #fff7e6;border-radius:6px;background:#7ec8f0}
h1{margin:0;font-size:clamp(34px,8vw,52px);letter-spacing:6px;color:#e63946;text-shadow:3px 3px 0 #1d4ed8,6px 6px 0 rgba(0,0,0,.35)}
.wrap{display:flex;flex-wrap:wrap;gap:16px;justify-content:center;align-items:flex-start;width:100%;max-width:1100px}
.game{flex:1 1 480px;max-width:900px;display:flex;flex-direction:column;gap:10px;align-items:center}
.rules{flex:0 1 280px;min-width:240px;background:rgba(255,247,230,.08);border:3px solid var(--ink);border-radius:10px;padding:14px 18px}
.rules h2{margin:0 0 8px;font-size:18px;color:#ffd23f}
.rules ul{margin:0;padding-left:20px;font-size:14px;line-height:1.55}
.rules li{margin-bottom:6px}
kbd{background:var(--ink);color:var(--bg);border-radius:4px;padding:1px 6px;font-size:12px}
.pad{display:flex;gap:10px}
.pad button{width:72px;height:56px;font-size:24px;border:3px solid var(--ink);background:#ff6b3d;color:#fff;border-radius:10px;touch-action:none}
.pad button:focus-visible{outline:3px solid #ffd23f}
</style>
</head>
<body>
<h1>MARIO</h1>
<div class="wrap">
<div class="game">
<canvas id="c" width="480" height="270"></canvas>
<div class="pad">
<button id="bl" aria-label="Doleva">◀</button>
<button id="br" aria-label="Doprava">▶</button>
<button id="bj" aria-label="Skok">▲</button>
</div>
</div>
<aside class="rules">
<h2>Pravidla hry</h2>
<ul>
<li><b>Cíl:</b> dojdi až k vlajce na konci úrovně.</li>
<li><b>Pohyb:</b> <kbd>←</kbd> <kbd>→</kbd> nebo <kbd>A</kbd> <kbd>D</kbd>.</li>
<li><b>Skok:</b> <kbd>mezerník</kbd>, <kbd>W</kbd> nebo <kbd>↑</kbd>. Čím déle držíš, tím výš skočíš.</li>
<li><b>Mince:</b> každá ti přidá 1 bod.</li>
<li><b>Nepřátelé:</b> seskoč na ně shora a získáš 2 body. Dotek z boku tě stojí život.</li>
<li><b>Jámy:</b> pád do jámy tě stojí život.</li>
<li><b>Životy:</b> máš jich 5. Po ztrátě života se vrátíš na poslední checkpoint a chvíli jsi nezranitelný (blikáš). Když přijdeš o všechny, je konec hry.</li>
<li><b>Checkpointy:</b> úroveň je dlouhá a má jich 5. Dotkni se šedé vlaječky, zezelená a po ztrátě života se vrátíš k ní.</li>
<li><b>Nová hra:</b> klávesa <kbd>R</kbd> nebo klepnutí na plátno.</li>
<li><b>Mobil:</b> použij tlačítka pod hrou.</li>
</ul>
</aside>
</div>
<script>
const cv=document.getElementById('c'),g=cv.getContext('2d');
const T=16,W=480,H=270;
// Mapa: # zem, B cihla, c mince, e nepřítel, F vlajka
const map=[
"                                                                                                                                                                                                                                                                                                                                               ",
"                                                                                                                                                                                                                                                                                                                                               ",
"                                                                                                                                                                                                                                                                                                                                               ",
"                                                                                                                                                                                                                                                                                                                                               ",
"                                                                                                                                                                                                                                                                                                                                               ",
"                                                                         c                                                                                        c                                c          c               c                                       c                                                                        ",
"          BBB         BBB                                               BBB           BB                                                                         BB                                B         BB              BB                          BBB         BB            B                                                           ",
"                                                                                                                                                                                                                                                                                                                                       F       ",
"            c           c         c                                     c                        c                                           c                   c                                 c         c               c                           c c                       c                        c                 B                ",
"         BBBBB       BBBBB       BBB           BBB         BBBB        BBBBB         BBBB   B   BBBB                            BBB         BBB       BBB       BBBB                    BBB       BBB       BBBB            BBBB                        BBBBB       BBBB       B  BBB                      BBB               BB                ",
"              c               c                                       c               c    BB       c                           c               c    BB     c                               c            c    BB           c                              c              c    BB                                   c        BBB                ",
"      ccc              e               e      e        K         ccc         e            BBB             e        K               cc  e            BBB             e          K                  e         cBBB      e          e         K                    e   e        BBB     e   e    e        K                   BBBB                ",
"##############   #############   ####################################    #############   ##########    ########################    #############   #########   #############################   #########    ##############    ###########################    ############   ######################################    #########################",
"##############   #############   ####################################    #############   ##########    ########################    #############   #########   #############################   #########    ##############    ###########################    ############   ######################################    #########################",
"##############   #############   ####################################    #############   ##########    ########################    #############   #########   #############################   #########    ##############    ###########################    ############   ######################################    #########################",
"##############   #############   ####################################    #############   ##########    ########################    #############   #########   #############################   #########    ##############    ###########################    ############   ######################################    #########################",
"##############   #############   ####################################    #############   ##########    ########################    #############   #########   #############################   #########    ##############    ###########################    ############   ######################################    #########################"
];
const rows=map.length,cols=map[0].length;
let grid,coins,enemies,flag,score,p,keys={},cam,state,lives,inv,safe,cps,msg=0,msgText="";
function reset(){
  grid=[];coins=[];enemies=[];cps=[];msg=0;score=0;cam=0;state='play';lives=5;inv=0;
  for(let y=0;y<rows;y++){grid[y]=[];for(let x=0;x<cols;x++){
    const ch=map[y][x]||' ';
    grid[y][x]=(ch==='#'||ch==='B')?ch:' ';
    if(ch==='c')coins.push({x:x*T+4,y:y*T+4,w:8,h:8});
    if(ch==='e')enemies.push({x:x*T,y:y*T,w:14,h:14,vx:-0.5,dead:false});
    if(ch==='K')cps.push({x:x*T,y:y*T,on:false});
    if(ch==='F')flag={x:x*T,y:y*T};
  }}
  p={x:20,y:100,w:12,h:16,vx:0,vy:0,ground:false,face:1};
  safe={x:20,y:100};
}
const solid=(x,y)=>{const tx=Math.floor(x/T),ty=Math.floor(y/T);
  if(tx<0)return true;if(ty>=rows)return false;if(ty<0||tx>=cols)return false;return grid[ty][tx]!==' '};
function hitsSolid(o){return solid(o.x,o.y)||solid(o.x+o.w-.01,o.y)||solid(o.x,o.y+o.h-.01)||solid(o.x+o.w-.01,o.y+o.h-.01)}
function move(o){
  o.x+=o.vx;if(hitsSolid(o)){o.x-=o.vx;o.hitWall=true}else o.hitWall=false;
  o.y+=o.vy;o.ground=false;
  if(hitsSolid(o)){o.y-=o.vy;if(o.vy>0)o.ground=true;o.vy=0;
    // doraz
    while(hitsSolid(o)&&false){}
  }
}
const ov=(a,b)=>a.x<b.x+b.w&&a.x+a.w>b.x&&a.y<b.y+b.h&&a.y+a.h>b.y;
function die(){
  lives--;
  if(lives<=0){state='lose';return}
  p.x=safe.x;p.y=safe.y;p.vx=0;p.vy=0;inv=120;
}
function update(){
  if(state!=='play')return;
  const l=keys.ArrowLeft||keys.a||keys.bl,r=keys.ArrowRight||keys.d||keys.br,j=keys[' ']||keys.ArrowUp||keys.w||keys.bj;
  p.vx=(r?2:0)-(l?2:0);if(p.vx)p.face=Math.sign(p.vx);
  if(j&&p.ground){p.vy=-6.2;p.ground=false}
  p.vy=Math.min(p.vy+0.3,6);
  if(!j&&p.vy<-2.5)p.vy=-2.5;
  move(p);
  if(inv>0)inv--;
  if(msg>0)msg--;
  for(const c of cps){if(!c.on&&ov(p,{x:c.x,y:c.y-32,w:16,h:48})){c.on=true;if(c.x>safe.x){safe={x:c.x,y:c.y}}msg=120;msgText='Checkpoint '+(cps.indexOf(c)+1)+'/'+cps.length}}
  if(p.y>H+40)die();
  coins=coins.filter(c=>{if(ov(p,c)){score++;return false}return true});
  for(const e of enemies){if(e.dead)continue;
    e.vy=Math.min((e.vy||0)+0.3,6);
    const px=e.x;e.vx=e.vx||-0.5;move(e);
    if(e.hitWall)e.vx*=-1;
    // nepřítel nesmí spadnout z okraje
    if(e.ground&&!solid(e.vx>0?e.x+e.w+1:e.x-1,e.y+e.h+2))e.vx*=-1;
    if(ov(p,e)){
      if(p.vy>0&&p.y+p.h-e.y<10){e.dead=true;p.vy=-4.5;score+=2}
      else if(inv<=0)die();
    }
  }
  if(flag&&p.x+p.w>flag.x)state='win';
  cam=Math.max(0,Math.min(p.x-W/2,cols*T-W));
}
function draw(){
  g.fillStyle='#7ec8f0';g.fillRect(0,0,W,H);
  g.fillStyle='#fff';
  for(let i=0;i<8;i++){const cx=((i*170)-cam*0.3)%(W+120);g.fillRect(cx,30+(i%3)*25,50,12);g.fillRect(cx+10,22+(i%3)*25,30,10)}
  g.save();g.translate(-Math.floor(cam),0);
  const x0=Math.floor(cam/T),x1=Math.min(cols,x0+Math.ceil(W/T)+2);
  for(let y=0;y<rows;y++)for(let x=x0;x<x1;x++){
    const t=grid[y][x];if(t===' ')continue;
    if(t==='#'){g.fillStyle='#8b5a2b';g.fillRect(x*T,y*T,T,T);
      if(grid[y-1]&&grid[y-1][x]===' '){g.fillStyle='#4caf50';g.fillRect(x*T,y*T,T,4)}}
    else{g.fillStyle='#d9742f';g.fillRect(x*T,y*T,T,T);g.fillStyle='#8a3d12';
      g.fillRect(x*T,y*T+7,T,2);g.fillRect(x*T+7,y*T,2,7);g.fillRect(x*T+3,y*T+9,2,7)}
  }
  g.fillStyle='#ffd23f';coins.forEach(c=>{g.beginPath();g.arc(c.x+4,c.y+4,4,0,7);g.fill()});
  enemies.forEach(e=>{if(e.dead)return;g.fillStyle='#7a3b1e';g.fillRect(e.x,e.y+2,e.w,e.h-2);
    g.fillStyle='#fff';g.fillRect(e.x+2,e.y+4,3,3);g.fillRect(e.x+9,e.y+4,3,3);
    g.fillStyle='#000';g.fillRect(e.x+3,e.y+5,1,2);g.fillRect(e.x+10,e.y+5,1,2)});
  cps.forEach(c=>{g.fillStyle='#ddd';g.fillRect(c.x+6,c.y-32,3,48);g.fillStyle=c.on?'#4caf50':'#9aa0a6';g.fillRect(c.x+9,c.y-32,12,8)});
  if(flag){g.fillStyle='#fff';g.fillRect(flag.x+6,flag.y,3,T*5);g.fillStyle='#e63946';g.fillRect(flag.x+9,flag.y,14,10)}
  // hráč
  if(!(inv>0&&Math.floor(inv/6)%2)){
  g.fillStyle='#e63946';g.fillRect(p.x,p.y,p.w,6);
  g.fillStyle='#ffcc99';g.fillRect(p.x+1,p.y+6,p.w-2,4);
  g.fillStyle='#1d4ed8';g.fillRect(p.x,p.y+10,p.w,6);
  g.fillStyle='#000';g.fillRect(p.x+(p.face>0?8:3),p.y+7,2,2);
  }
  g.restore();
  g.fillStyle='#fff7e6';g.font='bold 14px Trebuchet MS';g.fillText('Mince: '+score+'    Životy: '+'♥'.repeat(Math.max(0,lives))+'    CP: '+cps.filter(c=>c.on).length+'/'+cps.length,10,20);
  if(msg>0){g.textAlign='center';g.fillText(msgText,W/2,48);g.textAlign='left'}
  if(state!=='play'){
    g.fillStyle='rgba(0,0,0,.6)';g.fillRect(0,0,W,H);g.fillStyle='#fff7e6';g.textAlign='center';
    g.font='bold 28px Trebuchet MS';
    g.fillText(state==='win'?'Vyhráls! Skóre: '+score:'Game over',W/2,H/2-6);
    g.font='14px Trebuchet MS';g.fillText('Stiskni R nebo klepni pro novou hru',W/2,H/2+20);g.textAlign='left';
  }
}
function loop(){update();draw();requestAnimationFrame(loop)}
addEventListener('keydown',e=>{const k=e.key.length===1?e.key.toLowerCase():e.key;keys[k]=true;
  if(k==='r')reset();if([' ','ArrowUp','ArrowDown'].includes(e.key))e.preventDefault()});
addEventListener('keyup',e=>{keys[e.key.length===1?e.key.toLowerCase():e.key]=false});
[['bl','bl'],['br','br'],['bj','bj']].forEach(([id,k])=>{const b=document.getElementById(id);
  const on=e=>{e.preventDefault();keys[k]=true},off=e=>{e.preventDefault();keys[k]=false};
  b.addEventListener('pointerdown',on);b.addEventListener('pointerup',off);b.addEventListener('pointerleave',off);b.addEventListener('pointercancel',off)});
cv.addEventListener('pointerdown',()=>{if(state!=='play')reset()});
reset();loop();
</script>
</body>
</html>
