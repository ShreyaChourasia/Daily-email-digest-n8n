# Architecture

Technical deep-dive into how the Daily Email Digest workflow operates.

---

## Node Map

The workflow consists of **17 active nodes** plus **3 sticky note** documentation nodes.

| # | Node Name | Type | Purpose |
|---|-----------|------|---------|
| 1 | Daily 8AM Trigger | Schedule Trigger | Fires workflow daily at 08:00 |
| 2 | Fetch Unread Emails | Gmail | Fetches unread emails from last 24h |
| 3 | Check Gmail Error | IF | Routes Gmail errors vs successful fetch |
| 4 | Post Gmail Error to Slack | Slack | Notifies Slack of Gmail failures |
| 5 | Mark Failed - Gmail | Set | Terminal: marks workflow as failed |
| 6 | Check Emails Exist | IF | Routes based on whether emails were found |
| 7 | Post No Emails to Slack | Slack | Posts "no emails" message |
| 8 | Mark Complete - No Emails | Set | Terminal: marks workflow complete |
| 9 | Extract Email Fields | Set | Extracts subject, from, body, date |
| 10 | Summarize Email with Gemini | HTTP Request | Calls Gemini API for each email |
| 11 | Set Gemini Summary | Set | Extracts AI summary from response |
| 12 | Set Fallback Summary | Set | Provides fallback when Gemini fails |
| 13 | Aggregate All Summaries | Aggregate | Combines all items into one array |
| 14 | Format Slack Message | Code (JS) | Builds bullet-point formatted message |
| 15 | Post Digest to Slack | Slack | Sends final digest to channel |
| 16 | Mark Complete - Success | Set | Terminal: marks workflow complete |

---

## Data Flow

### Gmail Response Schema

When the Gmail node fetches emails with Simplify ON, each item contains:

```json
{
  "id": "19eb9e19086174fe",
  "threadId": "19eb9e19086174fe",
  "snippet": "Here's a special offer to help you explore more.",
  "Subject": "Course recommendations for you",
  "From": "Udemy <hello@students.udemy.com>",
  "To": "Your Name <you@gmail.com>",
  "internalDate": "1781235027000",
  "labels": [
    { "id": "INBOX", "name": "INBOX" },
    { "id": "UNREAD", "name": "UNREAD" }
  ]
}
```

**Important**: Gmail returns capitalized field names (`Subject`, `From`, `To`) — not lowercase.

### Extracted Fields Schema

After the Extract Email Fields node:

```json
{
  "subject": "Course recommendations for you",
  "from": "Udemy <hello@students.udemy.com>",
  "body": "Here's a special offer to help you explore more.",
  "date": "1781235027000"
}
```

### Gemini API Request

```json
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=API_KEY

{
  "contents": [{
    "parts": [{
      "text": "Summarize this email in 1-2 sentences...\n\nSubject: ...\nFrom: ...\nBody: ..."
    }]
  }]
}
```

### Gemini API Response

```json
{
  "candidates": [{
    "content": {
      "parts": [{
        "text": "Udemy recommends exploring new courses based on career interests and learning history."
      }],
      "role": "model"
    },
    "finishReason": "STOP"
  }],
  "usageMetadata": {
    "promptTokenCount": 325,
    "candidatesTokenCount": 14,
    "totalTokenCount": 1143
  }
}
```

Summary is at: `candidates[0].content.parts[0].text`

---

## Error Handling Strategy

The workflow uses three layers of error handling:

### Layer 1: Node-Level (Continue on Error)
The Gmail and Gemini nodes are configured with `onError: continueErrorOutput`. Failed items route to the error output instead of stopping the workflow.

### Layer 2: Conditional Routing (IF Nodes)
IF nodes check for errors and empty results, routing to appropriate Slack notifications.

### Layer 3: Retry Logic
The Gemini HTTP Request node retries failed requests once with a 5-second delay before falling back.

---

## Key Expressions

| Expression | Used In | Purpose |
|-----------|---------|---------|
| `{{ $now.minus({ hours: 24 }).toISO() }}` | Gmail filter | Fetches emails from last 24h |
| `{{ $json.Subject \|\| '(No Subject)' }}` | Extract Fields | Safe subject extraction |
| `{{ $json.From \|\| 'Unknown Sender' }}` | Extract Fields | Safe sender extraction |
| `{{ $json.snippet \|\| '' }}` | Extract Fields | Email body from Gmail snippet |
| `{{ $json.candidates[0].content.parts[0].text }}` | Set Summary | Gemini response extraction |
| `{{ $execution.id }}` | Error notifications | Tracks failed executions |