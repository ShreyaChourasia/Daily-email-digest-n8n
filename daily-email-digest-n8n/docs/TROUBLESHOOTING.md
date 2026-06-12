# Troubleshooting

Common issues and their solutions.

---

## Gmail Issues

### "Operation: Send" instead of "Get Many"
**Symptom**: Gmail node tries to send an email instead of fetching.
**Fix**: Open the Gmail node → change Operation from `Send` to `Get Many`.

### No emails returned
**Symptom**: Gmail returns 0 items even though you have unread emails.
**Fix**: Check that the `Received After` filter expression is correct: `{{ $now.minus({ hours: 24 }).toISO() }}`. Also verify Read Status is set to `Unread emails only`.

### Gmail credential fails
**Symptom**: OAuth2 error or "invalid_grant".
**Fix**: Delete the Gmail credential in n8n and re-create it. Make sure you're authorizing with the correct Google account.

---

## Slack Issues

### "channel_not_found"
**Symptom**: Slack returns `channel_not_found` error.
**Fix**: The channel name in the Slack node doesn't match your actual Slack channel. Update it in ALL THREE Slack nodes.

### "not_in_channel"
**Symptom**: Slack returns `not_in_channel` error.
**Fix**: The bot hasn't been invited to the channel. In Slack, go to the channel and type: `/invite @YourBotName`

### Wrong channel in some nodes
**Symptom**: Some Slack messages post, others fail.
**Fix**: There are THREE Slack nodes — make sure all three have the same, correct channel name.

---

## Gemini Issues

### "No summary generated" for all emails
**Symptom**: Digest posts but every email shows "No summary generated".
**Fix**: The summary extraction expression may not match the Gemini response structure. Check the HTTP Request node output — the summary text is at `candidates[0].content.parts[0].text`.

### "No prompt specified"
**Symptom**: Basic LLM Chain returns this error.
**Fix**: This project uses an HTTP Request node instead. If you see a Basic LLM Chain node, it should be replaced with the HTTP Request approach described in the setup guide.

### HTTP 429 (Rate Limit)
**Symptom**: Gemini returns "Too Many Requests".
**Fix**: Add retry logic — in the HTTP Request node Settings: Retry on Fail = ON, Max Tries = 3, Wait Between Tries = 5000ms. For large inboxes, consider adding a Wait node (2-4 seconds) before the Gemini node.

### API key invalid
**Symptom**: HTTP 400 or 403 from Gemini.
**Fix**: Verify your API key at [aistudio.google.com/apikey](https://aistudio.google.com/apikey). Make sure it's pasted correctly in the URL with no extra spaces.

---

## General n8n Issues

### Workflow doesn't trigger at 8 AM
**Symptom**: Schedule doesn't fire.
**Fix**: Make sure the workflow is toggled **Active** (top-right switch). Also check your n8n instance timezone matches your local time.

### Expressions show as literal text
**Symptom**: `{{ $json.Subject }}` appears literally instead of being evaluated.
**Fix**: You need to enable expression mode by clicking the `=` icon next to the field before pasting the expression.

### "Cannot read property of undefined"
**Symptom**: Expression fails because upstream data is missing.
**Fix**: Click "Execute previous nodes" to load input data before testing a node, or run the full workflow.