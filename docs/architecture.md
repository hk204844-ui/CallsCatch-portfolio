# CallsCatch — Architecture (portfolio summary)

```
Inbound call/SMS ──► Twilio ──► Retell AI (voice agent)
                                    │
                                    ▼
                              n8n workflows
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
              Lead capture    Appointments     Notifications
```

## Design principles

- Voice agents maintain conversation state for natural SMB interactions.
- n8n handles branching logic, retries, and integrations without exposing API keys in this repo.
- Twilio provides PSTN/SMS transport in the Australian market.

## Security

Webhook secrets, Retell API keys, and Twilio credentials are stored only in the private deployment environment.
