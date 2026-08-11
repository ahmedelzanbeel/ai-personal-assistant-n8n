# 🤖 AI Personal Assistant — Multi-Agent n8n System

An AI-powered personal assistant built with **n8n** that understands natural-language requests and executes real-world tasks across email, calendar, contacts, web research, memory, and voice interactions.

The system is designed around a **modular AI-agent architecture**, where specialized sub-agent workflows handle domain-specific responsibilities while a central Personal Assistant coordinates incoming requests and routes tasks appropriately.

---

## 🚀 Overview

This project demonstrates an enterprise-grade AI automation system using **n8n AI Agents** and specialized sub-workflows.

The assistant can:
* 📧 Manage emails (send, read, reply, delete, send & wait for response)
* 📅 Manage calendar events (check availability, create, update, delete)
* 👤 Retrieve contact details via Google Sheets integration
* 🔎 Perform live web research using SerpApi, Wikipedia, and Hacker News
* 🧠 Maintain conversation context across turns using persistent memory
* 🎙️ Process incoming voice notes via OpenAI speech transcription
* 🔊 Deliver voice responses using ElevenLabs text-to-speech
* 🔄 Dynamically route tasks to specialized AI sub-agents

By separating core routing from execution logic, the system maintains clear boundaries, preventing context bloat and making individual capabilities easier to test, debug, and scale.

---

## 🏗️ Architecture

```text
                    ┌─────────────────────┐
                    │    Telegram User    │
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
          Email API        Calendar API     Web Research
              
                    ┌─────────────────────┐
                    │  Contact Database   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Conversation     │
                    │       Memory        │
                    └─────────────────────┘
The system employs a supervisor-worker orchestration pattern. A central Personal Assistant agent receives input from Telegram (voice or text), reads conversational context, queries local tools when appropriate, or delegates specialized domain tasks to downstream sub-agent workflows.🤖 Personal Assistant WorkflowThe Personal Assistant workflow serves as the central interface and task router for the entire multi-agent ecosystem. It handles incoming multi-modal triggers, manages conversational state, evaluates intent, and routes tasks to connected tools or downstream sub-agents.Workflow Components & Execution PathTrigger & Input Handling: The workflow starts with the Telegram Trigger node, which listens for incoming messages. Input is passed to a Switch node to differentiate between voice and text:Voice Path: Voice messages flow through Download voice file to retrieve the audio binary, followed by Transcribe a recording1 using OpenAI Speech-to-Text to convert audio into text.Text Path: Text messages bypass transcription and are structured via the Set text node before reaching the core agent.Core Intelligence & Tools: The central Personal Assistant agent node is powered by an OpenAI Chat Model and maintains state across interaction turns using a Simple Memory node. Attached directly to the agent are four specialized tools:Contact Database (Google Sheets integration for contact queries)emailAgent (Tool calling the specialized Email Agent sub-workflow)Call 'Calendar Agent Demo' (Tool calling the specialized Calendar Agent sub-workflow)Call 'Research Agent Demo' (Tool calling the specialized Research Agent sub-workflow)Multi-Modal Output Routing: Once processing completes, output flows to Switch1 to determine the user's preferred delivery format:Text Delivery: Directly calls Send a text message on Telegram.Voice Synthesis Delivery: Routes to Message a model to synthesize spoken responses, sends the script to ElevenLabs via an HTTP Request node, and delivers the generated voice file using Send an audio file on Telegram.📧 Email AgentThe Email Agent is a dedicated sub-agent workflow designed to execute email operations received from the primary Personal Assistant.Workflow Components & CapabilitiesTrigger & Input Processing: Triggered via the When Executed by Another Workflow node, accepting parameters passed from the main supervisor workflow. The input passes through Replace me with your logic directly into the EmailAgent node.Agent Logic & Tools: The EmailAgent is driven by an OpenAI Chat Model and dynamically executes the appropriate action using five integrated Gmail tools:Send Gmail: Drafting and dispatching new outbound email messagesGet Gmail: Searching and retrieving email threads and inbox detailsDelete Gmail1: Removing specified email messagesReply Gmail2: Sending inline responses to ongoing email conversationsSend and Wait for Response Gmail: Pausing workflow execution until an external recipient repliesOutput Processing: After the email action completes, results are formatted by the Response node and returned back to the calling workflow.📅 Calendar AgentThe Calendar Agent manages scheduling, agenda retrieval, and meeting modifications through dedicated calendar automation tools.Workflow Components & CapabilitiesTrigger & Logic: Initiated by the When Executed by Another Workflow trigger upon receiving a scheduling command from the supervisor agent. Input routes through Replace me with your logic to the core Calendar Agent node.Agent Logic & Tools: Driven by an OpenAI Chat Model, the agent evaluates scheduling requests and selects among four Google Calendar tools:Get events: Fetching existing appointments to check availability and agenda detailsCreate event: Booking new meetings and calendar eventsDelete an event in Google Calendar: Removing cancelled meetings from the scheduleUpdate an event in Google Calendar: Modifying or rescheduling existing appointmentsOutput Formatting: Results pass through an Edit Fields node to ensure clean, structured JSON parameters are returned to the parent workflow.🔎 Research AgentThe Research Agent is a specialized sub-agent workflow built for real-time web research, technology news collection, and factual knowledge retrieval.Workflow Components & CapabilitiesTrigger & Logic: Invoked by When Executed by Another Workflow when the main assistant requires external data. Inputs flow through Replace me with your logic to the Research Agent node.Agent Logic & Tools: The Research Agent utilizes an OpenAI Chat Model to decompose research queries and query three distinct data tools:Get many items in Hacker News: Retrieving tech news, developer trends, and community discussionsWikipedia: Performing factual lookups and encyclopedic reference checksGoogle search in SerpApi: Performing live Google web searches for up-to-date online informationExecution Routing: The agent features status-driven output paths, explicitly routing execution through Success or Try Again manual nodes to handle error recovery and ensure reliable research responses.👤 Contact DatabaseContact discovery is built directly into the main Personal Assistant workflow via the Contact Database tool node.Integration: Connects to a Google Sheets document serving as a lightweight CRM.Functionality: Reads contact information (such as email addresses or phone numbers) when requested by the user or when needed by the EmailAgent / CalendarAgent.Safety: Prevents the LLM from hallucinating recipient contact details by forcing a mandatory lookup before executing communication tasks.🎙️ Voice AutomationThe system supports end-to-end multi-modal voice processing through Telegram:Input Voice PipelineUser sends a Telegram voice note.Telegram Trigger catches the update and routes through Switch (Voice path).Download voice file fetches the raw audio binary.Transcribe a recording1 sends audio to OpenAI Whisper API to convert speech into text.Transcribed text is passed directly into the Personal Assistant agent input.Output Voice PipelinePersonal Assistant generates a textual resolution.Output routes through Switch1 (Audio path).Message a model formats the text for conversational speech.HTTP Request calls the ElevenLabs API endpoint to generate high-quality voice audio.Send an audio file delivers the spoken audio file back to the user on Telegram.🧠 AI Agent ArchitectureThe modular design relies on strict task delegation rather than monolithic prompts:PlaintextUser Request (Telegram)
         │
         ▼
Personal Assistant (Central Router)
         │
         ├─── Query Contacts? ──► Contact Database (Google Sheets)
         │
         ├─── Manage Email? ────► Email Agent Sub-Workflow
         │                            ├── Send / Get / Delete / Reply
         │                            └── Send & Wait for Response
         │
         ├─── Manage Calendar? ──► Calendar Agent Sub-Workflow
         │                            ├── Get / Create Events
         │                            └── Delete / Update Events
         │
         └─── Live Research? ───► Research Agent Sub-Workflow
                                      ├── Google Search (SerpApi)
                                      ├── Wikipedia
                                      └── Hacker News
🛡️ Reliability & SafetyThe system enforces operational guardrails to maintain execution accuracy:No Information Invention: Agents are configured to state missing data rather than hallucinating facts.Deterministic Tool Calls: Email addresses and event details must be confirmed via database or tool outputs prior to execution.Explicit Execution Routing: Sub-workflows include fallback paths (Success / Try Again) to manage API rate limits or missing results gracefully.State Isolation: Sub-agents operate with distinct execution contexts, keeping sub-task tool outputs isolated from main memory.🛠️ Technology StackCategoryTechnology / ServiceUsage in ProjectOrchestrationn8nWorkflow orchestration, AI Agent nodes, tool bindingLLM EngineOpenAILanguage logic (OpenAI Chat Model), Speech TranscriptionInterfaceTelegram Bot APIUser communication (Text & Voice)Voice AIOpenAI Whisper / ElevenLabsSpeech-to-Text transcription & Text-to-Speech synthesisDatabaseGoogle Sheets APIContact information storage and retrievalEmailGmail APIReading, sending, replying, and managing email threadsCalendarGoogle Calendar APIFetching availability, creating, updating, and deleting eventsResearch ToolsSerpApi / Wikipedia / Hacker NewsReal-time web search and knowledge retrieval📁 Repository StructurePlaintextai-personal-assistant-n8n/
├── screenshots/
│   ├── personal-assistant.png
│   ├── email-agent.png
│   ├── calendar-agent.png
│   └── research-agent.png
├── personal-assistant.json
├── email-agent.json
├── calendar-agent.json
├── research-agent.json
└── README.md
⚙️ Setup1. Import WorkflowsImport all four JSON files into your n8n instance:email-agent.jsoncalendar-agent.jsonresearch-agent.jsonpersonal-assistant.json2. Configure CredentialsSet up the required API credentials within n8n:Telegram API (Telegram Trigger & Send nodes)OpenAI API (Chat Models & Audio Transcription)Google OAuth2 / Service Account (Gmail, Google Calendar, Google Sheets)ElevenLabs API (HTTP Request node for TTS)SerpApi Key (Google Search tool)3. Update Sub-Workflow ReferencesIn personal-assistant.json, open the tool nodes for emailAgent, Call 'Calendar Agent Demo', and Call 'Research Agent Demo', then select the imported workflow IDs corresponding to your instance.4. Activate WorkflowsToggle all four workflows to Active. Connect your Telegram bot and start sending text or voice commands.🔐 SecurityCredential Storage: All API keys, tokens, and OAuth secrets must be managed strictly using n8n's internal encrypted credential store.No Secret Exposure: Never commit workflow files containing exposed credentials or private keys to public repositories.💡 Example RequestsPlaintext"Find Ahmed's email address in my contacts."

"Send an email to Ahmed letting him know the project files are ready."

"Check my calendar for tomorrow afternoon and schedule a meeting with Ahmed at 3 PM."

"Research the latest developments in AI agents and give me a summary."

"Send me a voice message with my schedule for today."
🎯 Project GoalsThis project demonstrates practical competence in:Architectural separation of multi-agent AI systems in n8nIntegration of multi-modal inputs/outputs (Voice STT/TTS & Text)Dynamic tool routing and agent task delegationProduction API integrations across communication, scheduling, and search servicesBuilding reliable, guardrailed AI workflows for real-world application📌 Future ImprovementsIntegration with WhatsApp and Slack channelsSupport for Microsoft Outlook and Teams APIsLong-term vector database memory for personal preferencesHuman-in-the-loop approval steps for destructive actions (e.g., deleting emails/events)👨‍💻 AuthorAhmed Gabr ElzanbeelAI Automation & Integration Engineer | AI Agents | n8n | APIs | Workflow Automation
