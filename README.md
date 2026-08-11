# 🤖 AI Personal Assistant — n8n

An AI-powered personal assistant built with **n8n** that can understand natural-language requests and execute real-world tasks across email, calendar, contacts, web research, memory, and voice interactions.

The system is designed around a **modular AI-agent architecture**, where specialized workflows handle specific responsibilities while a central Personal Assistant coordinates the user's requests.

---

## 🚀 Overview

This project demonstrates how to build a practical AI automation system using **n8n AI Agents** and specialized sub-workflows.

The assistant can:

* 📧 Manage emails
* 📅 Create and manage calendar events
* 👤 Search contact information
* 🔎 Perform web research
* 🧠 Maintain conversation context
* 🎙️ Understand voice messages
* 🔊 Respond with voice messages
* 🔄 Delegate tasks to specialized AI agents

The architecture separates responsibilities into independent workflows, making the system easier to maintain, debug, and extend.

---

## 🏗️ Architecture

```text
                    ┌─────────────────────┐
                    │   Telegram User     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Personal Assistant  │
                    │      AI Agent       │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
      │ Email Agent  │ │Calendar Agent│ │Research Agent│
      └──────────────┘ └──────────────┘ └──────────────┘
              │                │                │
              ▼                ▼                ▼
          Email API        Calendar API      Web Research
              
                    ┌─────────────────────┐
                    │  Contact Database   │
                    └─────────────────────┘
                    
                    ┌─────────────────────┐
                    │  Conversation       │
                    │      Memory         │
                    └─────────────────────┘
```

---

## 🧩 Workflows

### 1. Personal Assistant

**File:** `personal-assistant.json`

The main orchestration workflow.

Responsibilities:

* Receive user messages through Telegram
* Detect text or voice input
* Transcribe voice messages
* Maintain conversation memory
* Understand user intent
* Select the appropriate tool
* Delegate tasks to specialized agents
* Return text or voice responses
* Verify tool results before reporting success

---

### 2. Email Agent

**File:** `email-agent.json`

Handles email-related operations through a dedicated workflow.

Capabilities include:

* Send emails
* Search emails
* Read messages
* Reply to emails
* Forward emails
* Summarize email content

The Personal Assistant can delegate email tasks to this specialized agent instead of handling email logic directly.

---

### 3. Calendar Agent

**File:** `calendar-agent.json`

Handles calendar operations.

Capabilities include:

* Create events
* Check availability
* Find events
* Update events
* Reschedule meetings
* Cancel events
* Find meetings with specific people

The workflow is designed to verify calendar availability before creating meetings.

---

### 4. Research Agent

**File:** `research-agent.json`

A dedicated research workflow for external and current information.

It can use research sources such as:

* Google Search
* Wikipedia
* Hacker News

Typical use cases include:

* Current information
* Technology research
* News
* Companies and products
* Technical documentation
* Developer trends
* Fact checking

---

## 🎙️ Voice Automation

The assistant supports voice interactions through Telegram.

### Voice input

```text
Telegram Voice Message
        ↓
Download Audio
        ↓
Speech-to-Text
        ↓
Personal Assistant
        ↓
Tool Execution
```

### Voice output

```text
AI Response
     ↓
Text Processing
     ↓
Text-to-Speech
     ↓
Audio Response
     ↓
Telegram
```

Voice transcription is handled through OpenAI, while voice generation uses ElevenLabs.

---

## 🧠 AI Agent Design

The project uses a **central orchestration agent** with specialized tools.

Instead of creating one large workflow responsible for everything, the system delegates tasks to focused agents.

### Example

```text
User:
"Schedule a meeting with Ahmed tomorrow at 3 PM."

        ↓

Personal Assistant
        ↓
Contact Database
        ↓
Calendar Agent
        ↓
Check Availability
        ↓
Create Event
        ↓
Confirm Result
        ↓
User
```

This architecture makes the system easier to scale by adding new specialized agents without redesigning the entire assistant.

---

## 🛡️ Reliability & Safety

The Personal Assistant follows several execution rules:

* Never invent information
* Never guess email addresses
* Never guess calendar availability
* Never claim an action succeeded without tool confirmation
* Verify current information through connected tools
* Use the minimum required tool
* Preserve requested dates and times
* Ask only for missing required information
* Keep external data separate from conversation memory

This helps prevent common AI-agent problems such as fabricated confirmations and incorrect actions.

---

## 🔄 Tool Selection

The assistant determines which tool is required based on the user's intent.

| User Request                 | Tool                |
| ---------------------------- | ------------------- |
| Find a contact               | Contact Database    |
| Send an email                | Email Agent         |
| Find an email                | Email Agent         |
| Schedule a meeting           | Calendar Agent      |
| Check availability           | Calendar Agent      |
| Research current information | Research Agent      |
| Continue a conversation      | Conversation Memory |

---

## 🛠️ Technology Stack

* **n8n** — Workflow automation & orchestration
* **AI Agents** — Intelligent task routing
* **OpenAI** — Language model & speech transcription
* **Telegram** — User interface
* **Google Sheets** — Contact database
* **Email APIs** — Email automation
* **Calendar APIs** — Calendar automation
* **ElevenLabs** — Text-to-speech
* **SerpApi** — Web search
* **Wikipedia** — Knowledge research
* **Hacker News** — Technology research
* **Conversation Memory** — Context management

---

## 📁 Repository Structure

```text
ai-personal-assistant-n8n/
│
├── personal-assistant.json
├── email-agent.json
├── calendar-agent.json
├── research-agent.json
└── README.md
```

---

## ⚙️ Setup

### 1. Import the workflows

Import the JSON workflow files into your n8n instance.

### 2. Configure credentials

Connect the required credentials for:

* Telegram
* OpenAI
* Google Sheets
* Email provider
* Calendar provider
* ElevenLabs
* Research APIs

### 3. Configure workflow IDs

The Personal Assistant workflow delegates tasks to the specialized workflows.

Update the corresponding workflow references after importing the workflows into your n8n instance.

### 4. Configure the contact database

Create a Google Sheet containing the contact information required by the assistant.

### 5. Activate the workflows

After configuring credentials and workflow references, activate the required workflows and start interacting with the assistant through Telegram.

---

## 🔐 Security Note

API keys, credentials, tokens, and private configuration values should **never be committed to this repository**.

Use n8n's credential management system or environment variables for sensitive information.

---

## 💡 Example Requests

```text
"Send Ahmed an email saying I'll be late."

"Schedule a meeting with Ahmed tomorrow at 3 PM."

"What's on my calendar today?"

"Find Ahmed's email address."

"Research the latest developments in AI agents."

"Summarize this email for me."

"Send me a voice response."
```

---

## 🎯 Project Goals

This project was built to demonstrate practical experience with:

* AI Agent orchestration
* Workflow automation
* Tool calling
* Multi-agent architectures
* API integrations
* Voice AI
* Memory management
* Task delegation
* Reliable AI execution
* Modular n8n architecture

---

## 📌 Future Improvements

Potential extensions include:

* WhatsApp integration
* Slack integration
* Microsoft Outlook support
* More specialized agents
* Long-term user memory
* CRM integration
* Database-backed contact management
* Advanced authentication
* Human approval workflows
* Monitoring and evaluation
* Automated agent testing

---

## 👨‍💻 Author

**Ahmed Gabr Elzanbeel**

AI Automation & n8n Developer

GitHub: [@ahmedelzanbeel](https://github.com/ahmedelzanbeel)

---

## ⭐ Project

If you find this project useful or interesting, consider giving the repository a ⭐.
