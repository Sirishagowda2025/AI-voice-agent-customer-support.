# 🤖 AI Voice Calling Agent — Real Estate Lead Qualification

A fully autonomous outbound calling system that reads leads from Google Sheets, places real phone calls via VAPI, conducts natural AI-driven conversations in English/Hindi/Kannada, and logs results back — all without human intervention.

Built for a premium property consulting company. Deployable for any business that handles high-volume outbound calling.

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![VAPI](https://img.shields.io/badge/Voice-VAPI-blue)
![GPT-4o](https://img.shields.io/badge/LLM-GPT--4o-412991)
![ElevenLabs](https://img.shields.io/badge/TTS-ElevenLabs%20Turbo%20v2.5-black)
![Deepgram](https://img.shields.io/badge/STT-Deepgram%20Nova--2-13EF93)
![Make.com](https://img.shields.io/badge/Automation-Make.com-orange)

---

## 🏗️ System Architecture

![System Architecture](System__Architecture.png)

---

## ⚙️ How It Actually Works
```
┌─────────────────────────────────────────────────────────────┐
│  Google Sheets: "AI Cold Caller - Siri"                     │
│  Columns: Name (A) | Phone (B) | Industry (C) | Called (D) │
└────────────────────┬────────────────────────────────────────┘
                     │ Make.com reads up to 10 rows
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  Make.com Automation (6 modules)                            │
│  1. Google Sheets → Filter Rows (up to 10 leads)           │
│  2. Basic Aggregator → collect name, row number, status    │
│  3. Basic Feeder → iterate each lead one by one            │
│  4. Set Variables → inject VAPI API Key + Assistant ID     │
│  5. VAPI → createOutboundPhoneCall (phone from column B)   │
│  6. Google Sheets → update cell E = "Yes" (mark called)   │
└────────────────────┬────────────────────────────────────────┘
                     │ Real phone call placed
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  VAPI Voice Agent: "Sarah"                                  │
│  Brain:  GPT-4o (OpenAI) — maxTokens: 150, temp: 0.7      │
│  Voice:  ElevenLabs Turbo v2.5 — speed: 1x, stability: 0.5│
│  Speech: Deepgram Nova-2 transcriber                        │
│  Max call duration: 5 minutes                               │
│  Language: Auto-detects English / Hindi / Kannada           │
└────────────────────┬────────────────────────────────────────┘
                     │ Webhook fires after call ends
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  Structured JSON summary returned via webhook               │
│  Contains: lead qualification, intent, contact details,     │
│  appointment status, call outcome (Pass/Fail eval)          │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧠 Agent Design — "Sarah"

**AI Stack:**

| Component | Technology | Config |
|-----------|-----------|--------|
| Language Model | GPT-4o (OpenAI) | temp: 0.7, maxTokens: 150 |
| Text-to-Speech | ElevenLabs Turbo v2.5 | speed: 1x, stability: 0.5, similarity: 0.75 |
| Speech-to-Text | Deepgram Nova-2 | language: en (auto-switches) |
| Call Duration | Hard limit | 300 seconds (5 min) |
| First Speaker | Assistant | Sarah speaks first |

**Conversation Flow:**
1. Greet caller, introduce as "Sarah from Reshu & Realty"
2. Collect name → location preference → budget → property type → timeline
3. Qualify lead seriousness
4. Recommend property category based on answers
5. Push for site visit booking or free consultation
6. Collect WhatsApp / email for confirmation
7. Auto language switch if caller responds in Hindi or Kannada
8. End call → fire webhook with structured JSON summary

**Design decisions worth noting:**
- `maxTokens: 150` keeps responses short — no rambling, one question at a time
- `temperature: 0.7` balances natural conversation vs. staying on script
- `firstMessageMode: assistant-speaks-first` removes awkward silence on answer
- `backgroundSound: off` keeps audio clean for transcription accuracy
- `analysisPlan` with PassFail rubric auto-evaluates every call outcome

---

## 📋 Google Sheets Schema

| Column | Field | Description |
|--------|-------|-------------|
| A | Name | Lead's full name |
| B | Phone Number | Number VAPI dials |
| C | Industry | Lead segment / source |
| D | Called | Status flag |
| E | (auto-updated) | Written "Yes" after call placed |

---

## 🔁 Make.com Workflow

![Automation Workflow](automation-workflow.png)

| Step | Module | What it does |
|------|--------|--------------|
| 1 | `google-sheets:filterRows` | Reads Sheet1, ascending order, up to 10 rows |
| 2 | `builtin:BasicAggregator` | Collects name + row number into array |
| 3 | `builtin:BasicFeeder` | Iterates array — one lead per cycle |
| 4 | `util:SetVariables` | Sets VAPI API Key + Assistant ID for the cycle |
| 5 | `vapi:createOutboundPhoneCall` | Places the call using phone from column B |
| 6 | `google-sheets:updateCell` | Writes "Yes" to column E after call triggered |

**Scenario settings:** autoCommit on, maxErrors: 3, zone: eu2.make.com

---

## 📁 Repository Files
```
├── README.md                     # This file
├── System__Architecture.png      # Full system architecture diagram
├── automation-workflow.png       # Make.com scenario screenshot
├── automation-template.json      # ✅ Importable Make.com blueprint
└── configvapi-agent-config.json  # ✅ VAPI agent config (keys sanitised)
```

---

## 🚀 Setup Guide

### Prerequisites
- [VAPI account](https://vapi.ai) with a phone number provisioned
- [Make.com account](https://make.com)
- [ElevenLabs account](https://elevenlabs.io) for a voice ID
- Google account with Sheets access

### Step 1 — Configure the VAPI Agent
1. VAPI Dashboard → Assistants → Import → upload `configvapi-agent-config.json`
2. Replace placeholders:
   - `YOUR_VOICE_ID` → your ElevenLabs voice ID
   - `YOUR_WEBHOOK_URL` → your webhook endpoint
3. Note your **Assistant ID**

### Step 2 — Set Up Google Sheets
Create a sheet with these columns:

| A: Name | B: Phone Number | C: Industry | D: Called | E: (auto) |
|---------|----------------|-------------|-----------|-----------|
| Priya S | +919876543210  | Real Estate | | |

### Step 3 — Import Make.com Scenario
1. Make.com → Create Scenario → Import Blueprint → upload `automation-template.json`
2. Reconnect your Google account
3. In module 4 (Set Variables), enter your real VAPI API Key and Assistant ID
4. In module 5, update the Phone Number ID to match your VAPI number

### Step 4 — Test It
- Add one lead to the sheet
- Run the scenario manually in Make.com
- VAPI places the call
- Column E updates to "Yes" automatically ✅

---

## 💼 Adapt for Your Business

The agent prompt lives in `configvapi-agent-config.json` → `model.messages[0].content`. To repurpose:

- Change business name, agent name, and conversation script
- Update qualification questions for your use case
- Modify Google Sheet columns to match your lead fields
- Point the webhook at your own backend, Airtable, or Notion

Works for: Real Estate · Healthcare · Education · Banking · E-commerce · Recruitment

---

## 🔮 Roadmap

- [ ] Write full call transcript back to Google Sheets via webhook
- [ ] Dynamic filter for `Called = blank`
- [ ] CRM push to HubSpot / Zoho after call completion
- [ ] Inbound call handling with same agent
- [ ] Multi-agent support (different scripts per industry/language)

---

## 👩‍💻 Author

**Sirisha D** — AI & Automation Builder

---

## 📄 License

[MIT License](LICENSE)
