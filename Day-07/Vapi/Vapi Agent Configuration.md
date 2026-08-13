# Vapi Voice Agent — Client Onboarding Agent

Configuration of the Vapi voice assistant used as the entry point of the AI Client Onboarding System. The full system prompt is shown in [`Screenshots/Client Onboarding Agent - system prompt.png`](Screenshots/Client%20Onboarding%20Agent%20-%20system%20prompt.png).

## Assistant

| Setting | Value |
|---------|-------|
| Name | **AI Client Onboarding System** (assistant: *Client Onboarding Agent*) |
| Transcriber | Soniox |
| Model | OpenAI |
| Voice | Vapi |
| Version | v6 |

## Purpose

A caller phones the agent and describes their project in natural speech (English, Urdu, Roman Urdu, or a mix). The agent professionally onboards the client, confirming all critical details before ending the call.

## Information collected (6 fields)

1. Client name
2. Company name
3. Email
4. Service required
5. Project description
6. Budget

The agent naturally asks for anything the caller leaves out, then sends the collected data to the n8n webhook.

## First message

> "Hello! Welcome to MATalogics. I'm your Client Onboarding Assistant, and I'll help us understand your project requirements so our team can assist you effectively. To get started, may I have your name and company name, please?"

## System prompt (outline)

```
# Role
You are the Client Onboarding Agent for MATalogics, a software and AI solutions company.
Your job is to professionally and naturally onboard potential clients by understanding
their project requirements, collecting the required information accurately, and ensuring
all critical details are confirmed before completing the conversation.

# Language Behavior
Handles English / Urdu / Roman Urdu callers; confirms details back to the caller.
...
```

## Webhook (end-of-call)

At the end of the conversation, the agent POSTs the structured call data to the n8n webhook:

```
POST https://zonix89.app.n8n.cloud/webhook/client-onboarding
```

> **Security note:** Vapi can sign these requests with an `x-vapi-secret` header. Enabling server-secret verification on the n8n side is recommended so only genuine Vapi calls can create records — see the Day-07 README.
