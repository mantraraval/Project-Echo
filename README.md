# 🎓 Project Echo

![n8n](https://img.shields.io/badge/Orchestration-n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![OpenAI](https://img.shields.io/badge/AI_Agent-OpenAI_GPT-412991?style=flat-square&logo=openai&logoColor=white)
![Perplexity](https://img.shields.io/badge/Research_Tool-Perplexity_AI-20808D?style=flat-square&logoColor=white)
![TTS](https://img.shields.io/badge/Audio-OpenAI_TTS-412991?style=flat-square&logo=openai&logoColor=white)
![Gmail](https://img.shields.io/badge/Delivery-Gmail_API-D14836?style=flat-square&logo=gmail&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

> **Agentic AI Pipeline for Verified, Multi-Modal Research Synthesis** — A user submits a topic, GPT autonomously researches it via Perplexity, validates the output through guardrails and evaluation, converts it to audio, and delivers `audio.mp3` to the inbox — fully automated.

---

## 📌 Overview

Project Echo is a fully agentic n8n workflow. A user submits a research topic via a form, and the pipeline handles everything after autonomously.

**OpenAI GPT** acts as the agent brain — it reasons, orchestrates, and calls **Perplexity AI** as an integrated research tool to fetch and summarize web-grounded articles. The summary is then passed through a **GPT-powered content moderation layer**, a **quality evaluation sub-chain**, converted to **audio via OpenAI TTS**, and delivered as `audio.mp3` to the user's Gmail inbox — zero manual steps required.

---

## 🖼️ Project Artifacts

### Workflow Architecture
*Full n8n canvas showing every node and connection in the pipeline.*

![Workflow Architecture](./workflow.jpg)

### Email Delivery Output
*Gmail inbox showing the automated "Topic Summary" email with `audio.mp3` attached.*

![Email Output](./email%20output.jpg)

---

## 🎯 Pipeline Overview

```
┌──────────────────────────────────────────────────┐
│  TRIGGER                                         │
│  On Form Submission  /  When Fetching Dataset Row│
└───────────────────────┬──────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│  PHASE 1 — AI Agent                              │
│  • OpenAI GPT    →  agent reasoning backbone     │
│  • Simple Memory →  conversational context       │
│  • Perplexity    →  web retrieval & summarization│
└───────────────────────┬──────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│  PHASE 2 — Guardrails                            │
│  • Classify Text for Violations (OpenAI GPT)     │
│  • Switch — routes on classification result      │
│    ├── CLEAN   → Phase 3                         │
│    └── FLAGGED → Alert email (pipeline stops)    │
└───────────────────────┬──────────────────────────┘
                        │ (clean path only)
                        ▼
┌──────────────────────────────────────────────────┐
│  PHASE 3 — Quality Evaluation                    │
│  • Evaluation  (setOutputs)                      │
│  • Evaluation1 (setMetrics)                      │
│  • OpenAI GPT  scores summary quality            │
└───────────────────────┬──────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│  PHASE 4 — Audio Generation                      │
│  • OpenAI TTS → generates audio.mp3              │
└───────────────────────┬──────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│  PHASE 5 — Gmail Delivery                        │
│  • Gmail → "Topic Summary" email + audio.mp3     │
└──────────────────────────────────────────────────┘
```

---

## 🛠️ Node-by-Node Breakdown

### Phase 1 — AI Agent & Research

| Node | Role | Tool |
|:---|:---|:---|
| **On Form Submission** | Captures user's research topic as the primary trigger | n8n Form Trigger |
| **When Fetching a Dataset Row** | Enables batch topic processing over a dataset | n8n Dataset Trigger |
| **AI Agent** | Orchestrates the research sub-chain; GPT reasons and decides when to call Perplexity | n8n AI Agent Node |
| **OpenAI Chat Model** | Powers the AI Agent's reasoning, planning, and response generation | OpenAI GPT API |
| **Simple Memory** | Maintains conversational context across agent steps | n8n Memory Node |
| **Message a Model in Perplexity** | Called by GPT as a tool — fetches live web articles and returns a grounded summary | Perplexity AI API |

---

### Phase 2 — Content Guardrails

| Node | Role | Tool |
|:---|:---|:---|
| **Classify Text for Violations** | Reviews the agent's output for harmful, inappropriate, or unsafe content | OpenAI GPT API |
| **Switch** | Routes flow based on classification: clean content proceeds, flagged content is blocked | n8n Switch Node |
| **Send a Message1** | Sends an alert email if content is flagged — pipeline stops here | Gmail API |

---

### Phase 3 — Quality Evaluation

| Node | Role | Tool |
|:---|:---|:---|
| **Evaluation** | Captures output data and sets evaluation outputs | n8n Evaluation Node |
| **Evaluation1** | Records final quality metrics for the run | n8n Evaluation Node |
| **OpenAI Chat Model1** | Scores the summary for quality, coherence, and relevance | OpenAI GPT API |

---

### Phase 4 — Audio Generation

| Node | Role | Tool |
|:---|:---|:---|
| **Generate Audio** | Converts the verified, evaluated summary into `audio.mp3` | OpenAI TTS API |

---

### Phase 5 — Gmail Delivery

| Node | Role | Tool |
|:---|:---|:---|
| **Send a Message** | Sends "Topic Summary" email with `audio.mp3` attached to the user | Gmail API |

---

## ⚙️ Tech Stack

| Layer | Technology |
|:---|:---|
| **Orchestration** | n8n (self-hosted or cloud) |
| **AI Agent Backbone** | OpenAI GPT (Chat Completions API) |
| **Research & Retrieval Tool** | Perplexity AI API (called by GPT agent) |
| **Content Moderation** | OpenAI GPT (Classify Text for Violations) |
| **Quality Evaluation** | OpenAI GPT (Evaluation sub-chain) |
| **Audio Generation** | OpenAI TTS API |
| **Email Delivery** | Gmail API via n8n Gmail Node |
| **Agent Memory** | n8n Simple Memory Node |

---

## ⚠️ Limitations

- **Perplexity source quality** depends on web availability for the topic — niche or very recent topics may yield thinner summaries.
- **Guardrails are prompt-based** — they reduce but do not guarantee filtering of all policy-violating content.
- **Evaluation scoring is GPT-assessed** — it reflects model judgment of quality, not ground-truth factual accuracy.
- **TTS output is unreviewed** before delivery — for high-stakes use cases, add a manual approval node between Phase 4 and Phase 5.
- **Flagged path** currently sends an alert and stops — it does not retry or attempt content correction automatically.

---

## 📄 License

This project is licensed under the MIT License. See `LICENSE` for details.

---

<div align="center">
  <sub>Built by <a href="https://github.com/mantraraval">mantraraval</a></sub>
</div>
