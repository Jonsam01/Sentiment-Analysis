# Sentiment Analysis: Customer Feedback Triage

An n8n system that takes customer feedback from a web form, classifies the sentiment with an LLM, and alerts the business owner when something needs attention. Built for small businesses (pilot: a salon).

## How it works

1. **Sentiment Analysis**: a webhook receives a form submission. The workflow validates it, checks the form token, and saves it to Supabase. Personal details are redacted before the text goes to the model. Groq classifies it, with OpenAI as a fallback. The answer is parsed and verified, then saved. If the feedback is negative, a safety concern, or needs a reply, the owner gets an email with an "I'm handling this" button. If the model output can't be read, a failure notice is emailed instead.
2. **Acknowledge feedback**: the button in the alert email calls this webhook, which marks the case as acknowledged in Supabase.
3. **Escalate unacknowledged alerts**: runs hourly, finds alerts nobody has handled, and sends a reminder email.
4. **Weekly summary**: every Monday at 8am, emails the owner a summary of sentiment, response time and open alerts.

## Stack

n8n, Supabase (Postgres), Groq API, OpenAI API, Gmail, Lovable (feedback form)

## Workflows

| File | Trigger |
|------|---------|
| `workflows/Sentiment_Analysis.json` | Webhook (form submit) |
| `workflows/Acknowledge_feedback.json` | Webhook (button click) |
| `workflows/Escalate_unacknowledged_alerts.json` | Schedule (hourly) |
| `workflows/Weekly_summary.json` | Schedule (Mondays, 8am) |

## Setup

1. In n8n, import each JSON file from `workflows/`.
2. Create the credentials: Supabase, Gmail OAuth2, Groq (header auth), OpenAI.
3. Create the Supabase tables and views the workflows use (`feedback`, `overdue_alerts`, `weekly_summary`, plus a business table).
4. Replace every placeholder:
   - `YOUR_CREDENTIAL_ID`: re-link in each node
   - `YOUR_WEBHOOK_PATH` and `ack-YOUR_SECRET_PATH`: your webhook paths
   - `YOUR_FORM_TOKEN`: the token your form sends
   - `YOUR-FORM-DOMAIN`: your form's URL
   - `owner@example.com`: the alert recipient
5. Activate the workflows.

## Design notes

- Validation rejects bad input before anything is saved.
- Duplicate submissions are blocked by a unique `submission_id`.
- Customer details are redacted before reaching the LLM.
- Low-confidence results are flagged for a second look.
- Unhandled alerts escalate automatically, so nothing gets lost.

## Author

Jon Samuel, AI Systems Architect
