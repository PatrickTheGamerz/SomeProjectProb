<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Local LLM Chat (Dark)</title>
<meta name="viewport" content="width=device-width,initial-scale=1">
<style>
:root{--bg:#0b0f14;--panel:#0f1720;--muted:#9aa6b2;--accent:#7c5cff}
body{background:linear-gradient(180deg,#071018 0%,var(--bg) 100%);color:#e6eef6;font-family:Inter,system-ui;margin:0;padding:20px}
.container{max-width:900px;margin:0 auto}
.header{display:flex;align-items:center;gap:12px}
.h1{font-size:20px;font-weight:600}
.controls{margin-top:12px;display:flex;gap:8px;align-items:center}
#log{height:60vh;overflow:auto;background:var(--panel);border-radius:10px;padding:16px;margin-top:14px;box-shadow:0 6px 18px rgba(0,0,0,0.6)}
.msg{padding:10px;border-radius:8px;margin-bottom:8px;max-width:80%}
.user{background:linear-gradient(90deg,#0b1220,#071226);margin-left:auto}
.bot{background:linear-gradient(90deg,#0f1726,#0b1220)}
.inputRow{display:flex;gap:8px;margin-top:12px}
textarea{flex:1;min-height:64px;background:#07101a;color:#e6eef6;border:1px solid #12202b;padding:10px;border-radius:8px}
button{background:var(--accent);color:white;border:none;padding:10px 14px;border-radius:8px;cursor:pointer}
.small{font-size:13px;color:var(--muted)}
</style>
</head>
<body>
<div class="container">
  <div class="header">
    <div class="h1">Local LLM Chat</div>
    <div class="small">Run a model file in your browser (no API)</div>
  </div>

  <div class="controls">
    <label class="small">Load model file:</label>
    <input id="modelFile" type="file" accept=".bin,.ggml,.bin.q4_0" />
    <button id="initBtn">Initialize Model</button>
    <div id="status" class="small">Idle</div>
  </div>

  <div id="log"></div>

  <div class="inputRow">
    <textarea id="prompt" placeholder="Type your message..."></textarea>
    <button id="send">Send</button>
  </div>
  <div class="small" style="margin-top:8px">Tip: use a quantized GGML model for best browser performance.</div>
</div>

<script>
/* NOTE: This file shows the UI and control flow. To actually run inference you must
   include a WASM binding such as llama-cpp-wasm or wllama and call its API.
   Example integration points are marked below. See project docs: llama-cpp-wasm, wllama. */

const logEl = document.getElementById('log');
const status = document.getElementById('status');
let modelLoaded = false;
let modelHandle = null; // placeholder for WASM model instance

function append(who, text){
  const d = document.createElement('div');
  d.className = 'msg ' + (who==='user'?'user':'bot');
  d.textContent = text;
  logEl.appendChild(d);
  logEl.scrollTop = logEl.scrollHeight;
}

document.getElementById('initBtn').onclick = async ()=>{
  const f = document.getElementById('modelFile').files[0];
  if(!f){alert('Choose a model file first'); return;}
  status.textContent = 'Loading model...';
  // --- Integration point: load file into WASM runtime ---
  // Example pseudocode:
  // const arrayBuffer = await f.arrayBuffer();
  // modelHandle = await wasmBinding.loadModel(arrayBuffer, {quantized:true});
  // modelLoaded = true;
  // ----------------------------------------------------
  // For demo UI we simulate:
  await new Promise(r=>setTimeout(r,800));
  modelLoaded = true;
  status.textContent = 'Model loaded (simulated)';
};

document.getElementById('send').onclick = async ()=>{
  const p = document.getElementById('prompt').value.trim();
  if(!p) return;
  append('user', p);
  document.getElementById('prompt').value = '';
  append('bot', 'Thinking...');
  // --- Integration point: run inference ---
  // const reply = await wasmBinding.generate(modelHandle, p, {max_tokens:256,temperature:0.7});
  // Replace the simulated reply below with real output.
  await new Promise(r=>setTimeout(r,800));
  logEl.lastChild.textContent = 'Bot: ' + 'This is a simulated reply. Replace with WASM inference output.';
};
</script>
</body>
</html>
