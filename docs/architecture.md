# CallsCatch — Architecture (portfolio summary)

## System context

CallsCatch is an **AI phone receptionist and automation** product for Australian SMBs. This public repo documents architecture only; API keys, n8n exports, and Retell agent configs live in a **private** deployment repo.

```
                         ┌─────────────────────────────────────┐
                         │         callscatch.com (web)        │
                         │   marketing · lead forms · CTAs     │
                         └──────────────────┬──────────────────┘
                                            │
     PSTN / SMS                              ▼
┌──────────────┐                    ┌───────────────┐
│   Twilio     │◄──────────────────►│   Retell AI   │
│  (AU numbers)│   voice + webhooks │  voice agents │
└──────┬───────┘                    └───────┬───────┘
       │                                  │
       │         ┌────────────────────────┘
       │         │
       ▼         ▼
┌──────────────────────────────────────────────────┐
│                    n8n (orchestration)            │
│  branching · retries · schedules · CRM writes    │
└────────────┬─────────────────┬─────────────────┘
             │                 │
             ▼                 ▼
    ┌────────────────┐  ┌───────────────────┐
    │ Google Sheets  │  │ Gmail / webhooks  │
    │ (lightweight   │  │ (replies, booking) │
    │  CRM)          │  │                    │
    └────────────────┘  └───────────────────┘
```

## Inbound call flow

1. Caller dials the business Twilio number (or misses a call → SMS/voice follow-up path).
2. Twilio routes audio to a **Retell AI** agent with business-specific prompt and tools.
3. Agent maintains **stateful** conversation (intent, contact details, booking preference).
4. On key events (lead captured, appointment requested), Retell fires **webhooks** to n8n.
5. n8n writes to Sheets, sends notifications, or triggers downstream workflows (e.g. calendar link, SMS confirmation).

## GTM / outbound (related repo: CallsCatch-gtm-portfolio)

Separate automation stack for **lead acquisition** (not mixed with live call handling):

| Workflow | Purpose |
|----------|---------|
| Places scrape | Collect local business leads (Google Places) |
| Cold email | 3-step sequence with AU opt-out compliance |
| Gmail reply handler | Classify replies, route hot leads |
| Retell outbound | AI demo calls + webhook scheduling |
| Social drafts | OpenAI Agent → branded post suggestions |

Shared patterns: `TEST_MODE`, batch caps, Melbourne timezone, Sheets as CRM.

## Design principles

| Principle | Implementation |
|-----------|----------------|
| **Separation of concerns** | Voice runtime (Retell + Twilio) vs workflow logic (n8n) vs marketing site |
| **Fail-safe automation** | Rate limits, test flags, explicit error branches in n8n |
| **Latency** | Short agent responses; SMS paths for async follow-up |
| **Security** | Secrets only in private env; webhooks validated; no keys in this repo |

## Data & privacy

- Call metadata and lead fields stored per client configuration (Sheets / client CRM).
- Prompts and agent definitions are **not** published here.
- Australian market: opt-out language on email flows; business hours in local timezone.

## Observability (production)

- n8n execution history for workflow failures
- Retell call logs for conversation quality
- Twilio message/voice logs for delivery issues

## Extension points

- New SMB vertical → duplicate Retell agent template + n8n branch
- CRM upgrade → replace Sheets sink with HubSpot/Pipedrive nodes
- Human handoff → Twilio transfer node + n8n escalation rule
