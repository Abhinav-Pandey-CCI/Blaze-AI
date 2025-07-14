# Blaze-AI

<!DOCTYPE html> 
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Blaze Assistant</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0; 
      padding: 0;
    }

    body {
      background: radial-gradient(circle at center, #0f0f0f, #000000);
      font-family: 'Segoe UI', sans-serif;
      color: #00f9ff;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      overflow: hidden;
    }

    h1 {
      font-size: 3em;
      margin-bottom: 30px;
      text-shadow: 0 0 10px #00f9ff, 0 0 20px #00f9ff;
    }

    .glass-panel {
      background: rgba(255, 255, 255, 0.05);
      border: 1px solid rgba(0, 249, 255, 0.2);
      box-shadow: 0 0 30px rgba(0, 249, 255, 0.3);
      border-radius: 20px;
      padding: 30px;
      text-align: center;
      width: 80%;
      max-width: 600px;
      backdrop-filter: blur(10px);
    }

    #start-btn {
      background: #00f9ff;
      border: none;
      color: #000;
      padding: 15px 30px;
      font-size: 18px;
      font-weight: bold;
      border-radius: 50px;
      cursor: pointer;
      box-shadow: 0 0 10px #00f9ff, 0 0 30px #00f9ff;
      transition: 0.3s ease;
    }

    #start-btn:hover {
      transform: scale(1.05);
    }

    #wakeword-label {
      margin: 20px 0;
      font-size: 16px;
    }

    .voice-wave {
      margin: 20px auto;
      width: 100px;
      height: 100px;
      border-radius: 50%;
      background: rgba(0, 249, 255, 0.2);
      box-shadow: 0 0 15px #00f9ff;
      animation: pulse 2s infinite;
    }

    @keyframes pulse {
      0% { transform: scale(1); opacity: 0.8; }
      50% { transform: scale(1.2); opacity: 0.4; }
      100% { transform: scale(1); opacity: 0.8; }
    }

    p {
      margin: 15px 0;
      font-size: 18px;
      color: #ffffff;
      text-shadow: 0 0 5px #00f9ff;
    }

    .typing {
      border-right: 2px solid #00f9ff;
      white-space: nowrap;
      overflow: hidden;
      animation: typing 2s steps(30, end), blink 0.5s step-end infinite alternate;
    }

    @keyframes typing {
      from { width: 0 }
      to { width: 100% }
    }

    @keyframes blink {
      50% { border-color: transparent }
    }
  </style>
</head>
<body>
  <h1>B.L.A.Z.E Interface</h1>
  <div class="glass-panel">
    <div class="voice-wave" id="voice-wave" style="display: none;"></div>
    <button id="start-btn">🎤 Start Listening</button>
    <div id="wakeword-label">
      <label>
        <input type="checkbox" id="wakeword-toggle" checked />
        Use "Blaze" wake word
      </label>
    </div>
    <p id="user-query">You: </p>
    <p id="assistant-reply">B.L.A.Z.E: </p>
  </div>

  <script>
    const startBtn = document.getElementById("start-btn");
    const userQuery = document.getElementById("user-query");
    const assistantReply = document.getElementById("assistant-reply");
    const wakewordToggle = document.getElementById("wakeword-toggle");
    const voiceWave = document.getElementById("voice-wave");

    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    const recognition = new SpeechRecognition();

    recognition.lang = "en-US";
    recognition.continuous = true;
    recognition.interimResults = false;

    let isListening = false;
    let lastTranscript = "";
    let cachedVoices = [];

    window.speechSynthesis.onvoiceschanged = () => {
      cachedVoices = window.speechSynthesis.getVoices();
    };

    window.onload = () => {
      setTimeout(() => {
        speak("Hello, I am Blaze. Press the button to start listening.");
      }, 500);
    };

    startBtn.onclick = () => {
      if (!isListening) {
        recognition.start();
        assistantReply.textContent = "B.L.A.Z.E: Listening...";
        startBtn.textContent = "🛑 Stop Listening";
        voiceWave.style.display = "block";
        isListening = true;
      } else {
        recognition.stop();
        assistantReply.textContent = "B.L.A.Z.E: Stopped.";
        startBtn.textContent = "🎤 Start Listening";
        voiceWave.style.display = "none";
        isListening = false;
      }
    };

    recognition.onresult = (event) => {
      const transcript = event.results[event.results.length - 1][0].transcript.trim().toLowerCase();
      if (transcript === lastTranscript) return;
      lastTranscript = transcript;
      console.log("Heard:", transcript);

      const useWakeWord = wakewordToggle.checked;

      if (useWakeWord) {
        if (transcript.includes("blaze")) {
          const command = transcript.split("blaze")[1]?.trim();
          if (command) {
            userQuery.textContent = "You: " + command;
            getReply(command);
          } else {
            assistantReply.textContent = "B.L.A.Z.E: Yes?";
            speak("Yes?");
          }
        }
      } else {
        userQuery.textContent = "You: " + transcript;
        getReply(transcript);
      }
    };

    recognition.onerror = (event) => {
      console.error("Speech recognition error:", event.error);
      assistantReply.textContent = "Error: " + event.error;
    };

    async function getReply(command) {
      const responses = {
        hello: "Hello, I am Blaze.",
        time: "The time is " + new Date().toLocaleTimeString(),
        date: "Today is " + new Date().toDateString(),
        "your name": "I am Blaze, your virtual assistant.",
        "are you there": "Yes sir, I am here to assist you sir!",
        "open google": "Opening Google.",
        "open youtube": "Opening YouTube.",
      };

      let urlToOpen = null;

      for (let key in responses) {
        if (command.includes(key)) {
          const reply = responses[key];
          assistantReply.innerHTML = `B.L.A.Z.E: <span class="typing">${reply}</span>`;
          speak(reply);

          if (key.includes("google")) urlToOpen = "https://www.google.com";
          if (key.includes("youtube")) urlToOpen = "https://www.youtube.com";
          if (key === "play music") urlToOpen = "https://www.youtube.com/results?search_query=music";

          if (urlToOpen) {
            setTimeout(() => window.open(urlToOpen, "_blank"), 100);
          }

          return;
        }
      }

      if (command.startsWith("search for")) {
        const q = command.replace("search for", "").trim();
        const reply = `Searching Google for ${q}`;
        assistantReply.innerHTML = `B.L.A.Z.E: <span class="typing">${reply}</span>`;
        speak(reply);
        setTimeout(() => {
          window.open(`https://www.google.com/search?q=${encodeURIComponent(q)}`, "_blank");
        }, 100);
        return;
      }

      if (command.startsWith("search youtube for") || command.startsWith("search on youtube")) {
        const q = command.replace("search youtube for", "").replace("search on youtube for", "").trim();
        const reply = `Searching YouTube for ${q}`;
        assistantReply.innerHTML = `B.L.A.Z.E: <span class="typing">${reply}</span>`;
        speak(reply);
        setTimeout(() => {
          window.open(`https://www.youtube.com/results?search_query=${encodeURIComponent(q)}`, "_blank");
        }, 100);
        return;
      }

      assistantReply.innerHTML = `B.L.A.Z.E: <span class="typing">Thinking...</span>`;
      const aiReply = await askOpenAI(command);
      assistantReply.innerHTML = `B.L.A.Z.E: <span class="typing">${aiReply}</span>`;
      speak(aiReply);
    }

    async function askOpenAI(message) {
      try {
        const response = await fetch('http://localhost:3000/chat', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ message }),
        });

        if (!response.ok) throw new Error("Network response not ok");
        const data = await response.json();
        return data.choices[0].message.content.trim();
      } catch (err) {
        console.error("OpenAI API error:", err);
        return "Sorry, I am unable to process that right now.";
      }
    }

    function speak(text) {
      const utter = new SpeechSynthesisUtterance(text);
      const voices = speechSynthesis.getVoices();
      const voice = voices.find(v => v.name.toLowerCase().includes("daniel") || v.lang === "en-US");
      if (voice) utter.voice = voice;
      window.speechSynthesis.speak(utter);
    }
  </script>
</body> 
</html>
