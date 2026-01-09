<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Dark AI Chat — Single File</title>
<style>
  :root{
    --bg:#0b0f14;
    --card:#0f1720;
    --muted:#9aa6b2;
    --accent:#7c5cff;
    --accent-2:#00d4ff;
    --user:#1f2937;
    --bot:#0b1220;
    --glass: rgba(255,255,255,0.03);
    --radius:14px;
    --gap:14px;
    font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }

  *{box-sizing:border-box}
  html,body{height:100%}
  body{
    margin:0;
    background:
      radial-gradient(800px 400px at 10% 10%, rgba(124,92,255,0.08), transparent 8%),
      radial-gradient(600px 300px at 90% 90%, rgba(0,212,255,0.04), transparent 8%),
      var(--bg);
    color:#e6eef6;
    -webkit-font-smoothing:antialiased;
    -moz-osx-font-smoothing:grayscale;
    display:flex;
    align-items:center;
    justify-content:center;
    padding:28px;
  }

  .app {
    width:100%;
    max-width:920px;
    height:80vh;
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border-radius:20px;
    box-shadow: 0 10px 30px rgba(2,6,23,0.7), inset 0 1px 0 rgba(255,255,255,0.02);
    display:grid;
    grid-template-columns: 320px 1fr;
    overflow:hidden;
  }

  /* Sidebar */
  .sidebar{
    background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
    padding:20px;
    border-right: 1px solid rgba(255,255,255,0.03);
    display:flex;
    flex-direction:column;
    gap:12px;
  }
  .brand{
    display:flex;
    gap:12px;
    align-items:center;
  }
  .logo{
    width:44px;
    height:44px;
    border-radius:10px;
    background: linear-gradient(135deg,var(--accent),var(--accent-2));
    display:flex;
    align-items:center;
    justify-content:center;
    font-weight:700;
    color:white;
    box-shadow: 0 6px 18px rgba(124,92,255,0.12);
  }
  .brand h1{
    font-size:16px;
    margin:0;
    letter-spacing:0.2px;
  }
  .brand p{margin:0;font-size:12px;color:var(--muted)}

  .conversations{
    margin-top:6px;
    display:flex;
    flex-direction:column;
    gap:8px;
    overflow:auto;
    padding-right:6px;
  }
  .conv{
    background:var(--glass);
    padding:10px 12px;
    border-radius:10px;
    color:var(--muted);
    font-size:13px;
    cursor:pointer;
    transition:all .18s ease;
  }
  .conv:hover{transform:translateY(-3px); box-shadow: 0 6px 18px rgba(2,6,23,0.5)}
  .conv.active{background:linear-gradient(90deg, rgba(124,92,255,0.12), rgba(0,212,255,0.04)); color:#fff}

  .sidebar .footer{
    margin-top:auto;
    font-size:12px;
    color:var(--muted);
    display:flex;
    gap:8px;
    align-items:center;
  }

  /* Chat area */
  .chat {
    display:flex;
    flex-direction:column;
    padding:22px;
    gap:12px;
  }
  .chat-header{
    display:flex;
    align-items:center;
    justify-content:space-between;
    gap:12px;
  }
  .chat-title{
    display:flex;
    gap:12px;
    align-items:center;
  }
  .chat-title h2{margin:0;font-size:16px}
  .chat-title p{margin:0;color:var(--muted);font-size:13px}

  .messages{
    flex:1;
    overflow:auto;
    padding:8px;
    display:flex;
    flex-direction:column;
    gap:12px;
    scroll-behavior:smooth;
  }

  .bubble{
    max-width:72%;
    padding:12px 14px;
    border-radius:14px;
    line-height:1.35;
    font-size:14px;
    box-shadow: 0 6px 18px rgba(2,6,23,0.45);
    word-break:break-word;
  }
  .bubble.user{
    margin-left:auto;
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    color:#e6eef6;
    border-bottom-right-radius:6px;
  }
  .bubble.bot{
    margin-right:auto;
    background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
    border: 1px solid rgba(255,255,255,0.02);
    color:var(--muted);
    border-bottom-left-radius:6px;
  }

  .meta{
    font-size:12px;
    color:var(--muted);
    margin-top:6px;
  }

  /* Typing indicator */
  .typing{
    display:inline-flex;
    gap:6px;
    align-items:center;
    padding:8px 10px;
    border-radius:12px;
    background: rgba(255,255,255,0.02);
  }
  .dot{
    width:8px;height:8px;border-radius:50%;background:var(--muted);opacity:0.9;
    animation: blink 1s infinite;
  }
  .dot:nth-child(2){animation-delay:.15s}
  .dot:nth-child(3){animation-delay:.3s}
  @keyframes blink{0%{opacity:.15}50%{opacity:1}100%{opacity:.15}}

  /* Input area */
  .composer{
    display:flex;
    gap:10px;
    align-items:center;
    padding-top:8px;
    border-top:1px solid rgba(255,255,255,0.02);
  }
  .input{
    flex:1;
    display:flex;
    gap:8px;
    align-items:center;
    background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
    padding:10px;
    border-radius:12px;
    border:1px solid rgba(255,255,255,0.02);
  }
  .input input{
    background:transparent;
    border:0;
    outline:0;
    color:#e6eef6;
    font-size:14px;
    width:100%;
  }
  .send{
    background:linear-gradient(90deg,var(--accent),var(--accent-2));
    border:0;
    color:white;
    padding:10px 14px;
    border-radius:10px;
    cursor:pointer;
    font-weight:600;
    box-shadow: 0 8px 30px rgba(124,92,255,0.12);
    transition:transform .12s ease;
  }
  .send:active{transform:translateY(1px)}
  .send:disabled{opacity:.5;cursor:not-allowed}

  /* small screens */
  @media (max-width:880px){
    .app{grid-template-columns:1fr; height:92vh}
    .sidebar{display:none}
    .chat{padding:16px}
  }
</style>
</head>
<body>
  <main class="app" role="application" aria-label="AI chat">
    <aside class="sidebar" aria-hidden="false">
      <div class="brand">
        <div class="logo">AI</div>
        <div>
          <h1>Dark Chat</h1>
          <p>Local demo • Replies: Hi or Hello</p>
        </div>
      </div>

      <div class="conversations" id="conversations" aria-hidden="false">
        <div class="conv active">New chat</div>
      </div>

      <div class="footer">
        <div style="flex:1;color:var(--muted);font-size:13px">No backend • Privacy friendly</div>
        <div style="font-size:12px;color:var(--muted)">v1.0</div>
      </div>
    </aside>

    <section class="chat" aria-live="polite">
      <header class="chat-header">
        <div class="chat-title">
          <div style="width:44px;height:44px;border-radius:10px;background:linear-gradient(135deg,var(--accent),var(--accent-2));display:flex;align-items:center;justify-content:center;font-weight:700">B</div>
          <div>
            <h2>Bot</h2>
            <p>Built-in responder that only says Hi or Hello</p>
          </div>
        </div>
        <div style="color:var(--muted);font-size:13px">Local demo</div>
      </header>

      <div class="messages" id="messages" role="log" aria-live="polite" aria-atomic="false">
        <div class="bubble bot" id="welcome">
          Hello — this demo bot only replies with Hi or Hello. Try typing anything and press Enter.
          <div class="meta">Polished dark UI • No network required</div>
        </div>
      </div>

      <div class="composer" aria-hidden="false">
        <div class="input" role="search">
          <input id="input" type="text" placeholder="Type a message and press Enter" aria-label="Message input" autocomplete="off" />
        </div>
        <button id="send" class="send" title="Send message">Send</button>
      </div>
    </section>
  </main>

<script>
(function(){
  const messagesEl = document.getElementById('messages');
  const inputEl = document.getElementById('input');
  const sendBtn = document.getElementById('send');

  // Utility to create message bubble
  function addBubble(text, who='bot', meta=''){
    const el = document.createElement('div');
    el.className = 'bubble ' + who;
    el.textContent = text;
    if(meta){
      const m = document.createElement('div');
      m.className = 'meta';
      m.textContent = meta;
      el.appendChild(m);
    }
    messagesEl.appendChild(el);
    messagesEl.scrollTop = messagesEl.scrollHeight;
    return el;
  }

  // Typing indicator element
  function showTyping(){
    const wrap = document.createElement('div');
    wrap.className = 'bubble bot typing';
    wrap.setAttribute('data-typing','true');
    wrap.innerHTML = '<span class="dot"></span><span class="dot"></span><span class="dot"></span>';
    messagesEl.appendChild(wrap);
    messagesEl.scrollTop = messagesEl.scrollHeight;
    return wrap;
  }

  // Bot reply logic: only "Hi" or "Hello"
  function botReply(){
    // Choose randomly but keep it simple
    const choices = ['Hi', 'Hello'];
    return choices[Math.floor(Math.random()*choices.length)];
  }

  // Send flow
  function sendMessage(){
    const text = inputEl.value.trim();
    if(!text) return;
    // Append user bubble
    addBubble(text, 'user');
    inputEl.value = '';
    inputEl.focus();

    // Show typing
    const typingEl = showTyping();

    // Simulate thinking time (short)
    const delay = 600 + Math.random()*700;
    setTimeout(() => {
      // remove typing
      typingEl.remove();
      // Add bot reply (only Hi/Hello)
      const reply = botReply();
      addBubble(reply, 'bot');
    }, delay);
  }

  // Event listeners
  sendBtn.addEventListener('click', sendMessage);
  inputEl.addEventListener('keydown', function(e){
    if(e.key === 'Enter' && !e.shiftKey){
      e.preventDefault();
      sendMessage();
    }
  });

  // Accessibility: focus input on load
  window.addEventListener('load', () => inputEl.focus());

  // Optional: allow double-click to clear chat
  messagesEl.addEventListener('dblclick', () => {
    if(confirm('Clear chat history?')){
      messagesEl.innerHTML = '';
      addBubble('Hello — this demo bot only replies with Hi or Hello. Try typing anything and press Enter.','bot','Polished dark UI • No network required');
    }
  });

  // Small enhancement: keyboard shortcut Ctrl+K to focus input
  window.addEventListener('keydown', (e) => {
    if((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 'k'){
      e.preventDefault();
      inputEl.focus();
      inputEl.select();
    }
  });
})();
</script>
</body>
</html>
