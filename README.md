<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Ликорисы</title>
<style>
:root{color-scheme:dark;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
html,body{height:100%;margin:0;background:#000;color:#8a8f94;font-family:"Courier New",monospace}
body{display:flex;flex-direction:column;align-items:center;justify-content:center;gap:10px}
canvas{display:block;cursor:pointer;max-width:100%}
button{background:none;border:1px solid #333;color:#8a8f94;font:inherit;font-size:13px;padding:6px 14px;cursor:pointer;border-radius:2px}
button:hover,button:focus-visible{color:#fff;border-color:#888;outline:none}
</style>
</head>
<body>
<canvas id="c" role="img" aria-label="Букет красных ликорисов, раскрывающихся в ASCII-графике"></canvas>
<button id="r" type="button">распустить заново</button>
<script>
const RAMP=' .`^",:;Il!i><~+_-?][}{1)(|\\/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$';
const TAU=Math.PI*2,RM=matchMedia('(prefers-reduced-motion: reduce)').matches;
const view=document.getElementById('c'),vx=view.getContext('2d');
const off=document.createElement('canvas'),ox=off.getContext('2d',{willReadFrequently:true});
let cols,rows,sw,sh,cw,ch;
function layout(){
  const Wp=Math.floor(Math.min(innerWidth,(innerHeight-60)/1.4)),Hp=Math.floor(Wp*1.4);
  cols=Wp<520?110:170;rows=Math.round(cols*1.4/1.8);
  sw=cols*4;sh=Math.round(sw*1.4);off.width=sw;off.height=sh;
  const dpr=Math.min(devicePixelRatio||1,2);
  view.width=Wp*dpr;view.height=Hp*dpr;view.style.width=Wp+'px';view.style.height=Hp+'px';
  cw=view.width/cols;ch=view.height/rows;
}
const rnd=s=>{const x=Math.sin(s*127.1+3.7)*43758.5453;return x-Math.floor(x);};
const ss=x=>x<=0?0:x>=1?1:x*x*(3-2*x);const sm=x=>x<=0?0:x>=1?1:x*x*x*(x*(x*6-15)+10);
// stalks: top position, floret size, florets count, bloom delay, seed
const ST=[{x:340,y:210,L:56,n:3,d:.6,s:6,a:.45},{x:670,y:190,L:58,n:3,d:1,s:7,a:.45},{x:150,y:400,L:56,n:3,d:1.8,s:8,a:.45},
{x:860,y:360,L:56,n:3,d:2,s:9,a:.45},{x:500,y:520,L:54,n:3,d:2.6,s:10,a:.45},
{x:365,y:410,L:68,n:4,d:1,s:11,a:.75},{x:640,y:390,L:70,n:4,d:1.6,s:12,a:.75},
{x:125,y:700,L:66,n:3,d:3,s:13,a:.75},{x:880,y:690,L:66,n:3,d:3.4,s:14,a:.75},
{x:500,y:300,L:88,n:4,d:0,s:1,a:1},{x:235,y:560,L:80,n:4,d:1.4,s:2,a:1},{x:770,y:520,L:82,n:4,d:2.3,s:3,a:1},
{x:385,y:850,L:62,n:3,d:3.2,s:4,a:1},{x:630,y:810,L:64,n:3,d:3.8,s:5,a:1},{x:510,y:700,L:72,n:4,d:2.9,s:15,a:1}];
ST.sort((p,q)=>p.a-q.a||p.y-q.y);ST.forEach(k=>{k.bo=(k.x-500)/500*40;k.f=[];for(let j=0;j<k.n;j++){
  const a=j/k.n*TAU+k.s,R=k.L*.5,dx=Math.cos(a)*R,dy=Math.sin(a)*R*.7-k.L*.2;
  k.f.push({a,R,dx,dy,ax:dx/R*.7,L:k.L*(.9+.2*rnd(k.s+j)),s:k.s*10+j,d:k.d+j*.5,q:k.a});
}});
function stem(k,sx,sy,g,T){
  const bo=k.bo,p=[[500+bo*1.2,1075],[500+bo*1.6,1075+(sy-1075)*.35],[sx+(500-sx)*.12,Math.max(sy,Math.min(sy+380,930))],[sx,sy]];
  const bz=t=>{const u=1-t;return[u*u*u*p[0][0]+3*u*u*t*p[1][0]+3*u*t*t*p[2][0]+t*t*t*p[3][0],u*u*u*p[0][1]+3*u*u*t*p[1][1]+3*u*t*t*p[2][1]+t*t*t*p[3][1]];};
  const am=60*Math.pow(1-g,1.3),wv=u=>am*Math.sin(u*18-T*3.5)*Math.min(1,u*5);
  ox.beginPath();ox.moveTo(...p[0]);for(let i=1;i<=40;i++){const u=g*i/40,q=bz(u);ox.lineTo(q[0]+wv(u),q[1]);}
  ox.strokeStyle='rgb(95,120,95)';ox.lineWidth=8+6*k.a;ox.stroke();
  ox.strokeStyle='rgb(175,205,175)';ox.lineWidth=3;ox.stroke();const q=bz(g);return[q[0]+wv(g),q[1]];
}
function floret(c,b,T){
  const L=c.L*(.55+.45*b),h0=-Math.PI/2+c.ax+Math.sin(T*.8+c.s)*.04*b;
  for(let i=0;i<6;i++){
    const r1=rnd(c.s*7+i),r2=rnd(c.s*13+i*3);
    let o=((i/6)*TAU+r1*.6-.3)%TAU;if(o>Math.PI)o-=TAU;
    const bp=sm(b*1.3-r1*.3),Lp=c.L*(.5+.5*bp),hs=h0+o*.92*bp+(r1-.5)*.35*(1-bp),th=(r2>.45?1:-1)*(1.3+2.2*r1)*bp*(1+.05*Math.sin(T*.9+r1*6));
    const W=c.L*.19*(.7+.3*r2),N=c.q<.5?10:c.q<.8?14:20,P=[];let x=c.x,y=c.y;
    for(let k=0;k<=N;k++){const t=k/N,h=hs+th*Math.pow(t,1.5);P.push([x,y,t]);x+=Math.cos(h)*Lp/N;y+=Math.sin(h)*Lp/N;}
    const Lf=[],Rt=[];
    for(let k=0;k<=N;k++){
      const a=P[Math.max(0,k-1)],d=P[Math.min(N,k+1)],t=P[k][2];
      let tx=d[0]-a[0],ty=d[1]-a[1];const m=Math.hypot(tx,ty)||1;tx/=m;ty/=m;
      const tw=.45+.55*Math.abs(Math.cos(t*5.5+r1*6));
      const w=W*(.25+.75*Math.sin(Math.PI*Math.min(1,t*1.05)))*tw*(.8+.2*bp)/2;
      Lf.push([P[k][0]-ty*w,P[k][1]+tx*w]);Rt.push([P[k][0]+ty*w,P[k][1]-tx*w]);
    }
    for(let k=0;k<N;k++){
      const u=.5+.5*Math.cos(k/N*5.5+r1*6+1);
      ox.fillStyle=ox.strokeStyle=`rgb(${(110+125*u)|0},${(6+36*u)|0},${(20+32*u)|0})`;
      ox.lineWidth=.8;ox.beginPath();ox.moveTo(...Lf[k]);ox.lineTo(...Lf[k+1]);ox.lineTo(...Rt[k+1]);ox.lineTo(...Rt[k]);
      ox.closePath();ox.fill();
    }
    ox.strokeStyle='rgba(255,175,170,.85)';ox.lineWidth=1.3;ox.beginPath();
    Lf.forEach((p,k)=>k?ox.lineTo(p[0],p[1]):ox.moveTo(p[0],p[1]));ox.stroke();
  }
  if(b>.18){
    const sb=sm((b-.18)/.82);
    const ns=c.q<.5?6:9;for(let j=0;j<ns;j++){
      const r=rnd(c.s*31+j);let o=((j/ns)*TAU+r*.5)%TAU;if(o>Math.PI)o-=TAU;
      const a0=h0+o*.8;let df=-Math.PI/2-a0;df=Math.atan2(Math.sin(df),Math.cos(df));
      const Ls=c.L*(1.5+.6*r)*sb,N=14;let x=c.x,y=c.y,h=a0;
      ox.strokeStyle='rgb(235,95,95)';ox.lineWidth=1.8;ox.beginPath();ox.moveTo(x,y);
      for(let k=1;k<=N;k++){h=a0+df*.85*Math.pow(k/N,1.4)+Math.sin(T*1.7+j*2+k*.35)*.03*(k/N);x+=Math.cos(h)*Ls/N;y+=Math.sin(h)*Ls/N;ox.lineTo(x,y);}
      ox.stroke();
      ox.fillStyle='rgb(255,160,150)';ox.beginPath();ox.ellipse(x,y,4*sb,2.4*sb,h,0,TAU);ox.fill();
    }
  }
}
function vasePath(){
  ox.beginPath();ox.moveTo(412,1020);
  ox.bezierCurveTo(424,1075,440,1105,440,1140);
  ox.bezierCurveTo(440,1190,338,1205,338,1280);
  ox.bezierCurveTo(338,1345,398,1365,408,1380);
  ox.lineTo(592,1380);
  ox.bezierCurveTo(602,1365,662,1345,662,1280);
  ox.bezierCurveTo(662,1205,560,1190,560,1140);
  ox.bezierCurveTo(560,1105,576,1075,588,1020);
  ox.ellipse(500,1020,88,20,0,0,Math.PI);ox.closePath();
}
function vaseBack(){
  ox.fillStyle='rgba(150,155,165,.35)';ox.beginPath();ox.ellipse(500,1386,140,7,0,0,TAU);ox.fill();
  ox.fillStyle='rgb(34,36,44)';ox.beginPath();ox.ellipse(500,1020,88,20,0,0,TAU);ox.fill();
  ox.strokeStyle='rgb(150,160,175)';ox.lineWidth=5;ox.stroke();
}
function vaseFront(){
  const g=ox.createLinearGradient(338,0,662,0);
  g.addColorStop(0,'rgb(45,50,62)');g.addColorStop(.28,'rgb(205,218,232)');g.addColorStop(.5,'rgb(140,152,170)');g.addColorStop(1,'rgb(26,29,38)');
  vasePath();ox.fillStyle=g;ox.fill();
  ox.save();vasePath();ox.clip();
  for(let r=0;r<8;r++)for(let c=-1;c<8;c++){
    const cx=338+c*56+(r%2?28:0),cy=1210+r*22;
    ox.beginPath();ox.arc(cx,cy,28,Math.PI,TAU);ox.closePath();ox.fillStyle=g;ox.fill();
    ox.strokeStyle='rgba(225,232,242,.7)';ox.lineWidth=2;
    for(let k=0;k<3;k++){ox.beginPath();ox.arc(cx,cy,28-k*8,Math.PI,TAU);ox.stroke();}
  }
  ox.restore();
  ox.strokeStyle='rgba(255,255,255,.45)';ox.lineWidth=6;ox.beginPath();ox.moveTo(412,1150);
  ox.bezierCurveTo(392,1180,368,1205,362,1260);ox.bezierCurveTo(360,1300,372,1335,384,1352);ox.stroke();
  ox.strokeStyle='rgb(175,185,200)';ox.lineWidth=3;vasePath();ox.stroke();
  ox.strokeStyle='rgb(205,70,80)';ox.lineWidth=6;ox.beginPath();ox.ellipse(500,1120,58,9,0,0,Math.PI);ox.stroke();
  ox.strokeStyle='rgb(225,232,242)';ox.lineWidth=2;ox.beginPath();ox.ellipse(500,1135,58,9,0,0,Math.PI);ox.stroke();
  ox.strokeStyle='rgb(215,225,238)';ox.lineWidth=6;ox.beginPath();ox.ellipse(500,1020,88,20,0,0,Math.PI);ox.stroke();
}
function drift(T){
  const a=sm((T-8)/4);
  ox.save();
  for(let i=0;i<36;i++){
    const r=rnd(i*5+1),r2=rnd(i*5+2),p=(T*(.018+.02*r)+r2)%1;
    ox.globalAlpha=Math.sin(p*Math.PI)*(.25+.35*rnd(i));ox.fillStyle='rgb(220,220,225)';
    ox.beginPath();ox.arc(60+r*880+Math.sin(T*.5+i)*18,1300-p*1250,2.6,0,TAU);ox.fill();
  }
  for(let i=0;i<14;i++){
    const r=rnd(i*3+40),r2=rnd(i*3+41),r3=rnd(i*3+42),p=(T*(.03+.02*r)+r2)%1,sp=T*(1.1+r)+i;
    ox.globalAlpha=a*.85*Math.min(1,p*8,(1-p)*8);
    ox.save();ox.translate(70+r3*860+Math.sin(T*.7+i*2)*45+p*60,-30+p*1380);
    ox.rotate(Math.sin(sp*.6)*1.2+r2*2);ox.scale(Math.cos(sp),1);
    ox.fillStyle='rgb(215,45,58)';ox.beginPath();ox.moveTo(0,-20);ox.quadraticCurveTo(12,0,0,20);ox.quadraticCurveTo(-5,0,0,-20);ox.fill();
    ox.restore();
  }
  ox.restore();
}
function scene(T){
  ox.fillStyle='#000';ox.fillRect(0,0,sw,sh);
  ox.save();ox.scale(sw/1000,sw/1000);ox.lineCap='round';ox.lineJoin='round';
  const G=ST.map(k=>sm((T-k.d*.4)/3));
  const X=ST.map(k=>k.x+Math.sin(T*.8+k.s*2)*6);
  const Y=ST.map(k=>k.y);vaseBack();
  
  
  
  ST.forEach((k,i)=>{
    ox.globalAlpha=k.a;const tp=stem(k,X[i],Y[i],G[i],T),sp=sm((G[i]-.4)/.6);const fs=k.f.map(f=>{const an=f.a+(1-sp)*(3.5+1.2*T),rs=.25+.75*sp;return{x:tp[0]+Math.cos(an)*f.R*rs,y:tp[1]+(Math.sin(an)*f.R*.7-k.L*.2)*rs,ax:Math.cos(an)*.7,L:f.L*(.45+.55*sm(G[i]*1.15)),s:f.s,q:f.q,b:sm((T-f.d*.4-2.9)/5.5)};});
    fs.forEach(f=>{ox.strokeStyle='rgb(125,50,58)';ox.lineWidth=5;ox.beginPath();ox.moveTo(tp[0],tp[1]);ox.lineTo(f.x,f.y);ox.stroke();});
    fs.sort((a,c)=>a.y-c.y).forEach(f=>floret(f,f.b,T));
  });
  ox.globalAlpha=1;vaseFront();drift(T);
  
  ox.restore();
}
const cache=new Map();
function col(r,g,b){
  const key=((r>>3)<<10)|((g>>3)<<5)|(b>>3);let s=cache.get(key);
  if(!s){s=`rgb(${r},${g},${b})`;cache.set(key,s);}return s;
}
function render(T){
  scene(T);const ccx=sw/2;
  const d=ox.getImageData(0,0,sw,sh).data;
  vx.fillStyle='#000';vx.fillRect(0,0,view.width,view.height);
  vx.font=Math.min(ch,cw*1.75)+'px "Courier New",monospace';vx.textAlign='center';vx.textBaseline='middle';
  const bw=sw/cols,bh=sh/rows,tk=Math.floor(T*4),n=RAMP.length-1,K=.34;let last='';
  for(let cy=0;cy<rows;cy++){
    const y0=Math.floor(cy*bh),y1=Math.floor((cy+1)*bh);
    for(let cx=0;cx<cols;cx++){
      const tg=0,px=(cx+.5)*bw-ccx,py=(cy+.5)*bh,zm=1,cs=1,sn=0;
      const ww=Math.ceil(bw/zm),hh=Math.ceil(bh/zm),sxp=Math.floor(ccx+px*cs/zm-ww/2),syp=Math.floor(py+px*sn/zm-hh/2);
      const x0=Math.max(0,sxp),x1=Math.min(sw,sxp+ww),yA=Math.max(0,syp),yB=Math.min(sh,syp+hh),st=Math.max(1,Math.round(1/zm));
      let r=0,g=0,b=0,c=0;
      for(let y=yA;y<yB;y+=2*st){let o=(y*sw+x0)*4;for(let x=x0;x<x1;x+=st,o+=4*st){r+=d[o];g+=d[o+1];b+=d[o+2];c++;}}
      c=c||1;r/=c;g/=c;b/=c;
      const m=Math.max(r,g,b),rv=1-.75*tg;
      if(m<10){if(tg>.05&&tg<.85&&Math.random()<.25*Math.sin(tg*Math.PI)){const s0='rgb(120,125,135)';if(last!==s0){vx.fillStyle=s0;last=s0;}vx.fillText(RAMP[1+(Math.random()*n|0)],(cx+.5)*cw,(cy+.5)*ch);}continue;}
      const h=Math.sin(cx*13.7+cy*91.3+tk*7.1)*43758.5453;
      const v=Math.min(1,Math.pow(m/255,.6)*1.35)+((h-Math.floor(h))-.5)*.08;
      let idx=Math.floor(Math.max(0,Math.min(1,v))*n);if(Math.random()<.6*Math.sin(tg*Math.PI))idx=1+(Math.random()*n|0);if(idx<1)continue;
      const gr=70+160*v,t=(60+170*v)/m,q=1-K;
      const s=col(Math.min(255,gr*q+r*t*K)|0,Math.min(255,gr*q+g*t*K)|0,Math.min(255,gr*q+b*t*K)|0);
      if(s!==last){vx.fillStyle=s;last=s;}
      vx.fillText(RAMP[idx],(cx+.5)*cw,(cy+.5)*ch);
    }
  }
}
let t0=performance.now(),lt=0;
function frame(now){requestAnimationFrame(frame);if(now-lt<42)return;lt=now;render(RM?20:(now-t0)/1000);}
function replay(){t0=performance.now();}
document.getElementById('r').onclick=replay;view.onclick=replay;
addEventListener('resize',layout);
layout();requestAnimationFrame(frame);
</script>
</body>
</html>
