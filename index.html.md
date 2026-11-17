<!DOCTYPE html>  
<html lang="en">  
<head>  
<meta charset="UTF-8"><title>AI Survival 3D</title>  
<style>  
body{margin:0;overflow:hidden;font-family:system-ui,sans-serif;user-select:none;background:#000}  
#ui,#chat,#net{position:absolute;color:#fff;background:rgba(15,20,25,.78);backdrop-filter:blur(8px);padding:12px;border-radius:10px;border:1px solid rgba(255,255,255,.1)}  
#ui{top:15px;left:15px;pointer-events:none;font-size:12px}  
.bar{width:170px;height:7px;background:rgba(255,255,255,.15);border-radius:4px;margin:4px 0 10px;overflow:hidden}  
#hp{width:100%;height:100%;background:#e74c3c}#hg{width:100%;height:100%;background:#f39c12}  
#chat{bottom:15px;left:15px;width:350px}#msgs{height:110px;overflow-y:auto;font-size:12px;margin-bottom:6px}  
#in{width:94%;padding:6px;border:none;border-radius:5px;background:rgba(255,255,255,.15);color:#fff;outline:none}  
#net{top:15px;right:15px;font-size:12px}#net input{width:110px;padding:5px;border-radius:5px;border:none;background:rgba(255,255,255,.15);color:#fff}  
#net button{padding:5px 10px;cursor:pointer;background:#27ae60;border:none;color:#fff;border-radius:5px}  
#aim{position:absolute;top:50%;left:50%;width:6px;height:6px;background:#fff;border-radius:50%;transform:translate(-50%,-50%);pointer-events:none}  
#tip{position:absolute;top:60%;left:50%;transform:translateX(-50%);background:rgba(0,0,0,.75);color:#fff;padding:5px 12px;border-radius:15px;font-size:12px;display:none}  
</style>  
</head>  
<body>  
<div id="aim"></div><div id="tip"></div>  
<div id="ui">  
<div id="adm-tag" style="display:none;color:#f1c40f;font-weight:bold;margin-bottom:4px">★ ADMIN MODE ACTIVE</div>  
<b>HEALTH: <span id="hpt">100%</span></b><div class="bar"><div id="hp"></div></div>  
<b>HUNGER: <span id="hgt">100%</span></b><div class="bar"><div id="hg"></div></div>  
WASD: Move/Drive | Mouse: Look | Click: Shoot<br>F: Car | E: Eat | Enter: Chat (/admin 1234)  
</div>  
<div id="net">  
ID: <b id="mid" style="color:#3498db">...</b><br><br>  
<input id="jid" placeholder="Friend's ID"><button onclick="join()">Join</button>  
<div id="st" style="margin-top:5px;color:#2ecc71">● Online (Host)</div>  
</div>  
<div id="chat"><div id="msgs"><i>Proximity Chat (35m range)...</i></div><input id="in" placeholder="Press Enter to speak or run /commands..."></div>  
<script src="https://cloudflare.com"></script>  
<script src="https://unpkg.com"></script>  
<script>  
const scene=new THREE.Scene();scene.background=new THREE.Color(0x9ec5e5);scene.fog=new THREE.FogExp2(0x9ec5e5,.0032);  
const cam=new THREE.PerspectiveCamera(65,innerWidth/innerHeight,.1,1000);  
const ren=new THREE.WebGLRenderer({antialias:true});ren.setSize(innerWidth,innerHeight);ren.shadowMap.enabled=true;ren.toneMapping=THREE.ACESFilmicToneMapping;document.body.appendChild(ren.domElement);  
scene.add(new THREE.HemisphereLight(0xdbeeff,0x3b4d36,.65));  
const sun=new THREE.DirectionalLight(0xfff5e4,1.3);sun.position.set(120,220,100);sun.castShadow=true;sun.shadow.mapSize.set(2048,2048);  
const d=130;Object.assign(sun.shadow.camera,{left:-d,right:d,top:d,bottom:-d});scene.add(sun,sun.target);  
  
const hAt=(x,z)=>Math.sin(x*.015)*Math.cos(z*.015)*6.5+Math.sin(x*.04+z*.03)*1.8;  
const M=(c,r=.7,m=0)=>new THREE.MeshStandardMaterial({color:c,roughness:r,metalness:m});  
  
function makeChar(col=0x2c3e50){  
 const g=new THREE.Group(),sk=M(0xe0ac69,.65),jk=M(col,.75),pt=M(0x262b30,.85),gm=M(0x1c1f24,.25,.8);  
 const torso=new THREE.Mesh(new THREE.CylinderGeometry(.32,.28,.95,14),jk);torso.position.y=1.45;torso.scale.set(1.25,1,.75);torso.castShadow=true;  
 const head=new THREE.Mesh(new THREE.SphereGeometry(.24,16,16),sk);head.position.y=2.22;head.castShadow=true;  
 const hair=new THREE.Mesh(new THREE.SphereGeometry(.25,16,12,0,Math.PI*2,0,Math.PI*.55),M(0x221a15,.9));hair.position.set(0,2.24,.02);  
 const leg=x=>{const h=new THREE.Group();h.position.set(x,1,0);const l=new THREE.Mesh(new THREE.CylinderGeometry(.13,.1,.85,10),pt);l.position.y=-.42;l.castShadow=true;h.add(l);g.add(h);return h;};  
 const lL=leg(-.18),rL=leg(.18),rS=new THREE.Group();rS.position.set(.46,1.82,0);rS.rotation.x=1.15;  
 const rA=new THREE.Mesh(new THREE.CylinderGeometry(.1,.085,.78,10),jk);rA.position.y=-.35;rS.add(rA);  
 const gun=new THREE.Mesh(new THREE.BoxGeometry(.08,.14,.8),gm);gun.position.set(-.08,-.68,-.3);gun.rotation.x=-1.15;rS.add(gun);  
 g.add(torso,head,hair,rS);g.userData={lL,rL,wt:0};return g;  
}  
  
function makeCar(col=0x8e2424){  
 const c=new THREE.Group(),pm=M(col,.25,.6),gl=M(0x111822,.1,.9),rm=M(0x181818,.9);  
 const b=new THREE.Mesh(new THREE.BoxGeometry(2.3,.72,4.7),pm);b.position.y=.72;b.castShadow=true;  
 const cb=new THREE.Mesh(new THREE.BoxGeometry(1.95,.65,2.35),gl);cb.position.set(0,1.38,.15);  
 c.add(b,cb);const wh=[];  
 [[-1.18,.44,-1.45],[1.18,.44,-1.45],[-1.18,.44,1.45],[1.18,.44,1.45]].forEach(p=>{  
  const w=new THREE.Mesh(new THREE.CylinderGeometry(.44,.44,.32,14),rm);w.rotation.z=Math.PI/2;w.position.set(...p);c.add(w);wh.push(w);  
 });  
 c.userData={wh,drv:null};return c;  
}  
  
const CS=90,chunks=new Set(),foods=[],cars=[],gMat=M(0x3f6838,.92),bMat=M(0x4a3525,.95),pMat=M(0x244223,.85),cMat=M(0x7a7d80,.88),aMat=M(0xc02929,.3,.1);  
function genChunk(cx,cz){  
 const k=cx+','+cz;if(chunks.has(k))return;chunks.add(k);  
 const sd=Math.abs(Math.sin(cx*12.9898+cz*78.233)*43758.5453);  
 const geo=new THREE.PlaneGeometry(CS,CS,14,14);geo.rotateX(-Math.PI/2);  
 const p=geo.attributes.position;for(let i=0;i<p.count;i++)p.setY(i,hAt(p.getX(i)+cx*CS,p.getZ(i)+cz*CS));  
 geo.computeVertexNormals();const gr=new THREE.Mesh(geo,gMat);gr.position.set(cx*CS,0,cz*CS);gr.receiveShadow=true;scene.add(gr);  
 for(let i=0;i<6;i++){  
  const rx=cx*CS+(((sd*(i+1)*19)%1)-.5)*(CS-14),rz=cz*CS+(((sd*(i+2)*37)%1)-.5)*(CS-14),ry=hAt(rx,rz);  
  if(i===0&&(cx+cz)%2===0){const h=7+(sd%5),bl=new THREE.Mesh(new THREE.BoxGeometry(11,h,11),cMat);bl.position.set(rx,ry+h/2-.5,rz);bl.castShadow=true;scene.add(bl);}  
  else{const tr=new THREE.Mesh(new THREE.CylinderGeometry(.35,.55,3.5,8),bMat);tr.position.set(rx,ry+1.5,rz);const cn=new THREE.Mesh(new THREE.ConeGeometry(2.5,6,8),pMat);cn.position.set(rx,ry+5.5,rz);cn.castShadow=true;scene.add(tr,cn);}  
 }  
 const fx=cx*CS+((sd*43)%50)-25,fz=cz*CS+((sd*89)%50)-25,ap=new THREE.Mesh(new THREE.SphereGeometry(.28,10,10),aMat);  
 ap.position.set(fx,hAt(fx,fz)+.28,fz);scene.add(ap);foods.push(ap);  
 if(Math.abs(cx*3+cz*7)%3===0){const car=makeCar([0x8e2424,0x2c3e50,0xd5d8dc][Math.abs(cx+cz)%3]);car.position.set(cx*CS+12,hAt(cx*CS+12,cz*CS-12),cz*CS-12);scene.add(car);cars.push(car);}  
}  
  
const me=makeChar(0x28527a),sx=(Math.random()-.5)*300,sz=(Math.random()-.5)*300;  
me.position.set(sx,hAt(sx,sz),sz);scene.add(me);  
let st={hp:100,hg:100,yaw:0,adm:false,god:false,spd:1},car=null,bul=[],rem={};  
const ADMIN_PASS="1234"; // Change your admin password here  
const hud=()=>{document.getElementById('hpt').innerText=st.god?'GOD':st.hp+'%';document.getElementById('hp').style.width=st.hp+'%';document.getElementById('hgt').innerText=st.god?'GOD':st.hg+'%';document.getElementById('hg').style.width=st.hg+'%';};  
const msg=(s,t,d)=>{const b=document.getElementById('msgs');b.innerHTML+=`<div><b style="color:${s==='ADMIN'?'#f1c40f':'#5dade2'}">[${s}]${d!==null?' ('+d+'m)':''}:</b> ${t}</div>`;b.scrollTop=b.scrollHeight;};  
  
function runCommand(raw){  
 const args=raw.trim().split(' '),cmd=args[0].toLowerCase();  
 if(cmd==='/admin'){  
  if(args[1]===ADMIN_PASS){st.adm=true;document.getElementById('adm-tag').style.display='block';msg('ADMIN','Admin unlocked! Commands: /god, /car, /heal, /speed [num], /tp [x] [z], /sayall [msg]',null);}  
  else msg('SYS','Wrong admin password.',null);  
  return;  
 }  
 if(!st.adm){msg('SYS','Unknown command. Unlock admin with /admin [password]',null);return;}  
 if(cmd==='/god'){st.god=!st.god;st.hp=st.hg=100;hud();msg('ADMIN','God Mode: '+(st.god?'ON':'OFF'),null);}  
 else if(cmd==='/heal'){st.hp=st.hg=100;hud();msg('ADMIN','Health & Hunger restored.',null);}  
 else if(cmd==='/car'){const c=makeCar(0xf1c40f);c.position.set(me.position.x+3,hAt(me.position.x+3,me.position.z),me.position.z);scene.add(c);cars.push(c);msg('ADMIN','Spawned Admin Car nearby.',null);}  
 else if(cmd==='/speed'){st.spd=Math.max(.1,parseFloat(args[1])||1);msg('ADMIN','Speed multiplier set to '+st.spd+'x',null);}  
 else if(cmd==='/tp'){const tx=parseFloat(args[1])||0,tz=parseFloat(args[2])||0;me.position.set(tx,hAt(tx,tz),tz);if(car)car.position.copy(me.position);msg('ADMIN',`Teleported to ${tx}, ${tz}`,null);}  
 else if(cmd==='/sayall'){const txt=args.slice(1).join(' ');msg('ADMIN',txt,null);send({t:'g',txt});}  
 else msg('ADMIN','Commands: /god, /car, /heal, /speed [n], /tp [x] [z], /sayall [msg]',null);  
}  
  
setInterval(()=>{if(st.god)return;st.hg=Math.max(0,st.hg-1);if(!st.hg)st.hp=Math.max(0,st.hp-3);if(st.hp<=0){msg("SYS","Respawning...",0);if(car){car.userData.drv=null;car=null;me.visible=true;}me.position.set((Math.random()-.5)*400,0,(Math.random()-.5)*400);st.hp=st.hg=100;}hud();},4000);  
  
const peer=new Peer(),conns=[];  
peer.on('open',id=>document.getElementById('mid').innerText=id);  
peer.on('connection',c=>setup(c));  
function join(){const id=document.getElementById('jid').value.trim();if(id)setup(peer.connect(id));}  
function setup(c){conns.push(c);document.getElementById('st').innerText=`● ${conns.length} Connected`;c.on('data',d=>{  
 if(d.t==='m'){if(!rem[c.peer]){rem[c.peer]=makeChar(0x8e352e);scene.add(rem[c.peer]);}rem[c.peer].position.set(d.x,d.y,d.z);rem[c.peer].rotation.y=d.yaw;rem[c.peer].visible=!d.c;}  
 if(d.t==='c'&&Math.hypot(me.position.x-d.x,me.position.z-d.z)<=35)msg(c.peer.slice(0,5),d.txt,Math.round(Math.hypot(me.position.x-d.x,me.position.z-d.z)));  
 if(d.t==='g')msg('ADMIN',d.txt,null);  
 if(d.t==='s'){const b=new THREE.Mesh(new THREE.SphereGeometry(.08),new THREE.MeshBasicMaterial({color:0xffaa00}));b.position.set(d.x,d.y,d.z);b.userData={dir:new THREE.Vector3(-Math.sin(d.yaw),0,-Math.cos(d.yaw)),l:70,e:1};scene.add(b);bul.push(b);}  
});}  
const send=d=>conns.forEach(c=>{if(c.open)c.send(d);});  
  
const keys={};  
onkeydown=e=>{  
 const inp=document.getElementById('in');  
 if(document.activeElement===inp){  
  if(e.key==='Enter'&&inp.value.trim()){  
   const val=inp.value.trim();  
   if(val.startsWith('/'))runCommand(val);  
   else{msg("You",val,0);send({t:'c',txt:val,x:me.position.x,z:me.position.z});}  
   inp.value='';inp.blur();  
  }return;  
 }  
 if(e.key==='Enter'){inp.focus();return;}  
 if(e.key.toLowerCase()==='f'){  
  if(car){me.position.set(car.position.x+2.2,hAt(car.position.x+2.2,car.position.z),car.position.z);car.userData.drv=null;car=null;me.visible=true;}  
  else for(let c of cars)if(me.position.distanceTo(c.position)<4.5&&!c.userData.drv){car=c;c.userData.drv=me;me.visible=false;break;}  
 }  
 if(e.key.toLowerCase()==='e')for(let i=foods.length-1;i>=0;i--)if(me.position.distanceTo(foods[i].position)<3.5){scene.remove(foods[i]);foods.splice(i,1);st.hg=Math.min(100,st.hg+30);st.hp=Math.min(100,st.hp+12);hud();msg("EAT","+30 Hunger, +12 HP",0);break;}  
 keys[e.key.toLowerCase()]=true;  
};  
onkeyup=e=>keys[e.key.toLowerCase()]=false;  
ren.domElement.onclick=()=>{ren.domElement.requestPointerLock();if(document.pointerLockElement===ren.domElement&&!car){  
 const b=new THREE.Mesh(new THREE.SphereGeometry(.08),new THREE.MeshBasicMaterial({color:0xffe066}));b.position.set(me.position.x,me.position.y+1.55,me.position.z);  
 b.userData={dir:new THREE.Vector3(-Math.sin(st.yaw),0,-Math.cos(st.yaw)),l:70};scene.add(b);bul.push(b);send({t:'s',x:b.position.x,y:b.position.y,z:b.position.z,yaw:st.yaw});  
}};  
onmousemove=e=>{if(document.pointerLockElement===ren.domElement)st.yaw-=e.movementX*.0028;};  
  
function loop(){  
 requestAnimationFrame(loop);let mv=0;  
 const fw=new THREE.Vector3(-Math.sin(st.yaw),0,-Math.cos(st.yaw)),rt=new THREE.Vector3(Math.cos(st.yaw),0,-Math.sin(st.yaw));  
 if(car){  
  if(keys['w']){car.position.addScaledVector(fw,.85*st.spd);mv=1;}if(keys['s']){car.position.addScaledVector(fw,-.4*st.spd);mv=1;}  
  car.rotation.y=st.yaw;car.position.y=hAt(car.position.x,car.position.z);me.position.copy(car.position);  
  if(mv)car.userData.wh.forEach(w=>w.rotation.x-=.25*st.spd);  
 }else{  
  if(keys['w']){me.position.addScaledVector(fw,.21*st.spd);mv=1;}if(keys['s']){me.position.addScaledVector(fw,-.15*st.spd);mv=1;}  
  if(keys['a']){me.position.addScaledVector(rt,-.17*st.spd);mv=1;}if(keys['d']){me.position.addScaledVector(rt,.17*st.spd);mv=1;}  
  me.position.y=hAt(me.position.x,me.position.z);me.rotation.y=st.yaw;  
  if(mv){me.userData.wt+=.22;me.userData.lL.rotation.x=Math.sin(me.userData.wt)*.65;me.userData.rL.rotation.x=-Math.sin(me.userData.wt)*.65;}  
  else me.userData.lL.rotation.x=me.userData.rL.rotation.x=0;  
 }  
 const tip=document.getElementById('tip');  
 tip.innerText=!car?(cars.some(c=>me.position.distanceTo(c.position)<4.5)?"Press [F] to Drive":foods.some(f=>me.position.distanceTo(f.position)<3.5)?"Press [E] to Eat":""):"";  
 tip.style.display=tip.innerText?"block":"none";  
 for(let i=bul.length-1;i>=0;i--){  
  const b=bul[i];b.position.addScaledVector(b.userData.dir,3);b.userData.l--;  
  if(!st.god&&b.userData.e&&b.position.distanceTo(new THREE.Vector3(me.position.x,me.position.y+1.3,me.position.z))<1.1){st.hp=Math.max(0,st.hp-38);hud();b.userData.l=0;}  
  if(b.userData.l<=0){scene.remove(b);bul.splice(i,1);}  
 }  
 sun.position.set(me.position.x+120,220,me.position.z+100);sun.target.position.copy(me.position);  
 const off=new THREE.Vector3(car?0:.85,car?3.8:2.35,car?8.5:4.2).applyAxisAngle(new THREE.Vector3(0,1,0),st.yaw);  
 cam.position.copy(me.position).add(off);cam.lookAt(me.position.x-Math.sin(st.yaw)*12,me.position.y+1.6,me.position.z-Math.cos(st.yaw)*12);  
 const cx=Math.floor(me.position.x/CS),cz=Math.floor(me.position.z/CS);  
 for(let x=-2;x<=2;x++)for(let z=-2;z<=2;z++)genChunk(cx+x,cz+z);  
 send({t:'m',x:me.position.x,y:me.position.y,z:me.position.z,yaw:st.yaw,c:!!car});  
 ren.render(scene,cam);  
}  
loop();  
</script>  
</body>  
</html>  
