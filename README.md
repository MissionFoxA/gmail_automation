# Email Triage

A Python script that reads today's emails from Gmail, asks a language model whether each one needs attention, and stores the results in a local SQLite database.

Built as a learning project to practise working with APIs, OAuth, SQL, and LLM output validation.

## What it does

1. Logs in to Gmail with read-only access (OAuth).
2. Fetches the emails from the last 24 hours, following Gmail's paginated results.
3. Runs a prompt-injection check on each email. Suspicious emails are flagged `needs_review` and never sent to the model.
4. Sends the rest to a language model, which labels each email `important`, `normal`, or `needs_review`.
5. Checks the model's reply against the allowed labels before saving it.
6. Stores the results in `emails.db` (SQLite). Re-running the script updates existing rows rather than duplicating them.

## Setup

**Requirements:** Python 3.10 or newer, and a Google account.

1. Clone the repository and create a virtual environment:
   ```bash
   git clone https://github.com/YOUR-USERNAME/email-triage.git
   cd email-triage
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install google-api-python-client google-auth-httplib2 google-auth-oauthlib openai
   ```

2. Create Google credentials:
   - In the Google Cloud Console, create a project and enable the Gmail API.
   - Set up the OAuth consent screen as External, and add your own address as a test user.
   - Create an OAuth client ID of type **Desktop app**, download the JSON file, and save it as `credentials.json` in the project folder.

3. Start a local model with [Ollama](https://ollama.com):
   ```bash
   ollama pull llama3.2
   ```

4. Run the script:
   ```bash
   python main.py
   ```
   The first run opens a browser so you can approve read-only access. This creates `token.json`.

## Privacy

- Emails are read with the `gmail.readonly` scope. The script can't send, delete, or modify anything.
- Only the sender, subject, and the first 3,000 characters of each body are sent to the model.
- With a local model, email content never leaves your computer.
- `credentials.json`, `token.json`, and `emails.db` are excluded from Git by `.gitignore`. Never commit them.

## Testing

The `test_cases.md` file lists 14 test emails with expected labels, the actual results for each prompt version, and notes on every mismatch.

Results across prompt versions:

| Version | Matched expected | Notes |
|---|---|---|
| v1 (original prompt) | 6 of 10 | Over-flagged promotions and empty emails |
| v2 (examples added) | 9 of 10 | Fixed promotions; prompt injection still succeeded |
| v3 (code check added) |  9 of 10 | Injection caught for the tested phrasing |

## Limitations

- **Prompt injection:** the code check uses a list of patterns. It catches the phrasings listed in `main.py`, not every possible attack. Emails with new phrasing can still reach the model, and in testing the model followed an injected instruction once.
- **Judgement calls:** some emails, such as meeting cancellations, have no single correct label.
- **HTML-only emails:** the body extractor reads plain text only, so an email with no plain-text part arrives with an empty body.
- **Scope:** the project only labels emails. It doesn't act on them.

## Project structure

```
main.py          Gmail login, classification, and storage
query.py         Run SQL queries against the results
test_cases.md    Test emails, expected labels, and results