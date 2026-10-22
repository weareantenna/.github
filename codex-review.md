# Codex code review

Every PR in `weareantenna` gets a Codex review, paid for by the ChatGPT subscription of the bot account `i-am-antenna-bot`. Codex only uses that subscription when the bot itself asks for the review, so the bot comments `@codex review` on the PR:

- when a PR is opened, or a draft is marked ready for review
- when someone comments `@i-am-antenna-bot review` on a PR

GitHub sends PR and comment events to an n8n flow, and the flow posts the comment as the bot.

```
GitHub org webhook ──▶ n8n "GitHub: ask Codex to review PRs" ──▶ comment "@codex review" as i-am-antenna-bot ──▶ Codex reviews
```

The flow: https://n8n.weareantenna.dev/workflow/SwASPf0G3ikbbw3A

## What the flow skips

- Drafts and new pushes to an open PR
- PRs and comments from bots (Dependabot and the like)
- `@i-am-antenna-bot review` from anyone who isn't an owner, member or collaborator
- Requests without a valid GitHub signature

## Setup

### 1. Bot account

1. `i-am-antenna-bot` is a member of `weareantenna` with access to the repos it should review.
2. Its GitHub account is connected to ChatGPT, under the subscription that pays for the reviews.
3. In [Codex settings → Code review](https://chatgpt.com/codex/settings/code-review), logged in as the bot, turn on code review **per repo**. A repo that isn't enabled there ignores `@codex review`.
4. Set **Automatic code review** to **Follow personal preferences**, not **All PRs**. "All PRs" needs paid Codex credits. Without credits, Codex replies "usage limits have been reached" on every new PR.

### 2. Token for the bot

A personal access token of `i-am-antenna-bot`:

- Fine-grained, resource owner `weareantenna`, all repositories
- Permissions: **Pull requests** and **Issues**, read and write
- Note when it expires. Once it does, the flow fails on the last step and no reviews are requested.

In n8n it lives in the Header Auth credential **Github Bot Auth**: name `Authorization`, value `Bearer <token>`.

### 3. Org webhook

GitHub → `weareantenna` → Settings → Webhooks → Add webhook:

| Field | Value |
|---|---|
| Payload URL | `https://n8n.weareantenna.dev/webhook/codex-review` |
| Content type | `application/json` |
| Secret | the HMAC secret from the n8n credential **GitHub webhook secret (codex-review)** |
| Events | Let me select individual events → **Pull requests** and **Issue comments** |

The secret has to be the same on both sides. The flow rejects every request whose `X-Hub-Signature-256` doesn't match.

### 4. New repos

Org webhooks cover new repos automatically. Codex doesn't: turn on code review for the new repo in Codex settings (step 1.3).

## Troubleshooting

| Symptom | Where to look |
|---|---|
| No `@codex review` from the bot | GitHub → Settings → Webhooks → Recent deliveries: was the event sent, and did n8n answer 200? Then n8n → the flow → Executions. |
| Execution fails with "Invalid GitHub signature" | The webhook secret in GitHub and the n8n credential differ. Set a new one on both sides (`openssl rand -hex 32`). |
| Execution fails on "Comment @codex review" with 401 | The bot token expired or was revoked. Create a new one and update **Github Bot Auth**. |
| The bot comments, Codex doesn't react | Code review is off for that repo in Codex settings, or the subscription hit its limit. |
| n8n was down when the PR was opened | Redeliver the event under Recent deliveries, or comment `@i-am-antenna-bot review`. GitHub doesn't retry on its own. |

When editing the flow in n8n, reload the editor first. Saving from a stale tab overwrites the live version.
