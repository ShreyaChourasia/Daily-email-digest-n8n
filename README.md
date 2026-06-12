# Daily-email-digest-n8n
Automated daily email digest that fetches unread Gmail, summarizes with Google Gemini AI, and posts to Slack — built with n8n
# 📬 Daily Email Digest Automation

**Automatically summarize your unread emails and deliver a daily digest to Slack using n8n, Gmail, Google Gemini AI, and Slack.**

![n8n](https://img.shields.io/badge/n8n-workflow-orange?logo=n8n&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-API-red?logo=gmail&logoColor=white)
![Gemini](https://img.shields.io/badge/Google%20Gemini-2.5%20Flash-blue?logo=google&logoColor=white)
![Slack](https://img.shields.io/badge/Slack-Bot-4A154B?logo=slack&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Overview

This project is a fully automated workflow built on **n8n** that runs every morning at 8:00 AM and performs the following:

1. Connects to your Gmail account via OAuth2
2. Fetches all unread emails from the last 24 hours
3. Extracts the subject, sender, body snippet, and date from each email
4. Sends each email to Google Gemini 2.5 Flash for a concise 1-2 sentence AI summary
5. Compiles all summaries into a clean, bullet-pointed Slack message
6. Posts the formatted digest to a designated Slack channel

No more inbox anxiety — get a quick snapshot of everything that matters, every morning.

---

## Workflow Architecture

```
┌──────────────────┐
│  Schedule Trigger │  ← Fires daily at 8:00 AM
│  (Cron: 0 8 * * *)│
└────────┬─────────┘
         ▼
┌──────────────────┐     ┌─────────────────────────┐
│  Fetch Unread    │────▶│  Check Gmail Error (IF)  │
│  Emails (Gmail)  │     └──────┬──────────┬────────┘
└──────────────────┘            │          │
                          Error ▼          ▼ Success
                   ┌─────────────┐  ┌──────────────────┐
                   │ Post Error  │  │ Check Emails     │
                   │ to Slack    │  │ Exist (IF)       │
                   └──────┬──────┘  └──┬───────────┬───┘
                          ▼            │           │
                   ┌─────────────┐  No Emails ▼    ▼ Has Emails
                   │ Mark Failed │  ┌──────────┐  ┌──────────────────┐
                   └─────────────┘  │ Post "No │  │ Extract Email    │
                                    │ Emails"  │  │ Fields (Set)     │
                                    │ to Slack │  └────────┬─────────┘
                                    └──────────┘           ▼
                                                  ┌──────────────────┐
                                                  │ Summarize with   │
                                                  │ Gemini (HTTP)    │
                                                  └───┬──────────┬───┘
                                                      │          │
                                                Success ▼        ▼ Error
                                            ┌───────────┐  ┌───────────┐
                                            │ Set Gemini│  │ Set       │
                                            │ Summary   │  │ Fallback  │
                                            └─────┬─────┘  └─────┬─────┘
                                                  │              │
                                                  ▼              ▼
                                            ┌──────────────────────┐
                                            │ Aggregate Summaries  │
                                            └──────────┬───────────┘
                                                       ▼
                                            ┌──────────────────────┐
                                            │ Format Slack Message │
                                            │ (Code Node - JS)     │
                                            └──────────┬───────────┘
                                                       ▼
                                            ┌──────────────────────┐
                                            │ Post Digest to Slack │
                                            └──────────┬───────────┘
                                                       ▼
                                            ┌──────────────────────┐
                                            │ Mark Complete ✅     │
                                            └──────────────────────┘
```

---

## Sample Output

Here's what the Slack digest looks like:

```
📬 Daily Email Digest — Friday, Jun 12, 2026

19 unread emails summarized:

• Your profile is popular - 21 search appearances (from: LinkedIn)
  LinkedIn reports 21 profile views this week, with interest from Tata Communications.

• Complete your enrollment to unlock this... (from: Internshala Trainings)
  Internshala prompts completion of a web development training enrollment.

• Essential tips to stay safe from Cyber Frauds (from: SBI)
  SBI shares cybersecurity best practices for safe online banking.

• Shreya, not sure what course is right for you? (from: Udemy)
  Udemy recommends exploring course categories based on career interests.
```

---

## Features

- **Scheduled automation** — Runs daily at 8:00 AM, zero manual effort
- **Smart filtering** — Only fetches unread emails from the last 24 hours
- **AI-powered summaries** — Google Gemini generates concise 1-2 sentence summaries
- **Formatted output** — Clean bullet-point digest posted to Slack
- **Comprehensive error handling** — Gmail failures, empty inboxes, and API errors are all handled gracefully
- **Production-ready** — Includes retry logic, fallback messages, and workflow completion markers
- **Fully documented** — Sticky notes inside the workflow explain each section

---

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Automation Platform | n8n (self-hosted) | Workflow orchestration |
| Email Source | Gmail API (OAuth2) | Fetch unread emails |
| AI Summarization | Google Gemini 2.5 Flash | Generate email summaries |
| Notification | Slack API (Bot Token) | Post daily digest |
| Language | JavaScript (Code nodes) | Data transformation |

---

## Prerequisites

Before setting up, make sure you have:

- **n8n** installed ([Docker](https://docs.n8n.io/hosting/installation/docker/) or [npm](https://docs.n8n.io/hosting/installation/npm/))
- A **Gmail account** with API access enabled
- A **Google Cloud project** with Gemini API key
- A **Slack workspace** where you can create apps

---

## Quick Start

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/daily-email-digest-n8n.git
cd daily-email-digest-n8n

# Start n8n (if using Docker)
docker volume create n8n_data
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n

# Or start with npm
npx n8n
```

Then import the workflow — see [SETUP.md](docs/SETUP.md) for detailed instructions.

---

## Project Structure

```
daily-email-digest-n8n/
├── README.md                          # This file
├── LICENSE                            # MIT License
├── .gitignore                         # Git ignore rules
├── workflows/
│   └── daily_email_digest.json        # Importable n8n workflow
├── docs/
│   ├── SETUP.md                       # Step-by-step setup guide
│   ├── ARCHITECTURE.md                # Technical architecture details
│   └── TROUBLESHOOTING.md             # Common issues and fixes
└── screenshots/
    ├── workflow-canvas.png            # Full workflow in n8n editor
    ├── workflow-success.png           # Successful execution view
    ├── slack-digest-output.png        # Slack message result
    ├── gmail-node-config.png          # Gmail node configuration
    └── gemini-node-config.png         # Gemini HTTP Request config
```

---

## Error Handling

| Scenario | Behavior | Slack Output |
|----------|----------|--------------|
| Gmail connection fails | Posts error + stops | ⚠️ Gmail Connection Failed + error details |
| No unread emails | Posts info + stops | 📭 No unread emails in the last 24 hours |
| Gemini API fails | Retries once, then fallback | ⚠️ Summary unavailable (Gemini API error) |
| Gemini rate limit (429) | Retries with 5s delay | Auto-recovers on retry |
| Slack post fails | Errors in n8n execution log | Visible in n8n Executions tab |

---

## Configuration Reference

### Gmail Node

| Setting | Value |
|---------|-------|
| Resource | Message |
| Operation | Get Many |
| Limit | 50 |
| Filter: Read Status | Unread emails only |
| Filter: Received After | `{{ $now.minus({ hours: 24 }).toISO() }}` |

### Gemini HTTP Request

| Setting | Value |
|---------|-------|
| Method | POST |
| URL | `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=YOUR_KEY` |
| Body | JSON with email subject, from, and body |
| Temperature | 0.2 (precise, factual summaries) |

### Slack Node

| Setting | Value |
|---------|-------|
| Resource | Message |
| Operation | Send |
| Channel | Your designated channel |
| Message Type | Simple Text Message |

---

## Known Issues & Roadmap

### Current Limitations

- **Gemini summary extraction** — The AI summary parsing from the HTTP response needs refinement in the Set Gemini Summary node. Currently falls back to showing email subjects without summaries.
- **Rate limiting** — Sending 20+ emails simultaneously can trigger Gemini's 429 rate limit. A Wait node between requests would resolve this.

### Future Improvements

- [ ] Fix Gemini response parsing for reliable AI summaries
- [ ] Add batch processing with delays to handle rate limits
- [ ] Support multiple Slack channels based on email category
- [ ] Add email filtering by label or sender
- [ ] Weekly digest mode option
- [ ] Priority email highlighting
- [ ] HTML-formatted Slack messages with Block Kit
- [ ] Dashboard for digest analytics

---

## Contributing

Contributions are welcome! See the [Known Issues](#known-issues--roadmap) for areas that need help.

1. Fork the repository
2. Create a feature branch (`git checkout -b fix/gemini-parsing`)
3. Commit your changes (`git commit -m 'Fix Gemini response extraction'`)
4. Push to the branch (`git push origin fix/gemini-parsing`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

## Acknowledgments

- [n8n](https://n8n.io/) — Open-source workflow automation
- [Google Gemini](https://ai.google.dev/) — AI summarization API
- [Slack API](https://api.slack.com/) — Messaging platform

---

**Built with ❤️ using n8n workflow automation**