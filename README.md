<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>MoonVM Key System</title>

<style>
:root{
  --purple: rgb(170,164,209);
  --cyan:#5efcff;
  --danger:#ff6b6b;
}
*{box-sizing:border-box;font-family:"Segoe UI",system-ui,sans-serif}
body{
  margin:0;height:100vh;
  background:radial-gradient(circle at top,#1a1f3d,#080a14);
  display:flex;align-items:center;justify-content:center;
}
body::before{
  content:"";position:absolute;width:760px;height:760px;
  background:linear-gradient(45deg,var(--purple),var(--cyan));
  filter:blur(240px);opacity:.35;
}
.card{
  width:480px;padding:30px;border-radius:22px;
  background:linear-gradient(180deg,rgba(255,255,255,.12),rgba(255,255,255,.03));
  backdrop-filter:blur(18px);
  border:1px solid rgba(170,164,209,.35);
  box-shadow:0 0 45px rgba(170,164,209,.45);
}
.title{text-align:center;font-size:26px;font-weight:700;color:var(--purple);margin-bottom:22px}
.hidden{display:none}
canvas{
  display:block;margin:0 auto 16px;border-radius:14px;
  background:rgba(8,10,24,.8);
  border:1px solid rgba(94,252,255,.35);
}
input{
  width:100%;height:50px;border-radius:14px;border:1px solid rgba(94,252,255,.35);
  background:rgba(15,20,45,.45);color:var(--cyan);font-size:16px;text-align:center;outline:none;margin-bottom:14px
}
button{
  width:100%;height:52px;border:none;border-radius:16px;
  background:linear-gradient(135deg,var(--purple),var(--cyan));
  color:#05070f;font-size:16px;font-weight:700;cursor:pointer
}
.row{display:flex;gap:10px}
.msg{font-size:13px;text-align:center;margin-top:10px}
.error{color:var(--danger)}
.info{color:rgba(200,220,255,.75)}
.keybox{
  height:54px;border-radius:14px;background:rgba(15,20,45,.45);
  color:var(--cyan);display:flex;align-items:center;justify-content:center;
  letter-spacing:1px;border:1px solid rgba(94,252,255,.35);margin-bottom:12px
}
</style>
</head>

<body>

<div id="captchaUI" class="card">
  <div class="title">Verification</div>
  <canvas id="captchaCanvas" width="380" height="140"></canvas>
  <input id="captchaInput" placeholder="Enter captcha">
  <div class="row">
    <button onclick="verifyCaptcha()">Verify</button>
    <button onclick="speakCaptcha()">🔊 Read</button>
  </div>
  <div id="captchaMsg" class="msg error"></div>
</div>

<div id="keyUI" class="card hidden">
  <div class="title">MoonVM Key System</div>
  <div id="keyBox" class="keybox">Press Generate</div>
  <div class="row">
    <button id="genBtn">Generate</button>
    <button id="copyBtn">Copy</button>
  </div>
  <div id="timer" class="msg info"></div>
  <div id="keyMsg" class="msg info"></div>
</div>

<script>

const canvas = document.getElementById("captchaCanvas");
const ctx = canvas.getContext("2d");
let captchaValue = "";

const r = (a,b)=>Math.random()*(b-a)+a;

function genCaptcha(){
  captchaValue="";
  ctx.clearRect(0,0,canvas.width,canvas.height);

  const chars="ABCDEFGHJKLMNPQRSTUVWXYZabcdefghkmnpqrstuvwxyz23456789";
  for(let i=0;i<7;i++) captchaValue+=chars[Math.floor(Math.random()*chars.length)];


  for(let i=0;i<900;i++){
    ctx.fillStyle=`rgba(${Math.floor(r(100,255))},${Math.floor(r(100,255))},255,${r(0.02,0.08)})`;
    ctx.fillRect(r(0,380),r(0,140),1,1);
  }


  let x=18;
  for(const ch of captchaValue){
    ctx.save();
    ctx.translate(x,r(70,105));
    ctx.rotate(r(-1.2,1.2));
    ctx.transform(
      1+r(-0.6,0.6),
      r(-0.9,0.9),
      r(-0.9,0.9),
      1+r(-0.6,0.6),
      r(-6,6),
      r(-6,6)
    );

    ctx.font=`${r(34,58)}px Impact, Arial Black, Segoe UI`;
    ctx.fillStyle=`hsl(${r(160,240)},100%,${r(55,80)}%)`;
    ctx.shadowColor=`hsl(${r(160,240)},100%,70%)`;
    ctx.shadowBlur=r(12,28);
    ctx.fillText(ch,0,0);
    ctx.restore();

    x+=r(32,56);
  }


  for(let i=0;i<18;i++){
    ctx.save();
    ctx.globalAlpha=r(0.35,0.7);
    ctx.fillStyle=`hsl(${r(0,360)},80%,30%)`;
    ctx.translate(r(0,380),r(0,140));
    ctx.rotate(r(0,Math.PI));
    ctx.fillRect(r(-40,0),r(-20,0),r(20,80),r(6,22));
    ctx.restore();
  }

  for(let i=0;i<14;i++){
    ctx.strokeStyle=`rgba(255,255,255,${r(0.1,0.3)})`;
    ctx.lineWidth=r(1,3);
    ctx.beginPath();
    ctx.moveTo(r(0,380),r(0,140));
    ctx.lineTo(r(0,380),r(0,140));
    ctx.stroke();
  }

  // BLUR PATCHES
  for(let i=0;i<6;i++){
    ctx.save();
    ctx.filter=`blur(${r(2,6)}px)`;
    ctx.fillStyle=`rgba(20,30,60,${r(0.4,0.7)})`;
    ctx.fillRect(r(0,380),r(0,140),r(40,120),r(18,40));
    ctx.restore();
  }
}

function verifyCaptcha(){
  const v=document.getElementById("captchaInput").value.trim();
  if(v.toLowerCase()!==captchaValue.toLowerCase()){
    document.getElementById("captchaMsg").textContent="Captcha not correct";
    document.getElementById("captchaInput").value="";
    genCaptcha();
    return;
  }
  captchaUI.classList.add("hidden");
  keyUI.classList.remove("hidden");
  loadKey();
}

function speakCaptcha(){
  speechSynthesis.cancel();
  const voices=speechSynthesis.getVoices();
  let i=0;
  function next(){
    if(i>=captchaValue.length) return;
    const phrases=[
      `The next character is ${captchaValue[i]}`,
      `Listen carefully, it is ${captchaValue[i]}`,
      `Now slowly, the character is ${captchaValue[i]}`
    ];
    const u=new SpeechSynthesisUtterance(phrases[Math.floor(Math.random()*phrases.length)]);
    if(voices.length) u.voice=voices[Math.floor(Math.random()*voices.length)];
    u.pitch=Math.random()*1.8+0.2;
    u.rate=Math.random()*0.4+0.45;
    u.onend=()=>setTimeout(()=>{i++;next()},Math.random()*700+400);
    speechSynthesis.speak(u);
  }
  next();
}


const KEY="MoonVM_Key", TIME="MoonVM_Time", EXPIRE=86400000;
const box=keyBox, genBtn=document.getElementById("genBtn"),
copyBtn=document.getElementById("copyBtn"),
timer=document.getElementById("timer"), keyMsg=document.getElementById("keyMsg");
let interval;

function makeKey(){
  const c="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
  let s=""; for(let i=0;i<18;i++) s+=c[Math.floor(Math.random()*c.length)];
  return "MoonVMKey-"+s;
}
function startTimer(end){
  clearInterval(interval);
  interval=setInterval(()=>{
    const d=end-Date.now();
    if(d<=0){timer.textContent="Key expired";return;}
    timer.textContent=`Expires in ${Math.floor(d/3600000)}h ${Math.floor(d/60000)%60}m ${Math.floor(d/1000)%60}s`;
  },1000);
}
function loadKey(){
  const k=localStorage.getItem(KEY), t=+localStorage.getItem(TIME);
  if(!k||Date.now()-t>EXPIRE) return;
  box.textContent=k; genBtn.disabled=true; keyMsg.textContent="Already has key";
  startTimer(t+EXPIRE);
}
genBtn.onclick=()=>{
  const k=localStorage.getItem(KEY), t=+localStorage.getItem(TIME);
  if(k && Date.now()-t<EXPIRE){keyMsg.textContent="Already has key";return;}
  const nk=makeKey();
  localStorage.setItem(KEY,nk); localStorage.setItem(TIME,Date.now());
  box.textContent=nk; genBtn.disabled=true; keyMsg.textContent="Key generated";
  startTimer(Date.now()+EXPIRE);
};
copyBtn.onclick=()=>{
  const k=localStorage.getItem(KEY);
  if(!k){keyMsg.textContent="No key to copy";return;}
  navigator.clipboard?.writeText(k).then(()=>keyMsg.textContent="Key copied").catch(()=>{
    const ta=document.createElement("textarea");ta.value=k;document.body.appendChild(ta);
    ta.select();document.execCommand("copy");document.body.removeChild(ta);
    keyMsg.textContent="Key copied";
  });
};

genCaptcha();
</script>
</body>
</html>
