<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Dark Chat — Local Smart Responder</title>
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
  html,body{height:100%;margin:0}
  body{
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

  /* App container */
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

  /* Left column */
  .left {
    padding:20px;
    border-right:1px solid rgba(255,255,255,0.03);
    background: linear-gradient(180deg, rgba(255,255,255,0.01), transparent);
    display:flex;
    flex-direction:column;
    gap:12px;
  }
  .brand { display:flex; gap:12px; align-items:center; }
  .logo {
    width:52px;height:52px;border-radius:12px;
    background: linear-gradient(135deg,var(--accent1),var(--accent2));
    display:flex;align-items:center;justify-content:center;font-weight:700;color:white;
    box-shadow: 0 8px 30px rgba(124,92,255,0.12);
    font-size:18px;
  }
  .brand h1{margin:0;font-size:18px}
  .brand p{margin:0;color:var(--muted);font-size:13px}

  .conversations { margin-top:6px; display:flex; flex-direction:column; gap:8px; overflow:auto; padding-right:6px; }
  .conv {
    background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
    padding:12px; border-radius:12px; color:var(--muted); font-size:13px; cursor:pointer;
    transition:all .16s ease; display:flex; align-items:center; gap:10px;
  }
  .conv:hover{transform:translateY(-4px); box-shadow: 0 10px 30px rgba(2,6,23,0.45)}
  .conv.active{background:linear-gradient(90deg, rgba(124,92,255,0.10), rgba(0,212,255,0.03)); color:#fff}
  .conv .mini { width:36px;height:36px;border-radius:8px;background:rgba(255,255,255,0.02);display:flex;align-items:center;justify-content:center;font-weight:600; }

  .left .footer{ margin-top:auto; font-size:12px; color:var(--muted); display:flex; gap:8px; align-items:center; justify-content:space-between; }

  /* Right column (chat) */
  .chat { display:flex; flex-direction:column; padding:22px; gap:12px; min-width:0; }
  .chat-header{ display:flex; align-items:center; justify-content:space-between; gap:12px; }
  .chat-title{ display:flex; gap:12px; align-items:center; }
  .avatar { width:56px;height:56px;border-radius:12px;background:linear-gradient(135deg,var(--accent1),var(--accent2)); display:flex;align-items:center;justify-content:center;font-weight:700;color:white;font-size:20px; box-shadow: 0 10px 30px rgba(124,92,255,0.12); }
  .chat-title h2{margin:0;font-size:18px}
  .chat-title p{margin:0;color:var(--muted);font-size:13px}

  /* Messages area: fixed, scrollable (won't expand the page) */
  .messages {
    flex:1;
    min-height:0; /* critical so flexbox allows scrolling */
    overflow-y:auto;
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
    white-space:pre-wrap;
  }
  .bubble.user{ margin-left:auto; background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); color:#e6eef6; border-bottom-right-radius:6px; border:1px solid rgba(255,255,255,0.02); }
  .bubble.bot{ margin-right:auto; background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00)); border: 1px solid rgba(255,255,255,0.02); color:var(--muted); border-bottom-left-radius:6px; }

  .meta{ font-size:12px; color:var(--muted); margin-top:8px; }
  .time { font-size:11px;color:var(--muted);margin-left:8px; }

  .typing{ display:inline-flex; gap:6px; align-items:center; padding:8px 10px; border-radius:12px; background: rgba(255,255,255,0.02); }
  .dot{ width:8px;height:8px;border-radius:50%;background:var(--muted);opacity:0.9; animation: blink 1s infinite; }
  .dot:nth-child(2){animation-delay:.15s}
  .dot:nth-child(3){animation-delay:.3s}
  @keyframes blink{0%{opacity:.15}50%{opacity:1}100%{opacity:.15}}

  /* Composer */
  .composer{ display:flex; gap:10px; align-items:center; padding-top:8px; border-top:1px solid rgba(255,255,255,0.02); }
  .input{ flex:1; display:flex; gap:8px; align-items:center; background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00)); padding:12px; border-radius:12px; border:1px solid rgba(255,255,255,0.02); }
  .input input{ background:transparent; border:0; outline:0; color:#e6eef6; font-size:15px; width:100%; }
  .send{ background:linear-gradient(90deg,var(--accent1),var(--accent2)); border:0; color:white; padding:10px 14px; border-radius:10px; cursor:pointer; font-weight:700; box-shadow: 0 10px 30px rgba(124,92,255,0.12); transition:transform .12s ease, opacity .12s ease; }
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
          <p style="opacity:.9">Local smart responder</p>
        </div>
      </div>

      <div class="conversations" id="conversations" aria-hidden="false">
        <div class="conv active" title="Conversation">
          <div class="mini">1</div>
          <div>
            <div style="font-weight:600">Conversation</div>
            <div style="font-size:12px;color:var(--muted)">Try: "hi", "write 100 word story about rain", "explain derivative", "calculate 12/3 + 4"</div>
          </div>
        </div>
      </div>

      <div class="footer">
        <div style="color:var(--muted);font-size:13px">Single-file • Offline</div>
        <div style="font-size:12px;color:var(--muted)">v3.0</div>
      </div>
    </aside>

    <section class="chat" aria-live="polite">
      <header class="chat-header">
        <div class="chat-title">
          <div class="avatar">B</div>
          <div>
            <h2>Dark Chat</h2>
            <p style="opacity:.9">A capable local responder — stories, explanations, calculations</p>
          </div>
        </div>
        <div style="color:var(--muted);font-size:13px">No network required</div>
      </header>

      <div class="messages" id="messages" role="log" aria-live="polite" aria-atomic="false">
        <div class="row">
          <div class="bubble bot" id="welcome">
            <strong style="display:block;margin-bottom:8px">Welcome.</strong>
            This local responder is designed to be useful without any external services. It recognizes greetings, can generate short stories of a requested length, explain many common concepts in plain language, and evaluate numeric expressions. Try examples:
            <ul style="margin:8px 0 0 18px;padding:0;color:var(--muted)">
              <li><em>hi</em></li>
              <li><em>write 100 word story about a lighthouse</em></li>
              <li><em>explain derivative</em></li>
              <li><em>calculate (12/3 + 4) * 2</em></li>
            </ul>
            <div class="meta">Tip: press Enter to send. The message area scrolls independently so the window size stays fixed.</div>
          </div>
        </div>
      </div>

      <div class="composer" aria-hidden="false">
        <div class="input" role="search">
          <input id="input" type="text" placeholder="Say something (try a command or a greeting)" aria-label="Message input" autocomplete="off" />
        </div>
        <button id="send" class="send" title="Send message">Send</button>
      </div>
    </section>
  </main>

<script>
/* Local Smart Responder
   - Scrollbar: messages container is scrollable and will not expand the page.
   - Intent handling: greetings, story generation, explain, calculate, short Q&A.
   - Story generator: template-based, aims for requested word count.
   - Calculation: safe numeric evaluation with basic sanitization.
   - All local, single-file, no external calls.
*/

(function(){
  const messagesEl = document.getElementById('messages');
  const inputEl = document.getElementById('input');
  const sendBtn = document.getElementById('send');

  // --- Utilities ---
  function nowTime(){
    const d = new Date();
    return d.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
  }
  function escapeHtml(str){
    return String(str).replace(/[&<>"']/g, function(m){ return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]; });
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

  // --- Simple NLP / intent detection ---
  const greetingPatterns = [/^hi\b/i, /^hello\b/i, /^hey\b/i, /\bgood (morning|afternoon|evening)\b/i, /\bgreetings\b/i, /^yo\b/i, /^sup\b/i];

  function isGreeting(text){
    return greetingPatterns.some(rx => rx.test(text.trim()));
  }

  // Detect "write N word story about X" or "write story about X"
  const storyRegex = /\bwrite\s*(?:a\s*)?(?:(\d{2,4})\s*word\s*)?story(?:\s*about|\s*on|\s*of)?\s*(.*)$/i;

  // Detect "explain X" or "what is X"
  const explainRegex = /^(?:explain|what is|what's|describe)\s+(.+)$/i;

  // Detect "calculate" or pure math expression
  const calcRegex = /^(?:calculate\s+)?(.+)$/i;

  // --- Story generator (template-based) ---
  const nouns = ["lighthouse","river","city","child","traveler","cat","ship","forest","clock","island","window","storm","letter","garden","bridge","mountain","train","teacher","painter","star"];
  const verbs = ["watched","followed","remembered","whispered","chased","built","found","lost","opened","closed","waited","sang","wrote","ran","walked","dreamed","glowed","faded","danced","listened"];
  const adjectives = ["lonely","quiet","ancient","bright","small","vast","gentle","restless","golden","hidden","silent","brave","curious","soft","wild","warm","cold","faint","steady","flickering"];
  const adverbs = ["softly","suddenly","slowly","quietly","eagerly","reluctantly","boldly","gently","calmly","wildly","brightly","faintly"];
  const connectors = ["and","but","so","while","as","then","because","until","when","where"];

  function pick(arr){ return arr[Math.floor(Math.random()*arr.length)]; }

  function sentenceAbout(topic){
    // Build a varied sentence mentioning the topic
    const sType = Math.random();
    if(sType < 0.25){
      return `The ${pick(adjectives)} ${topic} ${pick(verbs)} ${pick(adverbs)}.`;
    } else if(sType < 0.5){
      return `${pick(adjectives).charAt(0).toUpperCase()+pick(adjectives).slice(1)} ${topic} ${pick(verbs)} ${pick(connectors)} the ${pick(nouns)} ${pick(verbs)}.`;
    } else if(sType < 0.8){
      return `There was a ${pick(adjectives)} ${topic} that ${pick(verbs)} ${pick(adverbs)} among the ${pick(nouns)}.`;
    } else {
      return `In the ${pick(nouns)}, the ${topic} ${pick(verbs)} and ${pick(verbs)}.`;
    }
  }

  function generateStory(topic, targetWords){
    topic = topic || pick(nouns);
    // normalize topic to a single noun phrase
    topic = topic.trim().replace(/[.?!]$/,'');
    const sentences = [];
    let words = 0;
    // Opening sentence
    const opening = `Once, a ${pick(adjectives)} ${topic} ${pick(verbs)}.`;
    sentences.push(opening);
    words += opening.split(/\s+/).length;
    // Add sentences until reaching targetWords (with some tolerance)
    let safety = 0;
    while(words < targetWords && safety < 200){
      const s = sentenceAbout(topic);
      sentences.push(s);
      words += s.split(/\s+/).length;
      safety++;
    }
    // If overshot, trim last sentence words to match roughly
    const story = sentences.join(' ');
    // If user asked for exact-ish length, try to trim or pad with short phrases
    const currentWords = story.split(/\s+/).length;
    if(currentWords > targetWords + 10){
      // trim by removing last sentence(s)
      const trimmed = sentences.slice(0, Math.max(1, sentences.length - 1)).join(' ');
      return trimmed;
    }
    return story;
  }

  // --- Simple explanation generator ---
  function explainTopic(topic){
    topic = topic.trim();
    // Very generic, template-based explanation that tries to be helpful.
    const short = `Here's a concise explanation of ${escapeHtml(topic)}:\n\n`;
    const body = [
      `What it is: ${escapeHtml(topic)} refers to ${pick(adjectives)} aspects related to ${escapeHtml(pick(nouns))} and common patterns.`,
      `How it works: Typically, ${escapeHtml(topic)} involves observing inputs, applying rules or relationships, and producing an outcome that can be described step by step.`,
      `Why it matters: Understanding ${escapeHtml(topic)} helps you reason about similar problems and make better decisions when you encounter them.`,
      `Quick example: Imagine a simple case where ${escapeHtml(topic)} is like a ${escapeHtml(pick(nouns))} that ${escapeHtml(pick(verbs))}.`
    ];
    return short + body.join('\n\n');
  }

  // --- Safe numeric evaluation ---
  // Allow only digits, whitespace, parentheses, decimal point, + - * / ^ % and Math functions names we whitelist
  const allowedMathFns = ['sin','cos','tan','abs','sqrt','pow','log','ln','max','min','round','floor','ceil','exp'];
  function safeEvaluate(expr){
    // Remove commas and normalize
    let s = String(expr).replace(/,/g,'').trim();
    // Reject suspicious characters
    if(/[A-Za-z]/.test(s)){
      // allow function names but only from whitelist
      // replace function names with Math.<fn>
      allowedMathFns.forEach(fn => {
        const rx = new RegExp('\\b' + fn + '\\b','gi');
        s = s.replace(rx, 'Math.' + fn);
      });
      // After replacement, if any letters remain, reject
      if(/[A-Za-z]/.test(s)) throw new Error('Expression contains unsupported names.');
    }
    // Disallow characters other than digits, operators, parentheses, dot, Math, spaces
    if(/[^0-9+\-*/^%(). Math]/.test(s)) throw new Error('Expression contains invalid characters.');
    // Replace ^ with ** for exponentiation
    s = s.replace(/\^/g, '**');
    // Evaluate using Function in a sandboxed way
    // Note: still use caution; we sanitized above.
    try {
      // eslint-disable-next-line no-new-func
      const fn = new Function('return (' + s + ')');
      const result = fn();
      if(typeof result === 'number' && isFinite(result)) return result;
      throw new Error('Result is not a finite number.');
    } catch(e){
      throw new Error('Could not evaluate expression.');
    }
  }

  // --- Main reply logic ---
  function chooseGreetingReply(){
    const opts = ['Hi.', 'Hello.', 'Hey there.', 'Hi — nice to see you.', 'Hello — how can I help?'];
    return opts[Math.floor(Math.random()*opts.length)];
  }

  function handleMessage(text){
    const raw = text.trim();
    if(!raw) return;

    // Add user bubble
    addBubble(escapeHtml(raw), 'user');

    // Intent detection
    // 1) Greeting
    if(isGreeting(raw)){
      const t = showTyping();
      setTimeout(()=>{ t.remove(); addBubble(escapeHtml(chooseGreetingReply()), 'bot'); }, 400 + Math.random()*700);
      return;
    }

    // 2) Story request
    const storyMatch = raw.match(storyRegex);
    if(storyMatch){
      const num = storyMatch[1] ? parseInt(storyMatch[1],10) : 100;
      const topic = (storyMatch[2] || '').trim() || pick(nouns);
      const target = Math.max(20, Math.min(2000, num)); // clamp
      const t = showTyping();
      setTimeout(()=>{
        t.remove();
        const story = generateStory(topic, target);
        // Try to approximate requested length: if user asked 100 words, we attempt to reach that
        addBubble(escapeHtml(story), 'bot');
      }, 600 + Math.random()*900);
      return;
    }

    // 3) Explain / what is
    const explainMatch = raw.match(explainRegex);
    if(explainMatch){
      const topic = explainMatch[1].trim();
      const t = showTyping();
      setTimeout(()=>{
        t.remove();
        addBubble(escapeHtml(explainTopic(topic)), 'bot');
      }, 500 + Math.random()*900);
      return;
    }

    // 4) Calculate or evaluate expression
    // If message starts with "calculate" or looks like a math expression (digits/operators)
    const calcCmd = raw.match(/^\s*calculate\s+(.+)$/i);
    const looksLikeMath = /^[0-9\s()+\-*/^%.,]*$/.test(raw) || /^[0-9\s().+\-*/^%]+[=]?$/.test(raw);
    if(calcCmd || looksLikeMath){
      const expr = calcCmd ? calcCmd[1] : raw;
      const t = showTyping();
      setTimeout(()=>{
        try{
          const result = safeEvaluate(expr);
          t.remove();
          addBubble(`<strong>Result:</strong> ${escapeHtml(String(result))}`, 'bot');
        } catch(e){
          t.remove();
          addBubble(`I couldn't evaluate that expression. Try a simpler numeric expression like <em>calculate (12/3 + 4) * 2</em>.`, 'bot');
        }
      }, 400 + Math.random()*700);
      return;
    }

    // 5) Short Q&A / "what is X" fallback (non-exact)
    const whatIs = raw.match(/^(?:what is|what's|who is|who's)\s+(.+)\?*$/i);
    if(whatIs){
      const topic = whatIs[1].trim();
      const t = showTyping();
      setTimeout(()=>{
        t.remove();
        addBubble(escapeHtml(explainTopic(topic)), 'bot');
      }, 500 + Math.random()*900);
      return;
    }

    // 6) If none matched, attempt a helpful fallback: short paraphrase or ask to rephrase
    // But user wanted broad replies; provide a concise, constructive reply.
    const t = showTyping();
    setTimeout(()=>{
      t.remove();
      addBubble(`I didn't detect a specific command. I can:\n\n• Reply to greetings\n• Generate a story: "write 100 word story about rain"\n• Explain a concept: "explain derivative"\n• Calculate: "calculate 12/3 + 4"\n\nTry one of those or rephrase your request.`, 'bot');
    }, 500 + Math.random()*900);
  }

  // --- Event listeners ---
  sendBtn.addEventListener('click', () => {
    handleMessage(inputEl.value);
    inputEl.value = '';
    inputEl.focus();
  });
  inputEl.addEventListener('keydown', function(e){
    if(e.key === 'Enter' && !e.shiftKey){
      e.preventDefault();
      handleMessage(inputEl.value);
      inputEl.value = '';
    }
  });

  // Double-click to clear conversation and restore welcome
  messagesEl.addEventListener('dblclick', () => {
    if(confirm('Clear conversation?')){
      messagesEl.innerHTML = '';
      addBubble('<strong style="display:block;margin-bottom:8px">Welcome.</strong>This local responder recognizes greetings, generates stories, explains concepts, and evaluates numeric expressions. Try: <em>write 100 word story about a lighthouse</em> or <em>calculate (12/3 + 4) * 2</em>.', 'bot');
    }
  });

  // Keyboard shortcut Ctrl/Cmd+K to focus input
  window.addEventListener('keydown', (e) => {
    if((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 'k'){
      e.preventDefault();
      inputEl.focus();
      inputEl.select();
    }
  });

  // Focus input on load
  window.addEventListener('load', () => inputEl.focus());

})();
</script>
</body>
</html>
