<!doctype html>
<html>
<head>
  <meta charset="utf-8"/>
  <title>Single‑File ChatGPT‑style</title>
  <style>
    body{font-family:system-ui,Segoe UI,Roboto,Arial;margin:20px}
    #log{height:60vh;overflow:auto;border:1px solid #ddd;padding:10px}
    .user{color:#0b5; margin:6px 0}
    .bot{color:#06f; margin:6px 0}
    textarea{width:100%;height:70px}
    input[type=text]{width:100%}
  </style>
</head>
<body>
  <h3>Single‑file Chat (OpenAI)</h3>
  <p><strong>Security note:</strong> do not paste a production key on public machines.</p>
  <label>OpenAI API key (paste for this session):</label>
  <input id="key" type="text" placeholder="sk-..." />
  <div id="log"></div>
  <textarea id="msg" placeholder="Type your message"></textarea>
  <button id="send">Send</button>

<script>
const log = (who, text) => {
  const d = document.createElement('div');
  d.className = who==='user'?'user':'bot';
  d.textContent = (who==='user'?'You: ':'Bot: ')+text;
  document.getElementById('log').appendChild(d);
  document.getElementById('log').scrollTop = 1e9;
};

async function send() {
  const key = document.getElementById('key').value.trim();
  if(!key){alert('Paste your OpenAI key'); return;}
  const user = document.getElementById('msg').value.trim();
  if(!user) return;
  log('user', user);
  document.getElementById('msg').value = '';
  // Minimal chat history kept client-side
  const messages = [{role:'user', content:user}];
  try {
    const resp = await fetch('https://api.openai.com/v1/chat/completions',{
      method:'POST',
      headers:{
        'Content-Type':'application/json',
        'Authorization':'Bearer '+key
      },
      body: JSON.stringify({
        model: 'gpt-3.5-turbo',
        messages: messages,
        max_tokens: 500,
        temperature: 0.7
      })
    });
    if(!resp.ok){
      const t = await resp.text();
      log('bot','Error: '+resp.status+' '+t);
      return;
    }
    const j = await resp.json();
    const text = j.choices?.[0]?.message?.content || '[no reply]';
    log('bot', text);
  } catch(e){
    log('bot','Network error: '+e.message);
  }
}

document.getElementById('send').onclick = send;
document.getElementById('msg').addEventListener('keydown', e=>{ if(e.key==='Enter' && (e.ctrlKey||e.metaKey)) send(); });
</script>
</body>
</html>
