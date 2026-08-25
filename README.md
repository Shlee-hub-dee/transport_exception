<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Transport Payment Exception</title>
<style>
  :root{
    --ink:#12261f;        /* deep field green */
    --paper:#f6f4ec;      /* register paper */
    --line:#d9d3c2;
    --clay:#b8562f;       /* exception / reject */
    --leaf:#3f7d54;       /* approve */
    --amber:#c9911f;      /* pending */
    --slate:#5a6b63;
    --card:#fffdf7;
  }
  *{box-sizing:border-box}
  body{
    margin:0;background:var(--paper);color:var(--ink);
    font-family:"Inter",system-ui,-apple-system,Segoe UI,Roboto,sans-serif;
    font-size:15px;line-height:1.5;
  }
  header{
    background:var(--ink);color:var(--paper);
    padding:22px 28px;border-bottom:4px solid var(--clay);
  }
  header h1{margin:0;font-size:19px;letter-spacing:.3px;font-weight:600}
  header p{margin:4px 0 0;font-size:13px;color:#b9c9bf}
  nav{display:flex;gap:2px;background:var(--ink);padding:0 20px}
  nav button{
    background:transparent;border:0;color:#9db3a8;padding:12px 18px;
    font-size:13.5px;font-weight:600;cursor:pointer;border-bottom:3px solid transparent;
  }
  nav button.active{color:var(--paper);border-bottom-color:var(--clay)}
  nav .count{background:var(--clay);color:#fff;border-radius:10px;padding:1px 7px;font-size:11px;margin-left:6px}
  main{max-width:1080px;margin:0 auto;padding:26px 22px 60px}
  .setup{
    background:#fff6e8;border:1px solid var(--amber);border-radius:8px;
    padding:14px 16px;font-size:13px;margin-bottom:22px
  }
  .setup code{background:#00000010;padding:1px 5px;border-radius:4px}
  label{display:block;font-size:12.5px;font-weight:600;color:var(--slate);margin:14px 0 5px;text-transform:uppercase;letter-spacing:.4px}
  input,select,textarea{
    width:100%;padding:10px 12px;border:1px solid var(--line);border-radius:7px;
    background:var(--card);font:inherit;color:var(--ink)
  }
  input:focus,select:focus,textarea:focus{outline:2px solid var(--leaf);outline-offset:1px;border-color:var(--leaf)}
  textarea{min-height:74px;resize:vertical}
  .grid{display:grid;grid-template-columns:1fr 1fr;gap:0 18px}
  .full{grid-column:1/-1}
  .btn{
    border:0;border-radius:7px;padding:11px 20px;font:inherit;font-weight:600;cursor:pointer
  }
  .btn-primary{background:var(--ink);color:var(--paper);margin-top:20px}
  .btn-approve{background:var(--leaf);color:#fff}
  .btn-reject{background:var(--clay);color:#fff}
  .btn-ghost{background:transparent;border:1px solid var(--line);color:var(--ink)}
  .btn:disabled{opacity:.5;cursor:not-allowed}
  .hint{font-size:12px;color:var(--slate);margin-top:4px}
  .card{
    background:var(--card);border:1px solid var(--line);border-left:5px solid var(--amber);
    border-radius:8px;padding:16px 18px;margin-bottom:14px
  }
  .card.Approved{border-left-color:var(--leaf)}
  .card.Rejected{border-left-color:var(--clay)}
  .card .top{display:flex;justify-content:space-between;align-items:baseline;gap:12px;flex-wrap:wrap}
  .card .id{font-family:ui-monospace,Menlo,monospace;font-size:12px;color:var(--slate)}
  .pill{font-size:11px;font-weight:700;padding:2px 9px;border-radius:20px;text-transform:uppercase;letter-spacing:.4px}
  .pill.Pending{background:#fdf1d6;color:#8a6410}
  .pill.Approved{background:#dcefe2;color:#255c39}
  .pill.Rejected{background:#f3ddd3;color:#8a3417}
  .over{background:var(--clay);color:#fff;font-size:11px;padding:2px 8px;border-radius:4px;font-weight:700}
  .kv{display:grid;grid-template-columns:auto 1fr;gap:2px 14px;margin:10px 0;font-size:13.5px}
  .kv dt{color:var(--slate);font-weight:600}
  .kv dd{margin:0}
  .actions{display:flex;gap:8px;margin-top:12px;flex-wrap:wrap;align-items:center}
  .actions input{width:auto;flex:1;min-width:160px}
  a.ev{color:var(--clay);font-weight:600;text-decoration:none}
  a.ev:hover{text-decoration:underline}
  .empty{text-align:center;color:var(--slate);padding:50px 20px;font-size:14px}
  .toast{position:fixed;bottom:20px;left:50%;transform:translateX(-50%);background:var(--ink);color:var(--paper);padding:11px 20px;border-radius:8px;font-size:14px;opacity:0;transition:opacity .3s;pointer-events:none;z-index:9}
  .toast.show{opacity:1}
  @media(max-width:640px){.grid{grid-template-columns:1fr}}
</style>
</head>
<body>
<header>
  <h1>Transport Payment Exception Process</h1>
  <p>Raise, review and settle transport payment exceptions with a full audit trail.</p>
</header>
<nav>
  <button data-view="form" class="active">New Request</button>
  <button data-view="rm">RM Review <span class="count" id="c-rm">0</span></button>
  <button data-view="finance">Finance <span class="count" id="c-fin">0</span></button>
  <button data-view="register">Register</button>
</nav>
<main id="main"></main>
<div class="toast" id="toast"></div>

<script>
/* ====== SET THIS after deploying Code.gs as a Web App ====== */
const SCRIPT_URL = "https://script.google.com/a/macros/oneacrefund.org/s/AKfycbwJHLo-2umpXZB0vQniPxV3kztLFPRQTD7NaokJQyhcw-qFI85wfHMWDukPDKde7unS/exec"; // 
/* Regions/rates load from the script; these are fallbacks if URL is blank */
let CONFIG = { regions:["Region A","Region B","Region C"], rates:{matatu:0,bike:0} };
let ROWS = [];

const $ = s => document.querySelector(s);
const main = $("#main");
const toast = (m)=>{const t=$("#toast");t.textContent=m;t.classList.add("show");setTimeout(()=>t.classList.remove("show"),2400)};

function readFile(file){
  return new Promise((res,rej)=>{
    const r = new FileReader();
    r.onload = ()=>res({ data:r.result.split(",")[1], mimeType:file.type, name:file.name });
    r.onerror = ()=>rej(new Error("read failed"));
    r.readAsDataURL(file);
  });
}
async function api(payload){
  if(!SCRIPT_URL){ toast("Set SCRIPT_URL first"); throw new Error("no url"); }
  const res = await fetch(SCRIPT_URL,{method:"POST",body:JSON.stringify(payload)});
  return res.json();
}
async function load(){
  if(!SCRIPT_URL){ render(); return; }
  try{
    const cfg = await fetch(SCRIPT_URL+"?action=config").then(r=>r.json());
    if(cfg.ok){ CONFIG = {regions:cfg.regions, rates:cfg.rates}; }
    const list = await fetch(SCRIPT_URL+"?action=list").then(r=>r.json());
    if(list.ok){ ROWS = list.rows; }
  }catch(e){ toast("Could not reach the script"); }
  render();
}

let view="form";
document.querySelectorAll("nav button").forEach(b=>b.onclick=()=>{
  view=b.dataset.view;
  document.querySelectorAll("nav button").forEach(x=>x.classList.toggle("active",x===b));
  render();
});

function counts(){
  $("#c-rm").textContent = ROWS.filter(r=>r.Status==="Pending").length;
  $("#c-fin").textContent = ROWS.filter(r=>r.Status==="Approved" && r["Finance Status"]!=="Paid").length;
}

function setupBanner(){
  if(SCRIPT_URL) return "";
  return `<div class="setup"><strong>Not connected yet.</strong> Deploy <code>Code.gs</code> as a Web App, paste its <code>/exec</code> URL into <code>SCRIPT_URL</code> at the top of this file. Until then the form shows fallback regions and nothing saves.</div>`;
}

function render(){
  counts();
  if(view==="form") return renderForm();
  if(view==="rm") return renderList("Pending","RM Review — pending requests");
  if(view==="finance") return renderFinance();
  if(view==="register") return renderRegister();
}

/* ---------- FORM ---------- */
function renderForm(){
  const regionOpts = CONFIG.regions.map(r=>`<option>${r}</option>`).join("");
  main.innerHTML = setupBanner() + `
  <div class="grid">
    <div><label>Region</label><select id="f-region">${regionOpts}</select></div>
    <div><label>Requester name</label><input id="f-requester" placeholder="Your name"></div>
    <div class="full"><label>Exception type</label>
      <select id="f-type">
        <option>Missing Distance</option>
        <option>Incorrect Distance</option>
        <option>Archived Approval</option>
      </select>
    </div>
    <div><label>Transport date</label><input id="f-date" type="date"></div>
    <div><label>Transport reference</label><input id="f-ref" placeholder="Reference, if available"></div>
    <div><label>Transport mode</label><select id="f-mode"><option value="matatu">Matatu</option><option value="bike">Bike</option></select></div>
    <div><label>Amount requested (KES)</label><input id="f-amount" type="number" min="0"></div>
    <div id="wrap-origin"><label>Origin</label><input id="f-origin"></div>
    <div id="wrap-dest"><label>Destination</label><input id="f-dest"></div>
    <div id="wrap-route" style="display:none"><label>Route</label><input id="f-route" placeholder="e.g. Town A – Town B"></div>
    <div id="wrap-orig-dist"><label>Original distance (km)</label><input id="f-odist" type="number" min="0" placeholder="If captured incorrectly"></div>
    <div><label>Validated distance (km)</label><input id="f-vdist" type="number" min="0"></div>
    <div class="full"><label>Reason for the exception</label><textarea id="f-reason" placeholder="Why is distance missing/incorrect, or the approval archived?"></textarea></div>
    <div><label>Mpesa reference</label><input id="f-mpesa" placeholder="e.g. SLJ7XK9P2Q"></div>
    <div><label>Mpesa screenshot (required)</label><input id="f-shot" type="file" accept="image/*">
      <div class="hint">Must be dated to the time of the transport activity.</div></div>
    <div class="full"><label>Other evidence link (optional)</label><input id="f-ev" placeholder="Drive link: delivery note, dispatch/stock record, archive ticket...">
      <div class="hint" id="ev-hint"></div></div>
  </div>
  <button class="btn btn-primary" id="submit">Submit for RM review</button>
  `;
  const typeSel = $("#f-type"), hint = $("#ev-hint");
  const syncType = ()=>{
    const archived = typeSel.value==="Archived Approval";
    $("#wrap-route").style.display = archived ? "" : "none";
    $("#wrap-origin").style.display = archived ? "none" : "";
    $("#wrap-dest").style.display = archived ? "none" : "";
    $("#wrap-orig-dist").style.display = typeSel.value==="Incorrect Distance" ? "" : "none";
    hint.textContent = archived
      ? "Include Mpesa screenshot dated to the transport, plus the archive-confirmation ticket/escalation."
      : "Include a delivery note, dispatch record, stock movement, receiving confirmation or transport log.";
  };
  typeSel.onchange = syncType; syncType();

  $("#submit").onclick = async ()=>{
    const file = $("#f-shot").files[0];
    const p = {
      action:"submit",
      region:$("#f-region").value, requester:$("#f-requester").value.trim(),
      exceptionType:typeSel.value, transportDate:$("#f-date").value,
      transportReference:$("#f-ref").value.trim(), transportMode:$("#f-mode").value,
      origin:$("#f-origin")?.value||"", destination:$("#f-dest")?.value||"",
      route:$("#f-route")?.value||"",
      originalDistance:$("#f-odist").value, validatedDistance:$("#f-vdist").value,
      amount:$("#f-amount").value, reason:$("#f-reason").value.trim(),
      mpesaReference:$("#f-mpesa").value.trim(),
      evidenceLink:$("#f-ev").value.trim()
    };
    if(!p.requester||!p.reason){ toast("Requester and reason are required"); return; }
    if(!p.mpesaReference){ toast("Mpesa reference is required"); return; }
    if(!file){ toast("Mpesa screenshot is required"); return; }
    if(file.size > 5*1024*1024){ toast("Screenshot must be under 5 MB"); return; }

    $("#submit").disabled=true;
    try{
      p.screenshot = await readFile(file);
      const r = await api(p);
      if(r.ok){ toast("Submitted — "+r.id); await load(); }
      else toast(r.error||"Failed");
    }catch(e){ toast("Upload failed"); }
    $("#submit").disabled=false;
  };
}

/* ---------- CARD ---------- */
function card(row, withActions){
  const over = row["Over Limit"]==="YES" ? `<span class="over">Over rate limit</span>` : "";
  const ev = row["Evidence Link"] ? `<a class="ev" href="${row["Evidence Link"]}" target="_blank" rel="noopener">Open evidence ↗</a>` : "—";
  const shot = row["Screenshot Link"] ? `<a class="ev" href="${row["Screenshot Link"]}" target="_blank" rel="noopener">View screenshot ↗</a>` : "—";
  const dist = row["Validated Distance"] || "—";
  const rate = row["Rate/km"] ? `@ ${row["Rate/km"]}/km` : "";
  return `<div class="card ${row.Status}">
    <div class="top">
      <div><strong>${row["Exception Type"]}</strong> · ${row.Region}
        <span class="id"> ${row.ID}</span></div>
      <div><span class="pill ${row.Status}">${row.Status}</span> ${over}</div>
    </div>
    <dl class="kv">
      <dt>Requester</dt><dd>${row.Requester||"—"}</dd>
      <dt>Mode</dt><dd>${row["Transport Mode"]||"—"}</dd>
      <dt>Route</dt><dd>${row.Route || [row.Origin,row.Destination].filter(Boolean).join(" → ") || "—"}</dd>
      <dt>Distance</dt><dd>${dist} km ${rate}</dd>
      <dt>Amount</dt><dd>KES ${row.Amount||"—"}</dd>
      <dt>Reason</dt><dd>${row.Reason||"—"}</dd>
      <dt>Mpesa ref</dt><dd>${row["Mpesa Reference"]||"—"}</dd>
      <dt>Screenshot</dt><dd>${shot}</dd>
      <dt>Other evidence</dt><dd>${ev}</dd>
      ${row["RM Notes"]?`<dt>RM notes</dt><dd>${row["RM Notes"]}</dd>`:""}
      ${row["Finance Status"]?`<dt>Finance</dt><dd>${row["Finance Status"]}</dd>`:""}
    </dl>
    ${withActions||""}
  </div>`;
}

/* ---------- RM LIST ---------- */
function renderList(status, title){
  const rows = ROWS.filter(r=>r.Status===status);
  main.innerHTML = setupBanner() + `<h2 style="margin:0 0 16px;font-size:16px">${title}</h2>` +
    (rows.length? rows.map(r=>card(r, actionsRM(r.ID))).join("") : `<div class="empty">No pending requests. Cleared out.</div>`);
  rows.forEach(r=>bindRM(r.ID));
}
function actionsRM(id){
  return `<div class="actions">
    <input id="who-${id}" placeholder="Your name (Regional Manager)">
    <input id="note-${id}" placeholder="Notes / reason (required to reject)">
    <button class="btn btn-approve" id="ap-${id}">Approve</button>
    <button class="btn btn-reject" id="rj-${id}">Reject</button>
  </div>`;
}
function bindRM(id){
  const send = async (decision)=>{
    const manager = $("#who-"+id).value.trim();
    const notes = $("#note-"+id).value.trim();
    if(!manager){ toast("Enter your name"); return; }
    if(decision==="reject" && !notes){ toast("Give a reason to reject"); return; }
    const r = await api({action:"decision", id, decision, manager, notes});
    if(r.ok){ toast(decision==="approve"?"Approved":"Returned/Rejected"); await load(); }
    else toast(r.error||"Failed");
  };
  $("#ap-"+id).onclick=()=>send("approve");
  $("#rj-"+id).onclick=()=>send("reject");
}

/* ---------- FINANCE ---------- */
function renderFinance(){
  const rows = ROWS.filter(r=>r.Status==="Approved" && r["Finance Status"]!=="Paid");
  main.innerHTML = setupBanner() + `<h2 style="margin:0 0 16px;font-size:16px">Finance — approved & awaiting payment</h2>` +
    (rows.length? rows.map(r=>card(r, actionsFin(r.ID))).join("") : `<div class="empty">Nothing awaiting Finance.</div>`);
  rows.forEach(r=>bindFin(r.ID));
}
function actionsFin(id){
  return `<div class="actions">
    <input id="fin-${id}" placeholder="Finance officer name">
    <button class="btn btn-approve" id="paid-${id}">Mark Paid</button>
    <button class="btn btn-ghost" id="hold-${id}">Flag / Query</button>
  </div>`;
}
function bindFin(id){
  $("#paid-"+id).onclick=async()=>{
    const u=$("#fin-"+id).value.trim(); if(!u){toast("Enter your name");return;}
    const r=await api({action:"finance", id, status:"Paid", financeUser:u});
    if(r.ok){toast("Marked paid");await load();} else toast(r.error||"Failed");
  };
  $("#hold-"+id).onclick=async()=>{
    const u=$("#fin-"+id).value.trim(); if(!u){toast("Enter your name");return;}
    const r=await api({action:"finance", id, status:"Queried", financeUser:u});
    if(r.ok){toast("Flagged");await load();} else toast(r.error||"Failed");
  };
}

/* ---------- REGISTER ---------- */
function renderRegister(){
  if(!ROWS.length){ main.innerHTML=setupBanner()+`<div class="empty">The register is empty.</div>`; return; }
  main.innerHTML = setupBanner() + ROWS.slice().reverse().map(r=>card(r,"")).join("");
}

load();
</script>
</body>
</html>
