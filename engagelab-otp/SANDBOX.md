# EngageLab Sandbox — Mock Account & Magic Numbers

The sandbox lets developers try EngageLab without signing up. Every call returns a realistic response, including webhooks if a webhook URL is configured. No real messages are sent.

## Sandbox credentials

```
ENGAGELAB_DEV_KEY=engagelab_sandbox_demo
ENGAGELAB_DEV_SECRET=engagelab_sandbox_secret
```

These credentials work against the production endpoint `https://otp.api.engagelab.cc` — the platform recognizes them and routes to the sandbox backend.

## Pre-built test templates

| Template ID            | Purpose |
|------------------------|---------|
| `sandbox-otp-tpl`      | Platform-generated 6-digit code, EN/zh_CN bilingual |
| `sandbox-otp-tpl-wa`   | WhatsApp-only template (for testing fallback flows) |
| `sandbox-custom-tpl`   | Custom-code template, expects `{code}` params |

## Magic recipients (phone numbers / emails)

Send to these recipients to deterministically trigger specific scenarios.

### Success scenarios

| Recipient | Behavior | When to use |
|-----------|----------|-------------|
| `+10000000000` | Delivered after 2s | Test happy-path end-to-end |
| `+10000000005` | Delivered; correct verify code is **`123456`** | Test verify() flow |
| `+10000000010` | Delivered immediately (no delay) | Test webhook latency in isolation |
| `sandbox@example.com` | Delivered via email channel | Test email delivery |

### Failure scenarios (immediate API errors)

| Recipient | API response | When to use |
|-----------|--------------|-------------|
| `+10000000001` | code `5011`, "Invalid phone number" | Test invalid-number error path |
| `+10000000011` | code `5013`, "Blacklisted" | Test do-not-retry logic |
| `+10000000012` | code `5019`, "Unsubscribed" | Test unsubscribe handling |
| `+10000000099` | code `3004`, "Rate limited" | Test rate-limit backoff |

### Delivery failure scenarios (200 OK, but webhook reports failure)

| Recipient | Webhook event | When to use |
|-----------|---------------|-------------|
| `+10000000002` | `delivered_failed` with error `5012` (unreachable) | Test post-send failure handling |
| `+10000000003` | No webhook ever (simulates lost SMS) | Test timeout behavior |
| `+10000000018` | `delivered_failed` with error `5018` (handset off) | Test retry-later logic |

### Multi-channel fallback scenarios

| Recipient | Sequence | When to use |
|-----------|----------|-------------|
| `+10000000004` | WA `delivered_failed` → SMS `sent` → SMS `delivered` | Test fallback handling |
| `+10000000014` | WA → SMS → Voice → all fail | Test exhausted fallback |
| `+10000000024` | WA `sent` → WA `delivered` (no fallback) | Test happy path on WA |

### Verify scenarios

After sending to `+10000000005`:

| Verify code submitted | Result |
|-----------------------|--------|
| `123456`              | `verified: true` |
| Anything else         | `verified: false` |
| Same code, twice      | Error `3003` (already used) |
| After 5 minutes       | Error `3003` (expired) |

## Sandbox quotas

```
500 requests / day              free, no signup
5,000 requests / day            free with sandbox API key
unlimited                       production tier
```

## Webhook testing in sandbox

Configure your webhook URL in the sandbox dashboard (or via API):

```
PUT https://otp.api.engagelab.cc/v1/sandbox/webhook
{
  "url": "https://your-server.example.com/webhook/engagelab",
  "username": "sandbox_demo",
  "secret": "sandbox_webhook_secret"
}
```

Signatures use `sandbox_webhook_secret` as the HMAC key — the same `WebhookVerifier` code works for sandbox and production, just different secrets.

For local development with no public URL, use [ngrok](https://ngrok.com):

```bash
ngrok http 3000
# Then set webhook URL to https://<random>.ngrok.io/webhook/engagelab
```

## Limits and disclaimers

- Sandbox responses are deterministic but slightly delayed (1-3s) to simulate real network latency.
- Sandbox does NOT validate template content — any params are accepted.
- Sandbox webhooks fire from IP `54.151.0.99` (sandbox-only). Whitelist it separately from production IPs.
- Sandbox credentials are publicly known — never use them for anything sensitive.
- Sandbox state resets every 24 hours; old `message_id`s become invalid.
