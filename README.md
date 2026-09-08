# AI Customer Support Voice Agent (n8n + ElevenLabs)

An AI-powered voice agent that handles real customer support phone calls — checking calendar availability, booking appointments, sending confirmation emails, and answering FAQs from a knowledge base — built with **n8n**, **ElevenLabs Conversational AI**, and **Groq**.

Deployed and tested live with ElevenLabs' voice agent platform. **100% tool-call success rate across 23 live executions.**

## How it works

```
ElevenLabs Voice Agent (caller conversation)
        │  POST webhook
        ▼
Webhook: Receive User Request
        │
        ▼
   AI Agent (Groq · openai/gpt-oss-20b)
        │
        ├── Simple Memory (per-call session history)
        ├── Calendar: Check Availability   (Google Calendar)
        ├── Calendar: Create Appointment   (Google Calendar)
        ├── Gmail Tool: Send Booking Confirmation
        └── Knowledgebase Lookup           (Google Sheets)
        │
        ▼
Webhook: Return AI Response
        │  JSON response
        ▼
ElevenLabs Voice Agent (spoken back to caller)
```

ElevenLabs' voice agent sends the caller's transcribed message to the n8n webhook. The AI Agent decides — based on the conversation — whether to check calendar availability, book an appointment, look up an FAQ, or just reply directly. The response is sent back to ElevenLabs and read aloud to the caller.

## Features

- **Appointment booking** — checks real Google Calendar availability before booking, and only books after the caller confirms a time.
- **Booking confirmation emails** — automatically emails the caller once an appointment is created (only if they gave an email).
- **FAQ knowledge base** — classifies each question into `Order`, `Delivery`, or `Refund` and looks up the answer from a Google Sheet.
- **Timezone-correct scheduling** — all appointment times are handled in Bangladesh local time (Asia/Dhaka, +06:00), with an explicit rule preventing the model from guessing weekdays.
- **Per-call memory** — keeps conversation context for the duration of a call using the ElevenLabs session ID.

## Tech stack

| Piece | Tool |
|---|---|
| Orchestration | [n8n](https://n8n.io) |
| Voice agent / telephony | [ElevenLabs Conversational AI](https://elevenlabs.io) |
| LLM | Groq (`openai/gpt-oss-20b`) |
| Calendar | Google Calendar API |
| Email | Gmail API |
| Knowledge base | Google Sheets |

## Setup

1. Import `customer-support-voice-agent.json` into your n8n instance.
2. Create credentials for: Groq, Google Calendar (OAuth2), Gmail (OAuth2), Google Sheets (OAuth2), and a Header Auth credential for the webhook.
3. Replace the placeholders in the workflow:
   - `your-calendar-email@gmail.com` → your Google Calendar ID
   - `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` → your knowledge base spreadsheet ID
   - `REPLACE_WITH_YOUR_CREDENTIAL_ID` → auto-filled once you attach your own credentials in the n8n UI
4. Create a Google Sheet with columns `Topic` and an answer column (e.g. `Answer`), with rows for `Order`, `Delivery`, and `Refund`.
5. Publish the workflow and copy the webhook URL + your header-auth secret.
6. In ElevenLabs, add a custom tool that calls your n8n webhook URL with the header auth key, sending `{ "message": "...", "session_id": "..." }`.

> **Security note:** The webhook uses header authentication so only your ElevenLabs agent (with the correct secret) can trigger it. Don't disable authentication on the webhook — an open endpoint could be used by anyone to trigger real calendar bookings and emails through your accounts.

## Author

**Apu Ghosh** — AI Automation Developer
[GitHub](https://github.com/apughosh123) · [LinkedIn](https://www.linkedin.com/in/apu-ghosh-automation/)
