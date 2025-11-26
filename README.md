<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>🧠 Neural CRT Chat</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      background: #0d0d0d;
      color: #ccff00;
      font-family: 'Courier New', monospace;
      height: 100vh;
      overflow: hidden;
    }

    .crt-shell {
      display: flex;
      flex-direction: column;
      height: 100%;
      padding: 20px;
      box-sizing: border-box;
      background-color: #1a1a00;
      border: 12px solid #333;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(102, 255, 0, 0.5), inset 0 0 20px rgba(102, 255, 0, 0.2);
      position: relative;
      overflow: hidden;
    }

    .crt-shell::before {
      content: "";
      position: absolute;
      top: 0; left: 0; right: 0; bottom: 0;
      background: repeating-linear-gradient(
        0deg,
        rgba(26,26,0,0.1),
        rgba(26,26,0,0.1) 2px,
        rgba(0,0,0,0.25) 2px,
        rgba(0,0,0,0.25) 4px
      );
      pointer-events: none;
      z-index: 1;
    }

    .crt-shell::after {
      content: "";
      position: absolute;
      top: 0; left: 0; right: 0; bottom: 0;
      background: radial-gradient(ellipse at center, rgba(102,255,0,0.1) 0%, rgba(102,255,0,0) 70%);
      pointer-events: none;
      z-index: 1;
    }

    .terminal {
      flex: 1;
      overflow-y: auto;
      padding-right: 10px;
      z-index: 2;
      scrollbar-width: thin;
      scrollbar-color: #66ff00 #1a1a00;
    }

    .terminal::-webkit-scrollbar {
      width: 8px;
    }

    .terminal::-webkit-scrollbar-track {
      background: #1a1a00;
    }

    .terminal::-webkit-scrollbar-thumb {
      background-color: #66ff00;
      border-radius: 4px;
    }

    .prompt-line {
      display: flex;
      align-items: flex-start;
      margin-top: 10px;
      line-height: 1.4;
    }

    .prompt {
      color: #66ff00;
      margin-right: 8px;
      font-weight: bold;
      flex-shrink: 0;
    }

    .command {
      color: #ccff66;
      word-break: break-word;
      flex: 1;
    }

    .response {
      margin-top: 5px;
      color: #ccff00;
      line-height: 1.4;
      word-break: break-word;
    }

    .cursor {
      display: inline-block;
      width: 8px;
      height: 16px;
      background-color: #ccff00;
      animation: blink 1s infinite;
      margin-left: 2px;
      vertical-align: middle;
    }

    @keyframes blink {
      0%, 100% { opacity: 1; }
      50% { opacity: 0; }
    }

    input[type="text"] {
      background-color: #000;
      color: #ccff00;
      border: 1px solid #66ff00;
      font-family: 'Courier New', monospace;
      font-size: 16px;
      width: 100%;
      box-sizing: border-box;
      padding: 10px;
      margin-top: 10px;
      z-index: 2;
      outline: none;
    }

    input[type="text"]:focus {
      box-shadow: 0 0 8px rgba(102, 255, 0, 0.7);
    }

    .viewer-container {
      display: flex;
      flex-wrap: wrap;
      gap: 15px;
      margin-top: 15px;
      z-index: 2;
    }

    .viewer {
      flex: 1 1 300px;
      min-width: 300px;
      height: 250px;
      border: 1px solid #66ff00;
      box-shadow: 0 0 10px rgba(102, 255, 0, 0.3);
      background: #000;
      overflow: hidden;
      position: relative;
    }

    .viewer-header {
      background: rgba(102, 255, 0, 0.1);
      padding: 5px 10px;
      font-size: 14px;
      border-bottom: 1px solid #66ff00;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .viewer-close {
      background: none;
      border: none;
      color: #ccff00;
      cursor: pointer;
      font-size: 16px;
    }

    .viewer-content {
      width: 100%;
      height: calc(100% - 30px);
      border: none;
      background: #000;
    }

    .help-text {
      margin-top: 10px;
      font-size: 14px;
      color: #99cc00;
      padding: 10px;
      background: rgba(0, 0, 0, 0.3);
      border-radius: 4px;
    }

    .typing-indicator {
      display: inline-block;
      margin-left: 5px;
    }

    .typing-dot {
      display: inline-block;
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: #ccff00;
      margin-right: 3px;
      animation: typing 1.4s infinite ease-in-out;
    }

    .typing-dot:nth-child(1) { animation-delay: 0s; }
    .typing-dot:nth-child(2) { animation-delay: 0.2s; }
    .typing-dot:nth-child(3) { animation-delay: 0.4s; }

    @keyframes typing {
      0%, 60%, 100% { transform: translateY(0); }
      30% { transform: translateY(-5px); }
    }

    .command-history {
      color: #99cc00;
      font-size: 14px;
      margin-top: 5px;
    }

    .welcome-message {
      margin-bottom: 15px;
      padding: 10px;
      background: rgba(0, 0, 0, 0.3);
      border-radius: 4px;
      border-left: 3px solid #66ff00;
    }

    .error-message {
      color: #ff6666;
    }

    .success-message {
      color: #66ff66;
    }

    .info-message {
      color: #66ccff;
    }

    .loading-message {
      color: #ffcc00;
    }

    .result-title {
      font-weight: bold;
      margin-top: 10px;
      color: #66ff00;
    }

    .result-extract {
      margin-left: 15px;
      margin-bottom: 10px;
    }

    .source-link {
      color: #66ccff;
      text-decoration: none;
      font-size: 14px;
    }

    .source-link:hover {
      text-decoration: underline;
    }
  </style>
</head>
<body>
  <div class="crt-shell" onclick="document.getElementById('input').focus()">
    <div class="terminal" id="terminal">
      <div class="welcome-message">
        <h2>🧠 Neural CRT Chat v2.1</h2>
        <p>Enter commands to search knowledge sources. Type <code>help</code> for available commands.</p>
      </div>
      <div class="prompt-line">
        <span class="prompt">$</span>
        <span id="command-line"></span>
        <span class="cursor"></span>
      </div>
    </div>
    <div class="command-history" id="command-history"></div>
    <input type="text" id="input" placeholder="Type a command (e.g., search: quantum computing)" autofocus />
    <div class="help-text">
      Commands: <code>search:</code>, <code>lookup</code>, <code>find</code>, <code>help</code>, <code>clear</code>
    </div>
  </div>

  <script>
    const terminal = document.getElementById("terminal");
    const input = document.getElementById("input");
    const commandLine = document.getElementById("command-line");
    const commandHistory = document.getElementById("command-history");
    let commandHistoryList = [];
    let historyIndex = -1;

    input.addEventListener("input", () => {
      commandLine.textContent = input.value;
    });

    input.addEventListener("keydown", (e) => {
      if (e.key === "Enter") {
        const command = input.value.trim();
        if (command) {
          executeCommand(command);
          addToHistory(command);
        }
      } else if (e.key === "ArrowUp") {
        e.preventDefault();
        navigateHistory(-1);
      } else if (e.key === "ArrowDown") {
        e.preventDefault();
        navigateHistory(1);
      }
    });

    function addToHistory(command) {
      commandHistoryList.push(command);
      historyIndex = commandHistoryList.length;
      updateHistoryDisplay();
    }

    function navigateHistory(direction) {
      if (commandHistoryList.length === 0) return;
      
      historyIndex += direction;
      
      if (historyIndex < 0) {
        historyIndex = 0;
      } else if (historyIndex >= commandHistoryList.length) {
        historyIndex = commandHistoryList.length;
        input.value = "";
        commandLine.textContent = "";
        return;
      }
      
      input.value = commandHistoryList[historyIndex];
      commandLine.textContent = input.value;
      input.setSelectionRange(input.value.length, input.value.length);
    }

    function updateHistoryDisplay() {
      if (commandHistoryList.length === 0) {
        commandHistory.textContent = "";
        return;
      }
      
      const lastCommands = commandHistoryList.slice(-3);
      commandHistory.textContent = "Last commands: " + lastCommands.join(", ");
    }

    function executeCommand(command) {
      // Echo the command
      const echo = document.createElement("div");
      echo.className = "prompt-line";
      echo.innerHTML = `<span class="prompt">$</span> <span class="command">${escapeHtml(command)}</span>`;
      terminal.appendChild(echo);

      // Process command
      if (command.toLowerCase() === "help") {
        showHelp();
      } else if (command.toLowerCase() === "clear") {
        clearTerminal();
      } else if (command.match(/^search:|^lookup |^find /i)) {
        const query = command.replace(/^search:|^lookup |^find /i, "").trim();
        if (query) {
          performSearch(query);
        } else {
          addResponse("❌ Please provide a search term.", "error-message");
        }
      } else {
        addResponse(`❌ Unknown command: ${escapeHtml(command)}`, "error-message");
        addResponse("Type 'help' for available commands.", "response");
      }

      // Clear input
      input.value = "";
      commandLine.textContent = "";
      terminal.scrollTop = terminal.scrollHeight;
    }

    function showHelp() {
      addResponse("Available commands:", "response");
      addResponse("  search: <query> - Search knowledge sources", "response");
      addResponse("  lookup <query> - Lookup information", "response");
      addResponse("  find <query> - Find relevant data", "response");
      addResponse("  clear - Clear the terminal", "response");
      addResponse("  help - Show this help message", "response");
    }

    function clearTerminal() {
      terminal.innerHTML = `
        <div class="welcome-message">
          <h2>🧠 Neural CRT Chat v2.1</h2>
          <p>Enter commands to search knowledge sources. Type <code>help</code> for available commands.</p>
        </div>
        <div class="prompt-line">
          <span class="prompt">$</span>
          <span id="command-line"></span>
          <span class="cursor"></span>
        </div>
      `;
      document.getElementById("command-line").textContent = "";
    }

    async function performSearch(query) {
      addResponse("🤖 Initializing Neural CRT Chat...", "info-message");
      addResponse("🧠 Analyzing query...", "info-message");
      addResponse(`🔍 Searching for: ${escapeHtml(query)}`, "info-message");
      
      // Show typing indicator
      const typingIndicator = document.createElement("div");
      typingIndicator.className = "response";
      typingIndicator.innerHTML = `🧠 Processing results<span class="typing-indicator"><span class="typing-dot"></span><span class="typing-dot"></span><span class="typing-dot"></span></span>`;
      terminal.appendChild(typingIndicator);
      terminal.scrollTop = terminal.scrollHeight;
      
      try {
        // Fetch Wikipedia data
        const wikiData = await fetchWikipedia(query);
        
        // Remove typing indicator
        typingIndicator.remove();
        
        if (wikiData) {
          addResponse(`<div class="result-title">🧠 Wikipedia Result:</div>`, "success-message");
          addResponse(`<div class="result-extract">${escapeHtml(wikiData.extract)}</div>`, "response");
          addResponse(`<a href="${wikiData.url}" target="_blank" class="source-link">🔗 View on Wikipedia</a>`, "response");
        }
        
        // Fetch DuckDuckGo data
        const duckData = await fetchDuckDuckGo(query);
        
        if (duckData) {
          addResponse(`<div class="result-title">🦆 DuckDuckGo Result:</div>`, "success-message");
          addResponse(`<div class="result-extract">${escapeHtml(duckData.abstract)}</div>`, "response");
          addResponse(`<a href="${duckData.url}" target="_blank" class="source-link">🔗 View on DuckDuckGo</a>`, "response");
        }
        
        if (!wikiData && !duckData) {
          addResponse("🤷 No results found for your query.", "error-message");
        }
        
        // Inject viewers
        injectViewers(query);
      } catch (error) {
        typingIndicator.remove();
        addResponse(`❌ Error during search: ${error.message}`, "error-message");
      }
    }

    async function fetchWikipedia(query) {
      const proxyUrl = "https://api.allorigins.win/raw?url=";
      const wikiUrl = `https://en.wikipedia.org/api/rest_v1/page/summary/${encodeURIComponent(query)}`;
      
      try {
        const response = await fetch(proxyUrl + encodeURIComponent(wikiUrl));
        if (!response.ok) throw new Error(`Wikipedia API error: ${response.status}`);
        
        const data = await response.json();
        if (data.extract) {
          return {
            extract: data.extract,
            url: `https://en.wikipedia.org/wiki/${encodeURIComponent(query)}`
          };
        }
        return null;
      } catch (error) {
        console.error("Wikipedia fetch error:", error);
        return null;
      }
    }

    async function fetchDuckDuckGo(query) {
      const proxyUrl = "https://api.allorigins.win/raw?url=";
      const duckUrl = `https://api.duckduckgo.com/?q=${encodeURIComponent(query)}&format=json&no_html=1&skip_disambig=1`;
      
      try {
        const response = await fetch(proxyUrl + encodeURIComponent(duckUrl));
        if (!response.ok) throw new Error(`DuckDuckGo API error: ${response.status}`);
        
        const data = await response.json();
        if (data.Abstract) {
          return {
            abstract: data.Abstract,
            url: data.AbstractURL || `https://duckduckgo.com/?q=${encodeURIComponent(query)}`
          };
        }
        return null;
      } catch (error) {
        console.error("DuckDuckGo fetch error:", error);
        return null;
      }
    }

    function injectViewers(query) {
      const viewerContainer = document.createElement("div");
      viewerContainer.className = "viewer-container";
      
      const sources = [
        { name: "Wikipedia", url: `https://en.wikipedia.org/wiki/Special:Search?search=${encodeURIComponent(query)}` },
        { name: "Internet Archive", url: `https://archive.org/search.php?query=${encodeURIComponent(query)}` },
        { name: "Open Library", url: `https://openlibrary.org/search?q=${encodeURIComponent(query)}` }
      ];
      
      sources.forEach(source => {
        const viewer = document.createElement("div");
        viewer.className = "viewer";
        
        const header = document.createElement("div");
        header.className = "viewer-header";
        header.innerHTML = `
          <span>${escapeHtml(source.name)}</span>
          <button class="viewer-close" onclick="this.closest('.viewer').remove()">×</button>
        `;
        
        const iframe = document.createElement("iframe");
        iframe.className = "viewer-content";
        iframe.src = source.url;
        iframe.sandbox = "allow-scripts allow-same-origin";
        
        viewer.appendChild(header);
        viewer.appendChild(iframe);
        viewerContainer.appendChild(viewer);
      });
      
      terminal.appendChild(viewerContainer);
    }

    function addResponse(text, className = "response") {
      const reply = document.createElement("div");
      reply.className = className;
      reply.innerHTML = text;
      terminal.appendChild(reply);
      terminal.scrollTop = terminal.scrollHeight;
    }

    function escapeHtml(unsafe) {
      return unsafe
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
    }

    // Initial focus
    window.addEventListener("load", () => {
      input.focus();
    });
  </script>
</body>
</html>
