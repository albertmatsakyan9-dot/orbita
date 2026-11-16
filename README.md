<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Орбита — аркада</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Unbounded:wght@500;800&family=Rubik:wght@400;600&display=swap" rel="stylesheet">
<style>
:root{
  --bg:#eef2fb;--ink:#0b1020;--mute:#5b6580;--panel:#ffffff;
  --me:#0a6cff;--p1:#6a4cff;--p2:#e0307a;--p3:#0aa88f;--p4:#f08a00;--star:#d99a00;--warn:#e5321f;--dot:#b8c2e0;
  box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px);
}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){
  --bg:#0a0f21;--ink:#eef2ff;--mute:#8d98bd;--panel:#161d38;
  --me:#5ee6ff;--p1:#8b74ff;--p2:#ff5c9e;--p3:#2fe0c0;--p4:#ffa631;--star:#ffd23f;--warn:#ff5a45;--dot:#2a3560;color-scheme:dark}}
:root[data-theme="dark"]{--bg:#0a0f21;--ink:#eef2ff;--mute:#8d98bd;--panel:#161d38;--me:#5ee6ff;--p1:#8b74ff;--p2:#ff5c9e;--p3:#2fe0c0;--p4:#ffa631;--star:#ffd23f;--warn:#ff5a45;--dot:#2a3560;color-scheme:dark}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
[hidden]{display:none!important}
html,body{height:100%;height:100dvh;margin:0}
body{background:var(--bg);color:var(--ink);font:400 15px/1.45 Rubik,system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;
  display:flex;justify-content:center;overflow:hidden;touch-action:none;-webkit-user-select:none;user-select:none}
#wrap{position:relative;width:100%;max-width:480px;height:100%}
canvas{display:block;width:100%;height:100%}
#hud{position:absolute;top:calc(env(safe-area-inset-top,0px) + 12px);left:0;right:0;text-align:center;pointer-events:none}
#score{font:800 56px/1 Unbounded,Rubik,system-ui,sans-serif;font-variant-numeric:tabular-nums}
#best,#pw{color:var(--mute);font-size:13px;margin-top:4px}
#pw{color:var(--ink);font-weight:600;min-height:20px}
#pz{position:absolute;top:calc(env(safe-area-inset-top,0px) + 14px);right:14px;width:44px;height:44px;padding:0;border-radius:50%;background:var(--panel);color:var(--ink);font-size:14px;box-shadow:0 2px 10px rgba(0,0,0,.2)}
#ov{position:absolute;inset:0;display:grid;place-items:center;padding:24px;background:color-mix(in srgb,var(--bg) 80%,transparent);overflow:auto}
#ov.off{display:none}
.box{background:var(--panel);border-radius:20px;padding:26px 22px;text-align:center;max-width:340px;width:100%;box-shadow:0 10px 40px rgba(0,0,0,.25)}
.box h1{margin:0 0 8px;font:800 34px/1.1 Unbounded,Rubik,system-ui,sans-serif}
.box p{margin:0 0 14px;color:var(--mute);font-size:14px;text-align:left}
.box p.c{text-align:center}
.st{margin:0 0 14px;font-weight:600}
.st small{display:block;color:var(--mute);font-weight:400;font-size:12px;margin-top:4px}
button{font:600 16px Rubik,system-ui,sans-serif;border:0;border-radius:14px;padding:13px 24px;background:var(--me);color:var(--bg);cursor:pointer}
.box button{display:block;width:100%;margin-top:10px}
button.s{background:transparent;color:var(--ink);box-shadow:inset 0 0 0 2px var(--dot)}
button:disabled{opacity:.4;cursor:default}
button:focus-visible{outline:3px solid var(--ink);outline-offset:3px}
.sk{display:flex;align-items:center;gap:12px;text-align:left}
.sk i{flex:none;width:22px;height:22px;border-radius:50%}
.sk span{flex:1}
.box .sk button{width:auto;margin:0;padding:9px 14px;font-size:14px}
</style>
</head>
<body>
<div id="wrap">
  <canvas id="cv" aria-label="Игровое поле"></canvas>
  <div id="hud"><div id="score">0</div><div id="best"></div><div id="pw"></div></div>
  <button id="pz" aria-label="Пауза" hidden>❚❚</button>
  <div id="ov"></div>
</div>
<script>
const W=360,cv=document.getElementById('cv'),ctx=cv.getContext('2d'),$=s=>document.getElementById(s);
const SK=[{n:'Кобальт',c:null,p:0},{n:'Роза',c:'#ff5c9e',p:15},{n:'Лайм',c:'#9be22f',p:30},{n:'Золото',c:'#ffc931',p:50},{n:'Призрак',c:'ink',p:80}];
let S={best:0,coins:0,skin:0,own:[0],snd:1,vib:1,games:0,total:0};
try{Object.assign(S,JSON.parse(localStorage.getItem('orbita-save')||'{}'));S.best=Math.max(S.best,+localStorage.getItem('orbita-best')||0)}catch(e){}
const save=()=>{try{localStorage.setItem('orbita-save',JSON.stringify(S))}catch(e){}};
let sc=1,H=640,C={},bg=[],planets,stars,haz,pl,camY,score,caps,gen,state='menu',parts=[],trail=[],pops=[],shake=0,last=0,T=0,run,wipe=0,AC;
const rnd=(a,b)=>a+Math.random()*(b-a),cl=(v,a,b)=>Math.max(a,Math.min(b,v));
function beep(f,d=.1,ty='sine',v=.06,f2){
  if(!S.snd)return;
  try{AC=AC||new(window.AudioContext||window.webkitAudioContext)();if(AC.state==='suspended')AC.resume();const t=AC.currentTime,o=AC.createOscillator(),g=AC.createGain();
    o.type=ty;o.frequency.setValueAtTime(f,t);if(f2)o.frequency.exponentialRampToValueAtTime(f2,t+d);
    g.gain.setValueAtTime(v,t);g.gain.exponentialRampToValueAtTime(.0001,t+d);o.connect(g);g.connect(AC.destination);o.start();o.stop(t+d)}catch(e){}
}
const vib=n=>{if(S.vib&&navigator.vibrate)try{navigator.vibrate(n)}catch(e){}};
function colors(){const s=getComputedStyle(document.documentElement);['bg','ink','mute','me','p1','p2','p3','p4','star','warn','dot'].forEach(k=>C[k]=s.getPropertyValue('--'+k).trim())}
const myC=()=>{const c=SK[S.skin].c;return c==null?C.me:c==='ink'?C.ink:c};
function resize(){const d=Math.min(devicePixelRatio||1,2);cv.width=cv.clientWidth*d;cv.height=cv.clientHeight*d;sc=cv.width/W;H=cv.height/sc;colors();if(planets&&pl&&(state==='play'||state==='pause'))camY=pl.y-H*.62}
addEventListener('resize',resize);
matchMedia('(prefers-color-scheme: dark)').addEventListener('change',colors);
for(let i=0;i<70;i++)bg.push({x:rnd(0,W),y:rnd(0,1200),r:rnd(.6,1.8)});

function addPlanet(){
  const l=planets[planets.length-1],n=gen++,r=rnd(14,24),x=cl(l.bx+rnd(-170,170),60,W-60),y=l.y-rnd(150,210);
  const q={x,bx:x,y,r,R:r+30,col:'p'+(1+n%4),timer:0,max:0,amp:0,sp:rnd(.8,1.4),ph:rnd(0,6.28),n};
  if(n>=9&&Math.random()<.4){q.bx=cl(x,100,W-100);q.amp=rnd(22,48)}
  planets.push(q);
  const t=rnd(.35,.65),k=Math.random(),mx=l.bx+(x-l.bx)*t+rnd(-25,25),my=l.y+(y-l.y)*t;
  if(k<.08)stars.push({t:'h',x:mx,y:my,on:true});
  else if(k<.17)stars.push({t:'c',x:mx,y:my,on:true});
  else if(k<.7)stars.push({t:'s',x:mx,y:my,on:true});
  if(n>=12&&Math.random()<.45)haz.push({x:cl(mx+(Math.random()<.5?-1:1)*rnd(48,75),25,W-25),y:l.y+(y-l.y)*rnd(.3,.7),r:13});
}
function reset(){
  gen=1;planets=[{x:W/2,bx:W/2,y:0,r:20,R:50,col:'p1',timer:6,max:6,amp:0,n:0}];stars=[];haz=[];
  while(planets[planets.length-1].y>-1400)addPlanet();
  pl={mode:'orbit',p:planets[0],a:Math.PI/2,dir:1,x:0,y:0,vx:0,vy:0,cool:0,fly:0,shield:false,freeze:0};
  score=0;caps=0;parts=[];trail=[];pops=[];shake=0;run={stars:0};camY=-H*.62;hud();
}
function hud(){$('score').textContent=score;$('best').textContent='рекорд '+S.best}
function pw(){
  if(!pl)return;
  const t='★ '+run.stars+(pl.shield?'   щит':'')+(pl.freeze>0?'   заморозка '+Math.ceil(pl.freeze):'');
  if($('pw').textContent!==t)$('pw').textContent=t;
}
const pop=(t,x,y)=>pops.push({t,x,y,l:1.2});
function setS(s){state=s;$('pz').hidden=s!=='play';$('hud').hidden=s==='menu'}
function launch(){
  if(state!=='play'||pl.mode!=='orbit')return;
  const sp=300+Math.min(80,caps*2),tx=-Math.sin(pl.a)*pl.dir,ty=Math.cos(pl.a)*pl.dir;
  pl.mode='fly';pl.vx=tx*sp;pl.vy=ty*sp;pl.cool=.35;pl.fly=0;pl.from=pl.p;beep(380,.09,'triangle',.05,620);
}
function die(why){
  if(state!=='play')return;
  if(pl.shield){
    pl.shield=false;pop('Щит спас!',pl.x,pl.y-20);beep(660,.25,'triangle',.07,990);vib(30);
    const q=why==='boom'?pl.p:(pl.from&&planets.includes(pl.from)?pl.from:pl.p);
    pl.mode='orbit';pl.p=q;pl.a=-Math.PI/2;q.max=Math.max(q.max||0,3.5);q.timer=q.max;trail=[];return;
  }
  setS('dead');shake=12;beep(220,.5,'sawtooth',.08,50);vib([80,40,120]);
  for(let i=0;i<28;i++){const a=rnd(0,6.28),s=rnd(40,200);parts.push({x:pl.x,y:pl.y,vx:Math.cos(a)*s,vy:Math.sin(a)*s,l:1})}
  const rec=score>S.best;if(rec)S.best=score;S.games++;S.total+=caps;save();hud();
  setTimeout(()=>{if(state==='dead')show(`<h1>${score}</h1><p class="c">${rec?'Новый рекорд!':'Рекорд '+S.best}<br>Звёзд за забег: ${run.stars}<br>Планет: ${caps}</p>`+btn('play','Ещё раз')+btn('menu','В меню','s'))},650);
}
function update(dt){
  for(const q of planets)if(q.amp)q.x=q.bx+Math.sin(T*q.sp+q.ph)*q.amp;
  if(state==='play'){
    const fz=pl.freeze>0;if(fz)pl.freeze-=dt;
    if(pl.mode==='orbit'){
      const q=pl.p;pl.a+=pl.dir*(2.3+Math.min(1.3,caps*.03))*dt;
      pl.x=q.x+Math.cos(pl.a)*q.R;pl.y=q.y+Math.sin(pl.a)*q.R;
      if(!fz)q.timer-=dt;if(q.timer<=0)die('boom');
    }else{
      pl.x+=pl.vx*dt;pl.y+=pl.vy*dt;pl.cool-=dt;pl.fly+=dt;
      for(const q of planets){
        if(q===pl.from&&pl.cool>0)continue;
        if(Math.hypot(pl.x-q.x,pl.y-q.y)<q.R+8){
          const rx=pl.x-q.x,ry=pl.y-q.y;
          pl.dir=(rx*pl.vy-ry*pl.vx)>=0?1:-1;pl.a=Math.atan2(ry,rx);pl.mode='orbit';pl.p=q;
          if(!q.seen){q.seen=true;q.fresh=true;caps++;score++;hud();beep(520+Math.min(caps,20)*18,.12,'sine',.07);vib(15);
            if(q.n===8)banner('Дальше планеты двигаются');if(q.n===11)banner('Берегись чёрных дыр')}
          if(q.fresh){q.fresh=false;q.max=Math.max(2.1,5.2-caps*.07);q.timer=q.max}break;
        }
      }
      if(pl.mode==='fly'){
        for(const h of haz)if(Math.hypot(pl.x-h.x,pl.y-h.y)<h.r+4){die('hole');break}
        if(pl.mode==='fly'&&(pl.y>camY+H+30||pl.x<-40||pl.x>W+40||pl.fly>3))die('out');
      }
    }
    for(const s of stars)if(s.on&&Math.hypot(pl.x-s.x,pl.y-s.y)<16){
      s.on=false;
      if(s.t==='s'){score+=2;S.coins++;run.stars++;save();hud();pop('+2',s.x,s.y);beep(880,.1,'square',.04,1175)}
      else if(s.t==='h'){pl.shield=true;pop('Щит',s.x,s.y);beep(500,.2,'triangle',.07,900)}
      else{pl.freeze=4;pop('Заморозка',s.x,s.y);beep(700,.2,'triangle',.07,420)}
    }
    trail.push({x:pl.x,y:pl.y});if(trail.length>22)trail.shift();
    const ty=pl.y-H*.62;if(ty<camY)camY+=(ty-camY)*Math.min(1,dt*5);
    while(planets[planets.length-1].y>camY-500)addPlanet();
    while(planets.length>3&&planets[0].y>camY+H+120&&planets[0]!==pl.p)planets.shift();
    stars=stars.filter(s=>s.y<camY+H+100);haz=haz.filter(h=>h.y<camY+H+100);
    pw();
  }
  parts.forEach(q=>{q.x+=q.vx*dt;q.y+=q.vy*dt;q.l-=dt*1.4});parts=parts.filter(q=>q.l>0);
  pops.forEach(p=>{p.l-=dt;if(p.y!=null)p.y-=dt*30});pops=pops.filter(p=>p.l>0);
  shake*=.9;
}
const banner=t=>pops.push({t,x:null,y:null,l:2.6});
function planet(q,pu,low){
  const r=q.r*pu,x=q.x,y=q.y,k=q.n%4,PI=Math.PI;
  if(k===3){ctx.strokeStyle='rgba(255,210,140,.7)';ctx.lineWidth=r*.22;ctx.beginPath();ctx.ellipse(x,y,r*1.9,r*.55,-.3,PI,2*PI);ctx.stroke()}
  ctx.save();ctx.beginPath();ctx.arc(x,y,r,0,6.3);ctx.clip();
  ctx.fillStyle=k===2?'#1f64d0':C[q.col];ctx.fillRect(x-r,y-r,r*2,r*2);
  if(k===0||k===3){for(let i=-3;i<=3;i++){ctx.fillStyle=i%2?'rgba(255,255,255,.2)':'rgba(0,0,0,.16)';ctx.fillRect(x-r,y+i*r*.3-r*.07,r*2,r*.14)}}
  else if(k===1){[[-.4,-.1,.28],[.3,.25,.2],[.1,-.5,.16],[-.15,.5,.18],[.55,-.15,.12]].forEach(c=>{ctx.fillStyle='rgba(0,0,0,.25)';ctx.beginPath();ctx.arc(x+c[0]*r,y+c[1]*r,c[2]*r,0,6.3);ctx.fill();ctx.strokeStyle='rgba(255,255,255,.28)';ctx.lineWidth=1;ctx.stroke()})}
  else{ctx.fillStyle='#4bbf62';[[-.3,-.2,.45,.3],[.35,.3,.35,.25],[0,.6,.3,.15]].forEach(c=>{ctx.beginPath();ctx.ellipse(x+c[0]*r,y+c[1]*r,c[2]*r,c[3]*r,.5,0,6.3);ctx.fill()});ctx.fillStyle='rgba(255,255,255,.4)';ctx.fillRect(x-r,y-r,r*2,r*.2)}
  if(low){ctx.fillStyle='rgba(229,50,31,.5)';ctx.fillRect(x-r,y-r,r*2,r*2)}
  const g=ctx.createRadialGradient(x-r*.4,y-r*.4,r*.1,x,y,r*1.05);g.addColorStop(0,'rgba(255,255,255,.3)');g.addColorStop(.55,'rgba(0,0,0,0)');g.addColorStop(1,'rgba(0,0,20,.6)');
  ctx.fillStyle=g;ctx.fillRect(x-r,y-r,r*2,r*2);ctx.restore();
  if(k===3){ctx.strokeStyle='rgba(255,210,140,.95)';ctx.lineWidth=r*.22;ctx.beginPath();ctx.ellipse(x,y,r*1.9,r*.55,-.3,0,PI);ctx.stroke()}
  if(q.amp){ctx.fillStyle=C.mute;for(const s of[-1,1]){ctx.beginPath();ctx.moveTo(x+s*(q.R-10),y);ctx.lineTo(x+s*(q.R-18),y-4);ctx.lineTo(x+s*(q.R-18),y+4);ctx.fill()}}
}
function draw(){
  ctx.setTransform(sc,0,0,sc,0,0);ctx.fillStyle=C.bg;ctx.fillRect(0,0,W,H);ctx.fillStyle=C.dot;
  for(const b of bg){const y=((b.y-camY*.25)%1200+1200)%1200;if(y<H){ctx.beginPath();ctx.arc(b.x,y,b.r,0,6.3);ctx.fill()}}
  const sx=shake?rnd(-shake,shake)/2:0,sy=shake?rnd(-shake,shake)/2:0,me=myC(),live=state==='play'||state==='pause';
  ctx.setTransform(sc,0,0,sc,sx*sc,(-camY+sy)*sc);
  for(const q of planets){
    if(q.y<camY-60||q.y>camY+H+60)continue;
    const cur=live&&pl.p===q&&pl.mode==='orbit',low=cur&&q.timer<1.2&&pl.freeze<=0;
    ctx.lineWidth=1.5;ctx.strokeStyle=C.mute;ctx.globalAlpha=.35;ctx.setLineDash([3,6]);
    ctx.beginPath();ctx.arc(q.x,q.y,q.R,0,6.3);ctx.stroke();ctx.setLineDash([]);ctx.globalAlpha=1;
    if(cur&&q.max){
      ctx.lineWidth=4;ctx.lineCap='round';ctx.strokeStyle=pl.freeze>0?C.p3:low?C.warn:C[q.col];
      ctx.beginPath();ctx.arc(q.x,q.y,q.R+8,-Math.PI/2,-Math.PI/2+6.2832*Math.max(0,q.timer/q.max));ctx.stroke();
    }
    planet(q,low?1+Math.sin(performance.now()/60)*.08:1,low);
  }
  for(const h of haz){
    if(h.y<camY-40||h.y>camY+H+40)continue;
    ctx.fillStyle='#000';ctx.beginPath();ctx.arc(h.x,h.y,h.r,0,6.3);ctx.fill();
    ctx.strokeStyle=C.warn;ctx.lineWidth=3;ctx.lineCap='round';
    for(let i=0;i<3;i++){ctx.beginPath();ctx.arc(h.x,h.y,h.r+5,T*3+i*2.1,T*3+i*2.1+1.2);ctx.stroke()}
  }
  for(const s of stars){
    if(!s.on||s.y<camY-20||s.y>camY+H+20)continue;
    if(s.t==='s'){ctx.fillStyle=C.star;ctx.beginPath();for(let i=0;i<8;i++){const a=i*Math.PI/4,r=i%2?3.5:9;ctx.lineTo(s.x+Math.cos(a)*r,s.y+Math.sin(a)*r)}ctx.closePath();ctx.fill()}
    else{
      ctx.strokeStyle=s.t==='h'?C.me:C.p3;ctx.lineWidth=3;ctx.beginPath();ctx.arc(s.x,s.y,9,0,6.3);ctx.stroke();
      if(s.t==='h'){ctx.fillStyle=C.me;ctx.beginPath();ctx.arc(s.x,s.y,3,0,6.3);ctx.fill()}
      else{ctx.lineWidth=2;ctx.beginPath();ctx.moveTo(s.x,s.y-5);ctx.lineTo(s.x,s.y);ctx.lineTo(s.x+4,s.y+2);ctx.stroke()}
    }
  }
  if(state!=='menu'){
    for(let i=0;i<trail.length;i++){ctx.globalAlpha=i/trail.length*.5;ctx.fillStyle=me;ctx.beginPath();ctx.arc(trail[i].x,trail[i].y,2+i/trail.length*3,0,6.3);ctx.fill()}
    ctx.globalAlpha=1;
  }
  if(live){
    if(pl.mode==='orbit'&&state==='play'){
      const tx=-Math.sin(pl.a)*pl.dir,ty=Math.cos(pl.a)*pl.dir;
      ctx.globalAlpha=.35;ctx.strokeStyle=me;ctx.lineWidth=2;ctx.setLineDash([4,6]);
      ctx.beginPath();ctx.moveTo(pl.x,pl.y);ctx.lineTo(pl.x+tx*90,pl.y+ty*90);ctx.stroke();ctx.setLineDash([]);ctx.globalAlpha=1;
    }
    ctx.shadowColor=me;ctx.shadowBlur=14;ctx.fillStyle=me;ctx.beginPath();ctx.arc(pl.x,pl.y,6,0,6.3);ctx.fill();ctx.shadowBlur=0;
    if(pl.shield){ctx.strokeStyle=me;ctx.lineWidth=2;ctx.beginPath();ctx.arc(pl.x,pl.y,12,0,6.3);ctx.stroke()}
  }
  for(const q of parts){ctx.globalAlpha=Math.max(0,q.l);ctx.fillStyle=me;ctx.beginPath();ctx.arc(q.x,q.y,3,0,6.3);ctx.fill()}
  ctx.globalAlpha=1;ctx.setTransform(sc,0,0,sc,0,0);ctx.textAlign='center';ctx.fillStyle=C.ink;
  for(const p of pops){
    ctx.globalAlpha=Math.min(1,p.l);
    if(p.x==null){ctx.font='700 18px Rubik,sans-serif';ctx.fillText(p.t,W/2,H*.3)}
    else{ctx.font='700 14px Rubik,sans-serif';ctx.fillText(p.t,p.x,p.y-camY)}
  }
  ctx.globalAlpha=1;
}
function loop(t){const dt=Math.min(.033,(t-last)/1000||0);last=t;if(state!=='pause')T+=dt;if(state!=='pause')update(dt);draw();requestAnimationFrame(loop)}

const btn=(a,t,c='',v='')=>`<button class="${c}" data-a="${a}" data-v="${v}">${t}</button>`;
function show(h){$('ov').innerHTML='<div class="box">'+h+'</div>';$('ov').classList.remove('off')}
const hide=()=>$('ov').classList.add('off');
function menu(){
  setS('menu');reset();wipe=0;
  show(`<h1>Орбита</h1><p class="c">Лети от планеты к планете, пока они не рухнули.</p><div class="st">Рекорд ${S.best}<br>Звёзд: ${S.coins}<small>Игр: ${S.games}, планет пройдено: ${S.total}</small></div>`+btn('play','Играть')+btn('shop','Магазин','s')+btn('help','Как играть','s')+btn('set','Настройки','s'));
}
function shop(){
  show(`<h1>Магазин</h1><p class="c">У тебя звёзд: ${S.coins}</p>`+SK.map((k,i)=>{
    const own=S.own.includes(i),col=k.c==null?'var(--me)':k.c==='ink'?'var(--ink)':k.c;
    return `<div class="sk" style="margin-top:10px"><i style="background:${col}"></i><span>${k.n}</span>`+
      (own?`<button class="${S.skin===i?'':'s'}" data-a="pick" data-v="${i}"${S.skin===i?' disabled':''}>${S.skin===i?'Выбран':'Выбрать'}</button>`
          :`<button data-a="buy" data-v="${i}"${S.coins<k.p?' disabled':''}>★ ${k.p}</button>`)+`</div>`}).join('')+btn('menu','Назад','s'));
}
function help(){
  show(`<h1>Как играть</h1><p>Ты летаешь вокруг планеты. Тап (или пробел) — оторваться и лететь по пунктирной линии.<br><br>Поймай следующую планету, пока эта не рухнула: дуга вокруг орбиты показывает время. Промах и вылет за экран — конец.<br><br>★ Звезда: +2 очка и 1 звезда для магазина.<br>Кольцо с точкой: щит, спасает от одной ошибки.<br>Кольцо со стрелкой: заморозка таймера на 4 секунды.<br><br>С 10-й планеты они двигаются, с 13-й появляются чёрные дыры.</p>`+btn('menu','Понятно'));
}
function settings(){
  show(`<h1>Настройки</h1>`+btn('snd','Звук: '+(S.snd?'вкл':'выкл'),'s')+btn('vib','Вибрация: '+(S.vib?'вкл':'выкл'),'s')+btn('wipe',wipe?'Точно сбросить всё?':'Сбросить прогресс','s')+btn('menu','Назад'));
}
function pause(){if(state!=='play')return;setS('pause');show(`<h1>Пауза</h1><p class="c">Счёт: ${score}</p>`+btn('resume','Продолжить')+btn('menu','В меню','s'))}
function start(){reset();setS('play');hide();beep(520,.08);if(!S.games)banner('Тап — оторваться от планеты')}
function act(a,v){
  if(a==='play')start();
  else if(a==='menu')menu();
  else if(a==='shop')shop();
  else if(a==='help')help();
  else if(a==='set')settings();
  else if(a==='resume'){setS('play');hide()}
  else if(a==='pick'){S.skin=+v;save();shop()}
  else if(a==='buy'){const k=SK[+v];if(S.coins>=k.p){S.coins-=k.p;S.own.push(+v);S.skin=+v;save()}shop()}
  else if(a==='snd'){S.snd=S.snd?0:1;save();settings();beep(600,.08)}
  else if(a==='vib'){S.vib=S.vib?0:1;save();settings();vib(40)}
  else if(a==='wipe'){
    if(!wipe){wipe=1;settings()}
    else{S={best:0,coins:0,skin:0,own:[0],snd:S.snd,vib:S.vib,games:0,total:0};save();menu()}
  }
}
$('ov').addEventListener('click',e=>{const b=e.target.closest('[data-a]');if(b&&!b.disabled)act(b.dataset.a,b.dataset.v)});
$('pz').addEventListener('click',e=>{e.currentTarget.blur();pause()});
cv.addEventListener('pointerdown',e=>{e.preventDefault();launch()});
addEventListener('keydown',e=>{
  if(e.code==='Space'||e.code==='Enter'){if(state==='play'){e.preventDefault();launch()}}
  else if(e.code==='KeyP'||e.code==='Escape'){if(state==='play')pause();else if(state==='pause')act('resume')}
});
document.addEventListener('visibilitychange',()=>{if(document.hidden)pause()});
resize();menu();requestAnimationFrame(loop);
</script>
</body>
</html>
