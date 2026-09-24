# Arena Auto Chat (v2.4.2)

An experimental Chrome Extension (Manifest V3) for automated interactions with **[Arena.ai](https://arena.ai)**. **Arena Auto Chat** enables a seamless side panel or floating window interface that connects directly to your active Arena browser tab to send prompts, monitor agent thinking and tool usage in real time, answer clarification cards, and capture final responses—all without requiring API keys or external proxies.

---

## 🌟 Key Features

- **Direct Tab Orchestration:** Uses your active, signed-in session on `arena.ai`. No auth tokens, API keys, or credentials required.
- **Real-Time Live Streaming:** Captures live thinking duration (e.g. *"Thought for 12s"*), tool execution labels (e.g. Bash output status), and streaming text output.
- **Clarification Card Support:** Renders interactive option buttons and custom input fields for Arena agent clarification questions directly inside the side panel.
- **Ephemeral Attachment Staging:** Stage images and files (up to 4 files, max 8 MB each) via copy-paste (`Ctrl+V`), drag-and-drop, or file picker. Files remain strictly in memory and are placed into Arena's composer upon sending.
- **Safety & Anti-Bypass Guardrails:** Automatically detects rate limits, CAPTCHA/Cloudflare security challenges, and login prompts in Arena, immediately halting execution (`SECURITY_CHECK`, `RATE_LIMIT`, `SIGN_IN_REQUIRED`) without attempting dangerous bypasses.
- **Automatic Reconnection & Recovery:** If a tab reloads or reconnects, the extension re-attaches to the ongoing task in read-only `WATCH` mode without re-submitting prompts.
- **Modern Glassmorphism UI:** Features light/dark/system themes, customizable text size and accent colors, scroll anchoring, and an optional detached floating window mode.

---

## 🚀 Supported Modes

| Chat Mode | URL Pattern | Status | Notes |
| :--- | :--- | :--- | :--- |
| **Agent Mode** | `https://arena.ai/agent` | ✅ Supported | Full multi-turn agent interaction with tools and question cards |
| **Direct Mode** | `https://arena.ai/text/direct` | ✅ Supported | Single model text chat; model selection via Arena's own `?model_a=` links |
| **Battle / Side-by-Side** | `https://arena.ai/text/` | ❌ Unsupported | Excluded to prevent prompt attribution ambiguity across multiple models |

---

## 🏗️ Architecture & Component Overview

```
                  ┌─────────────────────────────────────────┐
                  │          Chrome Extension Side Panel    │
                  │   (panel.html / panel.js / theme.js)    │
                  └────────────────────┬────────────────────┘
                                       │ chrome.tabs.connect
                                       │ (arena-agent-content-v3)
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │              Content Script             │
                  │   (agent-content.js / agent-dom.js)    │
                  └────────────────────┬────────────────────┘
                                       │
                    ┌──────────────────┴──────────────────┐
                    ▼                                     ▼
      ┌─────────────────────────┐           ┌───────────────────────────┐
      │     DOM Extraction      │           │     Single-Use Staging    │
      │  (Row Matching, Errors, │           │  (stage-main.js in MAIN)  │
      │   Thinking, Tools)      │           │   (via worker.js script)  │
      └─────────────────────────┘           └───────────────────────────┘
```

- **`manifest.json`**: Manifest V3 extension configuration with host permissions limited to `https://arena.ai/*`.
- **`worker.js`**: Background service worker handling side panel registration, floating window lifecycle, tab navigation, and single-use staged file grants.
- **`panel.js` / `panel.html` / `panel.css`**: The main side panel UI application. Manages session state, prompt input, attachment preview, settings, and theme customization.
- **`agent-client.js`**: Transport abstraction managing the `MessagePort` lifecycle between the panel and the active content script.
- **`agent-dom.js`**: DOM manipulation and inspection engine. Handles row parsing, prompt hash verification, error detection, input injection (`execCommand('insertText')`), and tool status extraction.
- **`agent-content.js`**: Injected content script running in Chrome's isolated world on `arena.ai` pages. Coordinates message events and page liveness.
- **`attachment-policy.js` / `attachment.js`**: File validation policy defining supported MIME types, file counts, and size limits.
- **`stage-main.js`**: Injected script operating in the page's `MAIN` execution context to interface with native file input elements during file staging.
- **`conversation-view.js` & `live-view.js`**: Scroller and view-rendering controllers for real-time turn tracking and clarification cards.

---

## 🛠️ Installation & Setup

1. **Clone or Download the Repository:**
   ```bash
   git clone https://github.com/your-username/arena-auto-chat.git
   ```

2. **Load into Google Chrome:**
   - Open Chrome and navigate to `chrome://extensions/`.
   - Enable **Developer mode** in the top right corner.
   - Click **Load unpacked** and select the root directory of this repository.

3. **Connect to Arena:**
   - Open [https://arena.ai](https://arena.ai) in a Chrome tab and sign in.
   - Navigate to **Agent Mode** (`/agent`) or a **Direct** chat (`/text/direct`).
   - Click the extension icon in the toolbar to open the **Arena Auto Chat** side panel.
   - In Settings, select your Arena tab, accept the confirmation checkboxes, and click **Connect & check controls**.

---

## 🔒 Security & Privacy Policy

- **Zero Credential Access:** The extension never inspects, stores, or transmits passwords, session cookies, or authentication tokens.
- **Strict Content Security Policy (CSP):** Configured with `connect-src 'none'`, ensuring that extension pages cannot transmit data to any external server or telemetry service.
- **In-Memory Ephemeral Storage:** Chat history exists strictly in memory during an active panel session. Closing or disconnecting the panel clears all local state. Appearance preferences and window geometries are the only persisted settings.
- **No Manual Fallback or Copy/Paste Prompt Interception:** Prompts are delivered exclusively through native DOM setters and input events (`execCommand('insertText')`), avoiding clipboard tampering.

---

## 📄 License

This repository is provided for research and experimental integration purposes. Refer to [Arena Terms of Use](https://help.arena.ai/articles/5629909088-terms-of-use) regarding automated usage policies.
