<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>OmniAI Assistant</title>
  <style>
    body { font-family: sans-serif; background: #f4f4f4; padding: 2rem; }
    #chat { max-width: 600px; margin: auto; background: white; padding: 1rem; border-radius: 8px; }
    .msg { margin: 1rem 0; }
    .user { font-weight: bold; color: #333; }
    .bot { color: #007acc; }
    input, button { padding: 0.5rem; font-size: 1rem; }
  </style>
</head>
<body>
  <div id="chat">
    <h2>🤖 OmniAI Assistant</h2>
    <div id="messages"></div>
    <input id="input" placeholder="Ask me anything..." />
    <button onclick="send()">Send</button>
  </div>

  <script>
    const messages = document.getElementById("messages");
    const input = document.getElementById("input");

    async function send() {
      const text = input.value.trim();
      if (!text) return;
      append("You", text, "user");
      input.value = "";

      if (text.toLowerCase().startsWith("wiki ")) {
        const topic = text.slice(5);
        const summary = await fetch(`https://en.wikipedia.org/api/rest_v1/page/summary/${encodeURIComponent(topic)}`)
          .then(res => res.json())
          .then(data => data.extract || "No summary found.")
          .catch(() => "Error fetching Wikipedia.");
        append("OmniAI", summary, "bot");
        return;
      }

      const response = await fetch("https://api-inference.huggingface.co/models/microsoft/DialoGPT-medium", {
        method: "POST",
        headers: {
          "Authorization": "Bearer YOUR_HUGGINGFACE_API_KEY",  // Replace with your key
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ inputs: { text } })
      }).then(res => res.json());

      const reply = response.generated_text || "Sorry, I couldn't think of a reply.";
      append("OmniAI", reply, "bot");
    }

    function append(sender, text, cls) {
      const div = document.createElement("div");
      div.className = "msg " + cls;
      div.innerHTML = `<strong>${sender}:</strong> ${text}`;
      messages.appendChild(div);
      messages.scrollTop = messages.scrollHeight;
    }
  </script>
</body>
</html>
