<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Dark Chat — Greeting Bot</title>
<style>
  :root{
    --bg:#071018;
    --panel:#0f1720;
    --muted:#9aa6b2;
    --accent1:#7c5cff;
    --accent2:#00d4ff;
    --glass: rgba(255,255,255,0.03);
    --radius:16px;
    --gap:14px;
    font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  *{box-sizing:border-box}
  html,body{height:100%}
  body{
    margin:0;
    background:
      radial-gradient(600px 300px at 10% 10%, rgba(124,92,255,0.06), transparent 8%),
      radial-gradient(500px 260px at 90% 90%, rgba(0,212,255,0.03), transparent 8%),
      var(--bg);
    color:#e6eef6;
    -webkit-font-smoothing:antialiased;
    display:flex;
    align-items:center;
    justify-content:center;
    padding:28px;
  }

  .app {
    width:100%;
    max-width:980px;
    height:84vh;
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border-radius:20px;
    box-shadow: 0 18px 50px rgba(2,6,23,0.7), inset 0 1px 0 rgba(255,255,255,0.02);
    display:grid;
    grid-template-columns: 320px 1fr;
    overflow:hidden;
  }

  /* Left column (controls / chats) */
  .left {
    padding:20px;
    border-right:1px solid rgba(255,255,255,0.03);
    background: linear-gradient(180deg, rgba(255,255,255,0.01), transparent);
    display:flex;
    flex-direction:column;
    gap:12px;
  }
  .brand {
    display:flex;
    gap:12px;
    align-items:center;
  }
  .logo {
    width:52px;height:52px;border-radius:12px;
    background: linear-gradient(135deg,var(--accent1),var(--accent2));
    display:flex;align-items:center;justify-content:center;font-weight:700;color:white;
    box-shadow: 0 8px 30px rgba(124,92,255,0.12);
    font-size:18px;
  }
  .brand h1{margin:0;font-size:18px}
  .brand p{margin:0;color:var(--muted);font-size:13px}

  .conversations {
    margin-top:6px;
    display:flex;
    flex-direction:column;
    gap:8px;
    overflow:auto;
    padding-right:6px;
  }
  .conv {
    background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
    padding:12px;
    border-radius:12px;
    color:var(--muted);
    font-size:13px;
    cursor:pointer;
    transition:all .16s ease;
    display:flex;
    align-items:center;
    gap:10px;
  }
  .conv:hover{transform:translateY(-4px); box-shadow: 0 10px 30px rgba(2,6,23,0.45)}
  .conv.active{background:linear-gradient(90deg, rgba(124,92,255,0.10), rgba(0,212,255,0.03)); color:#fff}

  .conv .mini {
    width:36px;height:36px;border-radius:8px;background:rgba(255,255,255,0.02);display:flex;align-items:center;justify-content:center;font-weight:600;
  }

  .left .footer{
    margin-top:auto;
    font-size:12px;
    color:var(--muted);
    display:flex;
    gap:8px;
    align-items:center;
    justify-content:space-between;
  }

  /* Right column (chat) */
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
  .avatar {
    width:56px;height:56px;border-radius:12px;background:linear-gradient(135deg,var(--accent1),var(--accent2));
    display:flex;align-items:center;justify-content:center;font-weight:700;color:white;font-size:20px;
    box-shadow: 0 10px 30px rgba(124,92,255,0.12);
  }
  .chat-title h2{margin:0;font-size:18px}
  .chat-title p{margin:0;color:var(--muted);font-size:13px}

  .messages{
    flex:1;
    overflow:auto;
    padding:8px;
    display:flex;
    flex-direction:column;
    gap:14px;
    scroll-behavior:smooth;
  }

  .row{display:flex;gap:10px;align-items:flex-end}
  .bubble{
    max-width:74%;
    padding:12px 14px;
    border-radius:14px;
    line-height:1.35;
    font-size:15px;
    box-shadow: 0 8px 30px rgba(2,6,23,0.45);
    word-break:break-word;
    position:relative;
  }
  .bubble.user{
    margin-left:auto;
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    color:#e6eef6;
    border-bottom-right-radius:6px;
    border:1px solid rgba(255,255,255,0.02);
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
    margin-top:8px;
  }

  .time {
    font-size:11px;color:var(--muted);margin-left:8px;
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

  /* Composer */
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
    padding:12px;
    border-radius:12px;
    border:1px solid rgba(255,255,255,0.02);
  }
  .input input{
    background:transparent;
    border:0;
    outline:0;
    color:#e6eef6;
    font-size:15px;
    width:100%;
  }
  .send{
    background:linear-gradient(90deg,var(--accent1),var(--accent2));
    border:0;
    color:white;
    padding:10px 14px;
    border-radius:10px;
    cursor:pointer;
    font-weight:700;
    box-shadow: 0 10px 30px rgba(124,92,255,0.12);
    transition:transform .12s ease, opacity .12s ease;
  }
  .send:active{transform:translateY(1px)}
  .send:disabled{opacity:.5;cursor:not-allowed}

  /* small screens */
  @media (max-width:880px){
    .app{grid-template-columns:1fr; height:92vh}
    .left{display:none}
    .chat{padding:16px}
  }
</style>
</head>
<body>
  <main class="app" role="application" aria-label="Chat">
    <aside class="left" aria-hidden="false">
      <div class="brand">
        <div class="logo">DC</div>
        <div>
          <h1>Dark Chat</h1>
          <p style="opacity:.9">A refined local greeting demo</p>
        </div>
      </div>

      <div class="conversations" id="conversations" aria-hidden="false">
        <div class="conv active" title="New conversation">
          <div class="mini">1</div>
          <div>
            <div style="font-weight:600">Conversation</div>
            <div style="font-size:12px;color:var(--muted)">Say hello to begin</div>
          </div>
        </div>
      </div>

      <div class="footer">
        <div style="color:var(--muted);font-size:13px">Made for quick local demos</div>
        <div style="font-size:12px;color:var(--muted)">v2.0</div>
      </div>
    </aside>

    <section class="chat" aria-live="polite">
      <header class="chat-header">
        <div class="chat-title">
          <div class="avatar">B</div>
          <div>
            <h2>Dark Chat</h2>
            <p style="opacity:.9">A small, elegant local responder</p>
          </div>
        </div>
        <div style="color:var(--muted);font-size:13px">No network required</div>
      </header>

      <div class="messages" id="messages" role="log" aria-live="polite" aria-atomic="false">
        <div class="row">
          <div class="bubble bot" id="welcome">
            <strong style="display:block;margin-bottom:8px">Welcome.</strong>
            This is a compact, local chat interface. Try sending a greeting — for example: <em>hi</em>, <em>hello</em>, <em>hey</em>, or <em>good morning</em>. The responder will reply only to greetings; other messages are kept private to your browser.
            <div class="meta">Tip: press <strong>Enter</strong> to send. Double‑click the message area to reset the conversation.</div>
          </div>
        </div>
      </div>

      <div class="composer" aria-hidden="false">
        <div class="input" role="search">
          <input id="input" type="text" placeholder="Say something (try a greeting)" aria-label="Message input" autocomplete="off" />
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

  // Greeting detection (simple, case-insensitive)
  const greetingPatterns = [
    /^hi\b/i,
    /^hello\b/i,
    /^hey\b/i,
    /\bgood (morning|afternoon|evening)\b/i,
    /\bgreetings\b/i,
    /^yo\b/i,
    /^sup\b/i
  ];

  function isGreeting(text){
    if(!text) return false;
    const t = text.trim();
    return greetingPatterns.some(rx => rx.test(t));
  }

  function nowTime(){
    const d = new Date();
    return d.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
  }

  function addBubble(text, who='bot', opts = {}){
    const row = document.createElement('div');
    row.className = 'row';
    const bubble = document.createElement('div');
    bubble.className = 'bubble ' + who;
    bubble.innerHTML = text;
    row.appendChild(bubble);

    const time = document.createElement('div');
    time.className = 'time';
    time.textContent = nowTime();
    row.appendChild(time);

    messagesEl.appendChild(row);
    messagesEl.scrollTop = messagesEl.scrollHeight;
    return bubble;
  }

  function showTyping(){
    const wrap = document.createElement('div');
    wrap.className = 'row';
    const typing = document.createElement('div');
    typing.className = 'bubble bot typing';
    typing.setAttribute('data-typing','true');
    typing.innerHTML = '<span class="dot"></span><span class="dot"></span><span class="dot"></span>';
    wrap.appendChild(typing);
    const time = document.createElement('div');
    time.className = 'time';
    time.textContent = nowTime();
    wrap.appendChild(time);
    messagesEl.appendChild(wrap);
    messagesEl.scrollTop = messagesEl.scrollHeight;
    return wrap;
  }

  // Polished replies: varied greetings and small friendly variants
  const replies = [
    'Hi.',
    'Hello.',
    'Hey there.',
    'Hello — nice to see you.',
    'Hi — hope you are well.',
    'Hey!'
  ];

  function chooseReply(){
    return replies[Math.floor(Math.random()*replies.length)];
  }

  function sendMessage(){
    const text = inputEl.value.trim();
    if(!text) return;
    // Add user bubble
    addBubble(escapeHtml(text), 'user');
    inputEl.value = '';
    inputEl.focus();

    // If it's a greeting, bot replies; otherwise remain silent
    if(isGreeting(text)){
      const typingRow = showTyping();
      const delay = 500 + Math.random()*700;
      setTimeout(() => {
        typingRow.remove();
        addBubble(escapeHtml(chooseReply()), 'bot');
      }, delay);
    } else {
      // No reply: subtle UX cue (no bot message). Optionally, we could show a tiny muted hint.
      // For now, do nothing so the bot stays silent.
    }
  }

  // Escape HTML to avoid injection
  function escapeHtml(str){
    return str.replace(/[&<>"']/g, function(m){ return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]; });
  }

  // Event listeners
  sendBtn.addEventListener('click', sendMessage);
  inputEl.addEventListener('keydown', function(e){
    if(e.key === 'Enter' && !e.shiftKey){
      e.preventDefault();
      sendMessage();
    }
  });

  // Focus input on load
  window.addEventListener('load', () => inputEl.focus());

  // Double-click to clear chat and restore welcome
  messagesEl.addEventListener('dblclick', () => {
    if(confirm('Clear conversation?')){
      messagesEl.innerHTML = '';
      addBubble('<strong style="display:block;margin-bottom:8px">Welcome.</strong> This is a compact, local chat interface. Try sending a greeting — for example: <em>hi</em>, <em>hello</em>, <em>hey</em>, or <em>good morning</em>. The responder will reply only to greetings; other messages are kept private to your browser.<div class="meta">Tip: press <strong>Enter</strong> to send. Double‑click the message area to reset the conversation.</div>', 'bot');
    }
  });

  // Ctrl/Cmd+K to focus input
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
