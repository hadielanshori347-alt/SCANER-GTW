<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="theme-color" content="#0b5fff">
<title>Scan IB/OB BDO</title>
<style>
:root{--bg:#f4f6fa;--card:#fff;--tx:#14181f;--mut:#667085;--bd:#dde2ea;--pri:#0b5fff;--ok:#12a150;--er:#d92d20;--wn:#b54708;
box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0e1116;--card:#171b22;--tx:#eef1f6;--mut:#98a2b3;--bd:#2a313c;--pri:#5b8cff;--ok:#32d583;--er:#f97066;--wn:#fdb022;color-scheme:dark}}
:root[data-theme="dark"]{--bg:#0e1116;--card:#171b22;--tx:#eef1f6;--mut:#98a2b3;--bd:#2a313c;--pri:#5b8cff;--ok:#32d583;--er:#f97066;--wn:#fdb022;color-scheme:dark}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--tx);font:15px/1.4 system-ui,-apple-system,"Segoe UI",Roboto,sans-serif}
.wrap{max-width:560px;margin:0 auto;padding:10px}
h1{font-size:17px;margin:4px 2px 8px}
.tabs{display:grid;grid-template-columns:1fr 1fr;gap:6px}
.tab{padding:10px 6px;border:1px solid var(--bd);background:var(--card);color:var(--tx);border-radius:10px;font-weight:600;font-size:13px}
.tab.on{background:var(--pri);border-color:var(--pri);color:#fff}
.card{background:var(--card);border:1px solid var(--bd);border-radius:12px;padding:10px;margin-top:10px}
label{font-size:12px;color:var(--mut);display:block;margin:6px 0 3px}
select,input{width:100%;padding:11px;border:1px solid var(--bd);border-radius:10px;background:var(--bg);color:var(--tx);font-size:16px}
.row{display:flex;gap:8px}.row>*{flex:1}
button.b{padding:11px;border:0;border-radius:10px;background:var(--pri);color:#fff;font-weight:600;font-size:14px}
button.g{background:transparent;color:var(--tx);border:1px solid var(--bd)}
button.r{background:transparent;color:var(--er);border:1px solid var(--er)}
#cam{display:none;margin-top:8px}#cam.on{display:block}
#reader{width:100%;border-radius:10px;overflow:hidden;background:#000}
#reader video{width:100%;display:block}
#msg{margin-top:8px;padding:9px;border-radius:10px;font-weight:600;font-size:14px;display:none}
#msg.ok{display:block;background:color-mix(in srgb,var(--ok) 18%,transparent);color:var(--ok)}
#msg.er{display:block;background:color-mix(in srgb,var(--er) 18%,transparent);color:var(--er)}
#msg.wn{display:block;background:color-mix(in srgb,var(--wn) 18%,transparent);color:var(--wn)}
.chips{display:flex;flex-wrap:wrap;gap:6px}
.chip{font-size:12px;padding:4px 8px;border-radius:99px;background:var(--bg);border:1px solid var(--bd)}
.chip b{color:var(--pri)}
.it{display:flex;align-items:center;gap:8px;padding:8px 0;border-bottom:1px solid var(--bd)}
.it:last-child{border:0}
.it .c{flex:1;min-width:0;word-break:break-all;font-size:13px}
.it small{color:var(--mut);display:block;font-size:11px}
.it button{border:0;background:none;color:var(--er);font-size:20px;padding:4px 8px}
.tot{font-size:26px;font-weight:700}
.mut{color:var(--mut);font-size:12px}
</style>
</head>
<body>
<div class="wrap">
  <h1>📦 Scan IB / OB – BDO</h1>
  <div class="tabs" id="tabs"></div>

  <div class="card">
    <label for="lane" id="laneLbl">Jalur</label>
    <select id="lane"></select>
    <label for="code" style="margin-top:10px">Kode Bag / AWB / BIG</label>
    <div class="row">
      <input id="code" placeholder="Scan atau ketik, lalu Enter" autocapitalize="characters" autocomplete="off" inputmode="text">
      <button class="b" id="add" style="flex:0 0 78px">Tambah</button>
    </div>
    <div class="row" style="margin-top:8px">
      <button class="b" id="camBtn">📷 Scan Kamera</button>
      <button class="b g" id="scanner">🔫 Alat Scanner</button>
    </div>
    <div id="cam"><div id="reader"></div></div>
    <div id="msg"></div>
  </div>

  <div class="card">
    <div class="row" style="align-items:center">
      <div><div class="mut">Total hari ini</div><div class="tot" id="tot">0</div></div>
      <div class="chips" id="chips" style="flex:2"></div>
    </div>
  </div>

  <div class="card">
    <div class="row" style="margin-bottom:6px">
      <button class="b r" id="clr">Hapus</button>
    </div>
    <div id="list"></div>
  </div>

  <div class="card">
    <div class="row" style="align-items:center"><div class="mut" id="sync" style="flex:2"></div><button class="b g" id="resend">Kirim ulang</button></div>
  </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/html5-qrcode/2.3.8/html5-qrcode.min.js"></script>
<script>
// Kolom scan sesuai spreadsheet asli (g=grup, f=nama kolom, c=nomor kolom, l=huruf kolom)
const MODES={"ibs":{"n":"Inbound Sameday","sheet":"INBOUND SAMEDAY","lanes":[{"g":"BAWEN","f":"INBOUND BAG/AWB/BIG","c":2,"l":"B"},{"g":"PP&PH","f":"INBOUND BAG/AWB/BIG","c":4,"l":"D"},{"g":"LINEHAUL","f":"INBOUND BAG/AWB/BIG","c":6,"l":"F"},{"g":"JABODETABEK","f":"INBOUND BAG/AWB/BIG","c":8,"l":"H"}]},"obs":{"n":"Outbound Sameday","sheet":"OUTBOUND SAMEDAY","lanes":[{"g":"JABODETABEK","f":"OUTBOUND BAG/AWB","c":2,"l":"B"},{"g":"JABODETABEK","f":"OUTBOUND BIG","c":4,"l":"D"},{"g":"JABODETABEK","f":"OUTBOUND DOKEL","c":6,"l":"F"},{"g":"BAWEN","f":"OUTBOUND REG/BIG BAG/AWB IKUT SAMEDAY","c":8,"l":"H"},{"g":"BAWEN","f":"OUTBOUND FROM SKD IKUT SAMEDAY","c":10,"l":"J"},{"g":"CIANJUR","f":"OUTBOUND AWB/BAG","c":12,"l":"L"},{"g":"CIANJUR","f":"OUTBOUND DOKEL","c":14,"l":"N"},{"g":"TASIK","f":"OUTBOUND AWB/BAG","c":16,"l":"P"},{"g":"TASIK","f":"OUTBOUND DOKEL","c":18,"l":"R"},{"g":"GARUT","f":"OUTBOUND AWB/BAG","c":20,"l":"T"},{"g":"GARUT","f":"OUTBOUND DOKEL","c":22,"l":"V"},{"g":"MAJALENGKA","f":"OUTBOUND AWB/BAG","c":24,"l":"X"},{"g":"MAJALENGKA","f":"OUTBOUND DOKEL","c":26,"l":"Z"},{"g":"MAJALENGKA","f":"OUTBOUND 260/SUMEDANG","c":28,"l":"AB"},{"g":"SUKAJADI","f":"OUTBOUND AWB/BAG PH TO PH","c":30,"l":"AD"},{"g":"SUKAJADI","f":"OUTBOUND DOKEL","c":32,"l":"AF"},{"g":"METRO","f":"OUTBOUND AWB/BAG","c":34,"l":"AH"},{"g":"METRO","f":"OUTBOUND DOKEL","c":36,"l":"AJ"},{"g":"DOKEL FROM PP TO PH","f":"IN PH","c":38,"l":"AL"},{"g":"DOKEL FROM PP TO PH","f":"FROM PP","c":39,"l":"AM"}]},"ibn":{"n":"Inbound Nextday","sheet":"INBOUND NEXTDAY","lanes":[{"g":"PP&PH","f":"INBOUND BAG/AWB/BIG","c":2,"l":"B"},{"g":"BAWEN","f":"INBOUND BAG/AWB/BIG","c":4,"l":"D"},{"g":"LINEHAUL","f":"INBOUND BAG/AWB/BIG","c":6,"l":"F"},{"g":"JABODETABEK","f":"INBOUND BAG/AWB/BIG","c":8,"l":"H"}]},"obn":{"n":"Outbound Nextday","sheet":"OUTBOUND NEXTDAY","lanes":[{"g":"JABODETABEK","f":"OUTBOUND BAG/AWB","c":2,"l":"B"},{"g":"JABODETABEK","f":"OUTBOUND DOKEL BAG/AWB","c":4,"l":"D"},{"g":"JABODETABEK","f":"OUTBOUND BIG","c":6,"l":"F"},{"g":"BAWEN","f":"OUTBOUND BAG/AWB","c":7,"l":"G"},{"g":"BAWEN","f":"OUTBOUND DOKEL BAG/AWB","c":9,"l":"I"},{"g":"BAWEN","f":"OUTBOUND BIG","c":11,"l":"K"},{"g":"MAJALENGKA","f":"OUTBOUND AWB/BAG","c":12,"l":"L"},{"g":"MAJALENGKA","f":"OUTBOUND DOKEL","c":14,"l":"N"},{"g":"MAJALENGKA","f":"OUTBOUND 260/SUMEDANG","c":16,"l":"P"},{"g":"METRO","f":"OUTBOUND AWB/BAG","c":18,"l":"R"},{"g":"METRO","f":"OUTBOUND DOKEL","c":20,"l":"T"},{"g":"SUKAJADI","f":"OUTBOUND AWB/BAG","c":22,"l":"V"},{"g":"SUKAJADI","f":"OUTBOUND DOKEL","c":24,"l":"X"},{"g":"TASIK","f":"OUTBOUND AWB/BAG","c":26,"l":"Z"},{"g":"TASIK","f":"OUTBOUND DOKEL","c":28,"l":"AB"},{"g":"GARUT","f":"OUTBOUND AWB/BAG","c":30,"l":"AD"},{"g":"GARUT","f":"OUTBOUND DOKEL","c":32,"l":"AF"},{"g":"DOKEL FROM PP TO PH","f":"IN PH","c":34,"l":"AH"},{"g":"DOKEL FROM PP TO PH","f":"FROM PP","c":35,"l":"AI"}]}};
// ===== ISI SEKALI: URL Web App Apps Script (yang berakhiran /exec) =====
const SCRIPT_URL="PASTE_URL_WEB_APP_DISINI";
const KEY="scan_ibob_v3";const url=SCRIPT_URL.startsWith("https://")?SCRIPT_URL:"";
let data={ibs:[],obs:[],ibn:[],obn:[]},mode="ibs",lastTxt="",lastAt=0,scanner=null,usingDetector=false,stream=null,raf=0;
try{const s=localStorage.getItem(KEY);if(s)data=Object.assign(data,JSON.parse(s))}catch(e){}
const $=id=>document.getElementById(id);
const save=()=>{try{localStorage.setItem(KEY,JSON.stringify(data))}catch(e){}};
const esc=s=>String(s).replace(/[&<>"]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;"}[c]));
const pad=n=>String(n).padStart(2,"0");
const now=()=>{const d=new Date();return d.getFullYear()+"-"+pad(d.getMonth()+1)+"-"+pad(d.getDate())+" "+pad(d.getHours())+":"+pad(d.getMinutes())+":"+pad(d.getSeconds())};
const today=()=>now().slice(0,10);

async function send(r){
 if(!url||r.sent)return;
 try{const res=await fetch(url,{method:"POST",headers:{"Content-Type":"text/plain;charset=utf-8"},body:JSON.stringify({t:r.t,mode:r.mode,sheet:MODES[r.mode].sheet,col:r.col,lane:r.lane,code:r.code})});
  const j=await res.json();if(j.ok||j.dup){r.sent=true;save();render()}}catch(e){}
 sync();
}
let sending=false;
async function sendPending(){if(sending)return;sending=true;try{for(const k in data)for(const r of data[k])if(!r.sent)await send(r)}finally{sending=false}sync()}
function sync(){const n=Object.values(data).flat().filter(r=>!r.sent).length;
 $("sync").textContent=url?(n?n+" data belum terkirim ke Sheet":"Terhubung ke Sheet • semua data terkirim"):"SCRIPT_URL belum diisi di index.html: data hanya tersimpan di HP"}
function beep(ok){try{const a=new (window.AudioContext||window.webkitAudioContext)(),o=a.createOscillator(),g=a.createGain();
 o.frequency.value=ok?1000:300;g.gain.value=.15;o.connect(g);g.connect(a.destination);o.start();o.stop(a.currentTime+(ok?.1:.35))}catch(e){}
 try{navigator.vibrate&&navigator.vibrate(ok?60:[120,60,120])}catch(e){}}
function msg(t,c){const m=$("msg");m.className=c;m.textContent=t}

function drawTabs(){$("tabs").innerHTML=Object.keys(MODES).map(k=>`<button class="tab ${k===mode?"on":""}" data-k="${k}">${MODES[k].n}</button>`).join("");
 document.querySelectorAll(".tab").forEach(b=>b.onclick=()=>{mode=b.dataset.k;drawTabs();drawLanes();render()})}
function drawLanes(){const sel=$("lane"),Ls=MODES[mode].lanes,gs={},order=[];
 Ls.forEach((x,i)=>{if(!gs[x.g]){gs[x.g]=[];order.push(x.g)}gs[x.g].push(i)});
 sel.innerHTML=order.map(g=>`<optgroup label="${esc(g)}">`+gs[g].map(i=>`<option value="${i}">${esc(Ls[i].f)} (${Ls[i].l})</option>`).join("")+`</optgroup>`).join("");
 try{const sv=localStorage.getItem(KEY+"_l_"+mode);if(sv&&Ls[+sv])sel.value=sv}catch(e){}
 sel.onchange=()=>{try{localStorage.setItem(KEY+"_l_"+mode,sel.value)}catch(e){}};
 $("laneLbl").textContent=mode[0]==="i"?"Kolom inbound":"Kolom outbound"}

function render(){
 const rows=data[mode].filter(r=>r.t.slice(0,10)===today());
 $("tot").textContent=rows.length;
 const cnt={};rows.forEach(r=>cnt[r.lane]=(cnt[r.lane]||0)+1);
 $("chips").innerHTML=Object.keys(cnt).map(l=>`<span class="chip">${esc(l)} <b>${cnt[l]}</b></span>`).join("")||'<span class="mut">Belum ada scan</span>';
 $("list").innerHTML=rows.slice().reverse().slice(0,200).map(r=>`<div class="it"><div class="c">${esc(r.code)}<small>${esc(r.lane)} • ${esc(r.t.slice(11))} • ${r.sent?"✓ Sheet":"⏳ belum terkirim"}</small></div><button data-id="${r.id}" aria-label="Hapus">×</button></div>`).join("");
 $("list").querySelectorAll("button").forEach(b=>b.onclick=()=>{data[mode]=data[mode].filter(r=>String(r.id)!==b.dataset.id);save();render()});
}

function addCode(raw){
 const code=String(raw||"").trim().toUpperCase();if(!code)return false;
 const LN=MODES[mode].lanes[+$("lane").value],lane=LN.g+" · "+LN.f;
 const dup=data[mode].find(r=>r.code===code&&r.t.slice(0,10)===today());
 if(dup){beep(false);msg("DUPLIKAT: "+code+" sudah discan ("+dup.lane+")","er");return false}
 const rec={id:Date.now()+Math.random(),t:now(),lane,col:LN.c,code,mode,sent:false};data[mode].push(rec);send(rec);
 save();render();beep(true);msg("OK: "+code+" → "+lane,"ok");return true;
}

function onScanText(t){const n=Date.now();if(t===lastTxt&&n-lastAt<2000)return;lastTxt=t;lastAt=n;addCode(t)}

async function startCam(){
 $("cam").classList.add("on");$("camBtn").textContent="⏹ Stop Kamera";
 try{
  if("BarcodeDetector"in window){
   usingDetector=true;const det=new BarcodeDetector();
   stream=await navigator.mediaDevices.getUserMedia({video:{facingMode:"environment"},audio:false});
   const v=document.createElement("video");v.setAttribute("playsinline","");v.muted=true;v.srcObject=stream;$("reader").innerHTML="";$("reader").appendChild(v);await v.play();
   const loop=async()=>{try{const r=await det.detect(v);if(r.length)onScanText(r[0].rawValue)}catch(e){}raf=requestAnimationFrame(()=>setTimeout(loop,120))};loop();
  }else if(window.Html5Qrcode){
   usingDetector=false;scanner=new Html5Qrcode("reader");
   await scanner.start({facingMode:"environment"},{fps:10,qrbox:{width:260,height:160}},onScanText,()=>{});
  }else throw new Error("no scanner");
 }catch(e){stopCam();msg("Kamera tidak bisa dibuka (izin kamera/koneksi). Gunakan kolom ketik atau alat scanner.","wn")}
}
async function stopCam(){
 cancelAnimationFrame(raf);
 if(stream){stream.getTracks().forEach(t=>t.stop());stream=null}
 if(scanner){try{await scanner.stop();scanner.clear()}catch(e){}scanner=null}
 $("reader").innerHTML="";$("cam").classList.remove("on");$("camBtn").textContent="📷 Scan Kamera";
}

$("add").onclick=()=>{if(addCode($("code").value))$("code").value="";$("code").focus()};
$("code").addEventListener("keydown",e=>{if(e.key==="Enter"){e.preventDefault();$("add").click()}});
$("camBtn").onclick=()=>$("cam").classList.contains("on")?stopCam():startCam();
$("scanner").onclick=()=>{$("code").focus();msg("Siap: arahkan scanner, kode masuk otomatis.","ok")};
$("clr").onclick=()=>{if(confirm("Hapus semua data "+MODES[mode].n+" hari ini?")){data[mode]=data[mode].filter(r=>r.t.slice(0,10)!==today());save();render()}};
$("resend").onclick=()=>sendPending().then(()=>msg(url?"Selesai mengirim ulang.":"SCRIPT_URL belum diisi.",url?"ok":"wn"));
drawTabs();drawLanes();render();sync();sendPending();
setInterval(sendPending,15000);window.addEventListener("online",sendPending);
</script>
</body>
</html>
