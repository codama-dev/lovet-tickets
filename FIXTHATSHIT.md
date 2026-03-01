# 🦞 @fixthatshit Bot Automation

> **⚠️ OpenClaw/AI Automation Documentation**  
> This file documents the automated issue-fixing bot powered by OpenClaw and Claude.  
> This is **NOT** part of the LoVeT codebase itself - it's automation tooling.

**Owner:** Nadav (ncamaa)  
**Powered by:** [OpenClaw](https://openclaw.ai) + Claude Code

---

Automated code assistant that works on GitHub issues when tagged with `@fixthatshit`.

## How It Works

1. **Tag the bot** in a GitHub issue comment: `@fixthatshit please fix this in clinic-ai-fe`
2. **GitHub Action triggers** (`.github/workflows/fixthatshit.yml`)
   - Creates a feature branch (`fix/issue-{number}`)
   - Comments on the issue with branch info
   - (TODO) Updates issue status to "In Progress"
   - (TODO) Sends webhook to local Claude Code bot
3. **Claude Code bot** receives webhook and:
   - Checks out the branch
   - Reads the issue description
   - Makes code changes
   - Runs Playwright tests
   - Pushes changes to the branch
   - Creates a Pull Request
4. **(TODO) GitHub Action** updates issue status to "PR Created"

## Current Status

✅ **Working:**
- Webhook server running at `http://127.0.0.1:8765`
- GitHub workflow triggers on `@fixthatshit` mentions
- Branch creation and linking to issues
- Auto-detection of target repo from comment

🚧 **TODO:**
1. **Find GitHub Project status IDs**
   - Need IDs for "In Progress" and "PR Created" statuses
   - Update `fixthatshit.yml` env vars with these IDs
   - Uncomment the status update steps

2. **Expose webhook publicly**
   - Options: ngrok, cloudflared tunnel, or public IP
   - Update `fixthatshit.yml` with the public webhook URL
   - Enable the webhook step

3. **Test Claude Code CLI integration**
   - Verify `claude code` command works with the parameters
   - Adjust `fixthatshit-webhook.js` based on actual Claude CLI syntax

4. **Add PR creation step**
   - Bot should create PR after pushing changes
   - Link PR back to the issue
   - Update issue status to "PR Created"

## Setup Files

- **Webhook Server:** `~/.openclaw/workspace/fixthatshit-webhook.js`
  - Listens on port 8765
  - Health check: `curl http://127.0.0.1:8765/health`
  - Logs: `~/.openclaw/workspace/fixthatshit.log`

- **GitHub Workflow:** `.github/workflows/fixthatshit.yml`
  - Triggers on issue comments containing `@fixthatshit`
  - Creates feature branches
  - (Will) Send webhooks to bot

- **Project Workspace:** `~/Documents/clinique-i/`
  - All repos cloned here
  - Separate from company work (IT won't monitor)

## Usage Examples

```
@fixthatshit please implement this feature in clinic-ai-fe
```

```
@fixthatshit fix this bug in the backend
```

```
@fixthatshit update the admin-frontend according to the spec above
```

## Repository Detection

The bot auto-detects which repo to work on from your comment:
- Mentions "clinic-ai-fe" → `codama-dev/clinic-ai-fe`
- Mentions "admin-frontend" → `codama-dev/clinic-i-admin-frontend`
- Mentions "backend" → `codama-dev/lovet-express-backend`
- No mention → Uses the repo where the issue exists

## Workflow States

```
Backlog → [tag @fixthatshit] → In Progress → [bot pushes code] → PR Created → [you review] → Done
```

## Security Notes

- Bot uses `ncamaa` GitHub account (not company account `nadav-hsp`)
- Workspace is in `~/Documents/clinique-i` (separate from company projects)
- Never pushes directly to main branch
- Always creates PR for review

## Troubleshooting

**Webhook server not running?**
```bash
node ~/.openclaw/workspace/fixthatshit-webhook.js &
```

**Check webhook logs:**
```bash
tail -f ~/.openclaw/workspace/fixthatshit.log
```

**Test webhook manually:**
```bash
curl -X POST http://127.0.0.1:8765/webhook/github \
  -H "Content-Type: application/json" \
  -d '{
    "issue_number": "123",
    "repo": "codama-dev/clinic-ai-fe",
    "branch": "fix/issue-123",
    "title": "Test issue",
    "body": "Test description",
    "issue_url": "https://github.com/codama-dev/lovet-tickets/issues/123"
  }'
```

## Next Steps

1. Find the GitHub Project status IDs (ask in issue or check project settings)
2. Set up a public webhook URL (ngrok/cloudflared)
3. Test the full flow end-to-end
4. Iterate based on what works!

---

**Bot created by:** @fixthatshit  
**Last updated:** 2026-02-28
