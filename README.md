# Claw-Talk

A lightning-fast, mobile-friendly Push-To-Talk (PTT) web interface for [OpenClaw](https://github.com/openclaw/openclaw). It allows you to seamlessly beam voice commands to multiple OpenClaw Gateways (like your desktop, server, or raspberry pi) straight from your phone's browser.

## 🚀 Why This App? (Differentiators)

- **Mobile-First & Extension-Free:** Other solutions are built as heavy desktop Chrome Extensions. This is a pure web app that works flawlessly in iOS Safari and mobile browsers.
- **Zero Backend Required:** No need to stand up complex FastAPI or Node.js proxy servers. This app is 100% static HTML/JS—it connects directly to Azure and directly to your Gateway WebSocket. Host it anywhere (even GitHub Pages) for free!
- **Bypasses iOS Dictation Limits:** We uniquely solve the notorious Apple iOS Safari background dictation timeouts by integrating the raw Microsoft Azure Speech SDK directly into the browser, guaranteeing perfect audio capture every time.
- **Multi-Gateway Switching:** Instantly swap between multiple OpenClaw Gateways (e.g. your desktop, a VPS, or a Raspberry Pi) with a single tap. One UI to rule all your agents.

## 🌟 How It Works

Apple's iOS Safari heavily restricts native web speech recognition (`webkitSpeechRecognition`) in background/sandboxed contexts. To bypass these strict limitations and guarantee flawless audio capture, this UI integrates the **official Microsoft Azure Speech AI SDK**.

1. **Capture:** When you hold the PTT button, your browser streams raw audio via WebSockets directly to Azure's Cognitive Services.
2. **Transcribe:** Azure processes the audio into highly accurate text in real-time.
3. **Route:** The moment you release the button, the transcribed text is injected into an authenticated JSON-RPC payload and fired securely over Tailscale (or your local network) to the target OpenClaw Gateway using the `chat.send` protocol.

Because the app is 100% static HTML/JS, it can be hosted absolutely anywhere (GitHub Pages, Vercel, or a local Python HTTP server).

## 📋 Prerequisites

- An **OpenClaw Gateway** running (v2026.2 or later).
- **HTTPS connection** (Browsers block microphone access on unencrypted `http://` pages. We recommend using **Tailscale** to easily provision a secure `https://` domain for your network).
- An **Azure Speech API Key** (See Free Tier recommendation below).

## 💸 Recommended Free API Route (Azure)

We highly recommend using **Microsoft Azure's Free Tier (F0)** for Speech Services. 
- It grants you **5 hours of free audio transcription every single month**, completely free. 
- Because voice commands are usually only 2-5 seconds long, 5 hours is practically infinite for this use case.
- You can create a free Azure account, search for "Speech Services", and generate a key without being charged.

## 🚀 Setup & Installation

### 1. Configure the Web App
1. Clone or download this repository.
2. Open `index.html` in any text editor.
3. Locate the Azure configuration block inside the `startRecording` function:
   ```javascript
   var key = "YOUR_AZURE_API_KEY_HERE";
   var region = "eastus"; // e.g., eastus, westus, westeurope
   ```
4. Paste your free Azure Speech key and region.

### 2. Host the UI
Since it's a static file, you can host it easily:
- **Locally:** `python3 -m http.server 8081`
- **Cloud:** Drag and drop the folder into Vercel, Cloudflare Pages, or GitHub Pages.

### 3. Expose via Tailscale (For Mobile/Remote Access)
To access the UI securely from your iPhone/Android:
```bash
# Proxy port 443 to your local Python server on port 8081
sudo tailscale serve --yes --bg --set-path / 127.0.0.1:8081
```
You can now open your Tailscale URL (e.g., `https://your-machine.tailnet.ts.net/`) in Safari.

### 4. Connect to OpenClaw
1. Open the Web UI on your device.
2. Enter your **OpenClaw API Token** (find this by running `openclaw token list` on your server).
3. Go to the **Hosts** tab and add your Gateway URL (e.g., `https://your-machine.tailnet.ts.net/api/`).
   *Note: If you have multiple OpenClaw instances, you can sync them to use the same token by running `openclaw token create --token <YOUR_TOKEN>` on your other machines.*
4. Select your target host, press and hold the microphone button, and speak!

## 🔐 Security
Your API tokens and configuration are stored exclusively in your browser's local `localStorage`. The app communicates directly with Azure and your OpenClaw Gateway; there is no middleman server.
