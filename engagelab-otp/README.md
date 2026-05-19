# EngageLab Agent Skills

Official [Agent Skills](https://agentskills.io) for the EngageLab platform. Install once; works with Claude Code, Cursor, GitHub Copilot, Codex, Windsurf, and any tool that supports the Skills format.

## Install

```bash
npx skills add engagelab/agent-skills
```

Or pick individual skills:

```bash
npx skills add engagelab/agent-skills:engagelab-otp-send
npx skills add engagelab/agent-skills:engagelab-otp-verify
npx skills add engagelab/agent-skills:engagelab-otp-webhook
```

## Try without signing up

These skills work with a public sandbox. No account needed to evaluate:

```
ENGAGELAB_DEV_KEY=sandbox_demo
ENGAGELAB_DEV_SECRET=sandbox_secret
```

Send a test OTP to a magic sandbox number — see [SANDBOX.md](SANDBOX.md) for the full magic number list.

## Skills in this repo

| Skill                       | When the AI uses it |
|-----------------------------|---------------------|
| `engagelab-otp-send`        | Sending OTP verification codes (SMS/WhatsApp/Voice/Email) |
| `engagelab-otp-verify`      | Verifying user-entered codes (platform-generated mode) |
| `engagelab-otp-webhook`     | Receiving delivery callbacks, processing fallback events |

## How Skills work

Skills are Markdown files (`SKILL.md`) with structured frontmatter. AI coding assistants read the `description` field to decide when to apply the skill, then load the body for step-by-step instructions. The AI writes correct code instead of guessing your API.

Without skills:

```
You: "Send an OTP to this user with EngageLab"
AI:  [guesses parameter names, uses old API, misses auth header format, errors]
```

With skills:

```
You: "Send an OTP to this user with EngageLab"
AI:  [reads engagelab-otp-send skill, generates correct, modern, working code]
```

## SDKs (the skills reference these)

- npm: [`engagelab-otp`](https://www.npmjs.com/package/engagelab-otp)
- PyPI: [`engagelab-otp`](https://pypi.org/project/engagelab-otp/)

## Production credentials

Get free production credentials at https://www.engagelab.com — no credit card needed for evaluation tier.

## License

MIT — fork, modify, share. PRs welcome.
