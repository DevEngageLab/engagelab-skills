# EngageLab OTP — Error code reference

| code | http | meaning                              | retryable | action |
|------|------|--------------------------------------|-----------|--------|
| 1000 | 500  | Internal error                       | yes       | backoff & retry |
| 2001 | 401  | Auth failed — bad credentials        | no        | check env vars |
| 2002 | 401  | Token expired or disabled            | no        | rotate credentials |
| 2004 | 403  | No permission for this API           | no        | contact support |
| 3001 | 400  | Invalid JSON format                  | no        | fix request body |
| 3002 | 400  | Bad parameters                       | no        | check param names/types |
| 3003 | 400  | Business validation failed / expired | no        | re-send a new OTP |
| 3004 | 400  | Rate limited (same recipient+tpl)    | no        | wait then retry |
| 4001 | 400  | Template not found                   | no        | verify template ID and approval |
| 5001 | 400  | Send failed (general)                | maybe     | try once, then switch channel |
| 5011 | 400  | Invalid phone number format          | no        | validate E.164 |
| 5012 | 400  | Recipient unreachable                | no        | switch channel or notify user |
| 5013 | 400  | Number blacklisted                   | no        | do not retry |
| 5014 | 400  | Content violates policy              | no        | review template wording |
| 5015 | 400  | Message intercepted/rejected         | no        | try different channel |
| 5016 | 400  | Internal send error                  | yes       | backoff & retry |
| 5017 | 400  | No permission for China region       | no        | apply for permission |
| 5018 | 400  | Handset off/suspended                | no        | retry later |
| 5019 | 400  | User unsubscribed                    | no        | do not retry |
| 5020 | 400  | Number unregistered/empty            | no        | validate number |

## Category mapping

```python
def classify(code):
    if code in (2001, 2002, 2004):     return "auth"
    if code in (3001, 3002):            return "bad_request"
    if code == 3003:                    return "expired"
    if code == 3004:                    return "rate_limited"
    if code == 4001:                    return "template_not_found"
    if code in (5011, 5020):            return "invalid_recipient"
    if code in (5012, 5018):            return "unreachable"
    if code in (5013, 5019):            return "blocked"
    if code in (5014, 5015):            return "content_blocked"
    if code in (1000, 5001, 5016):      return "transient"
    return "unknown"
```
