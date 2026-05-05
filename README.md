# 🚀 Self-Hosted LLM + n8n — 100% Local, Zero Paid APIs

A fully self-hosted AI chatbot pipeline running **n8n** (workflow automation) and **Ollama** (local LLM) entirely on your PC — no cloud APIs, no paid subscriptions. Connected to a live **Lovable** web app via **ngrok** for production-grade public access.

---

## 📸 Project Overview

| Step | Screenshot |
|------|-----------|
| n8n Running Locally | ![n8n Running Locally](Simple%20Run.JPG) |
| Local LLM (Ollama) | ![Local LLM - Ollama](Local%20LLM.JPG) |
| Ollama Models | ![Ollama Models](Olama%20models.JPG) |
| n8n Workflow Active | ![n8n Workflow Live](LiveRun.JPG) |
| Chatting via n8n | ![Chatting App](ChattingApp.JPG) |
| ngrok Tunnel | ![ngrok Tunnel](ngrok.JPG) |
| Production View | ![Production](production.JPG) |
| Lovable App Chat | ![Lovable App Chat](ChattingLovableAPP.JPG) |

---

## 🏗️ Architecture

```
User (Browser / Lovable App)
        │
        ▼
   ngrok (Public URL → localhost:5678)
        │
        ▼
   n8n Webhook (POST /zakeen)
        │
        ▼
   AI Agent (n8n LangChain Node)
        │
        ▼
   Ollama Chat Model (gemma4:26b — LOCAL)
        │
        ▼
   Respond to Webhook → JSON back to User
```

**Flow:**
1. A user sends a chat message from the Lovable app (or any HTTP client).
2. The request hits the **ngrok** public URL, which tunnels to your local n8n instance.
3. n8n's **Webhook** node receives the POST request at `/zakeen`.
4. The **AI Agent** node passes the message to the **Ollama Chat Model** (`gemma4:26b`).
5. Ollama generates a response **locally on your GPU/CPU** — no external API calls.
6. The **Respond to Webhook** node sends the AI reply back as JSON.

---

## 📋 Step-by-Step Setup

### Step 1: Install & Run n8n Locally

n8n is a free, open-source workflow automation tool. Running it locally gives you the **free forever** plan with full API access and no execution limits.

```bash
# Install n8n globally via npm
npm install -g n8n

# Start n8n from PowerShell / Command Prompt
n8n start
```

n8n will start on `http://localhost:5678`. Open it in your browser and complete the initial setup (create your owner account).

![n8n Running Locally](Simple%20Run.JPG)

> **Why local?** The self-hosted version of n8n is **free forever** — no execution limits, no node limits, and you get a full API key for automation. You own your data completely.

---

### Step 2: Install Ollama & Download the LLM

[Ollama](https://ollama.com) lets you run large language models locally on your PC — no paid APIs needed.

```bash
# Install Ollama (download from https://ollama.com)

# Pull the Gemma 4 27B model
ollama pull gemma4:26b

# Verify it's running
ollama list
```

![Local LLM - Ollama](Local%20LLM.JPG)

![Ollama Models](Olama%20models.JPG)

> **Why local LLM?** Zero API costs, full privacy, works offline. The `gemma4:26b` model runs entirely on your hardware.

---

### Step 3: Build the n8n Workflow

1. Open n8n at `http://localhost:5678`
2. Create a **New Workflow**
3. Add the following nodes:

| Node | Type | Configuration |
|------|------|--------------|
| **Webhook** | `n8n-nodes-base.webhook` | Method: `POST`, Path: `zakeen`, Response Mode: `Response Node`, Allowed Origins: `*` |
| **AI Agent** | `@n8n/n8n-nodes-langchain.agent` | Prompt: `{{ $json.body.message }}` |
| **Ollama Chat Model** | `@n8n/n8n-nodes-langchain.lmChatOllama` | Model: `gemma4:26b`, Credential: Ollama account |
| **Respond to Webhook** | `n8n-nodes-base.respondToWebhook` | (default) |

4. Connect them: **Webhook → AI Agent → Respond to Webhook**, and connect **Ollama Chat Model** to the AI Agent's language model input.
5. **Activate** the workflow (toggle the switch in the top-right).

![n8n Workflow Live](LiveRun.JPG)

The workflow JSON is available in [`Workflow.json`](Workflow.json) — you can import it directly into your n8n instance.

---

### Step 4: Test Locally

Send a test request to your local n8n webhook:

```bash
curl -X POST http://localhost:5678/webhook/zakeen \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello, how are you?"}'
```

You should receive a JSON response from the local LLM.

![Chatting via n8n](ChattingApp.JPG)

---

### Step 5: Expose n8n to the Internet with ngrok

To make your local n8n accessible from the internet (for the Lovable app or any external client), use **ngrok** — a free tunneling tool.

```bash
# Install ngrok (download from https://ngrok.com)

# Authenticate (one-time, free account)
ngrok config add-authtoken YOUR_AUTH_TOKEN

# Tunnel to your local n8n instance
ngrok http 5678
```

ngrok will give you a public URL like `https://xxxx-xx-xx.ngrok-free.app` that forwards all traffic to `localhost:5678`.

![ngrok Tunnel](ngrok.JPG)

> **Why ngrok?** It creates a secure public URL for your local server — no need to configure port forwarding, firewall rules, or buy a domain. The free tier is sufficient for development and demos.

---

### Step 6: Connect to Lovable App for Live Production

[Lovable](https://lovable.dev) is an AI-powered web app builder. Connect it to your n8n webhook via the ngrok URL:

1. In your Lovable app, set the API endpoint to your ngrok URL:
   ```
   https://xxxx-xx-xx.ngrok-free.app/webhook/zakeen
   ```
2. Send POST requests with the user's message in the body:
   ```json
   { "message": "Your question here" }
   ```
3. The Lovable app receives the AI response and displays it in the chat UI.

![Production View](production.JPG)

![Lovable App Chat](ChattingLovableAPP.JPG)

> **How it works end-to-end:** User types in the Lovable app → request goes to ngrok URL → ngrok tunnels to your local n8n → n8n sends message to Ollama → Ollama generates response locally → response flows back through n8n → ngrok → Lovable app displays the answer.

---

## 💰 Cost Breakdown

| Component | Cost |
|-----------|------|
| n8n (self-hosted) | **Free forever** |
| Ollama + Gemma 4 27B | **Free** (runs on your hardware) |
| ngrok (free tier) | **Free** |
| Lovable | **Free tier available** |
| **Total** | **$0/month** |

---

## 📁 Project Files

```
Self-hosted-LLM_and_N8N/
├── Workflow.json              # n8n workflow (import this into your n8n)
├── Simple Run.JPG             # n8n running from PowerShell
├── Local LLM.JPG             # Ollama running locally
├── Olama models.JPG           # Available Ollama models
├── LiveRun.JPG                # n8n workflow active
├── ChattingApp.JPG            # Chat test via n8n
├── ngrok.JPG                  # ngrok tunnel setup
├── production.JPG             # Production deployment view
├── ChattingLovableAPP.JPG     # Live chat via Lovable app
└── README.md                  # This file
```

---

## 🔑 Key Takeaways

- **100% local** — n8n and the LLM run on your machine, no cloud dependency
- **Zero paid APIs** — Ollama replaces OpenAI/Anthropic/etc. entirely
- **Free forever n8n** — self-hosting removes all usage limits
- **Production-ready** — ngrok + Lovable gives you a live, public-facing AI chat app
- **Full privacy** — your data never leaves your machine (except through the ngrok tunnel when chatting)

