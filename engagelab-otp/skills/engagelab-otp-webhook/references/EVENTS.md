# EngageLab OTP — Webhook event schemas

## 1. message_status event

```json
{
  "total": 1,
  "rows": [{
    "message_id": "1725407449772531712",
    "to": "+6598765432",
    "server": "sms",
    "channel": "sms",
    "itime": 1701234567,
    "custom_args": { "order_id": "ORD123", "user_id": "U456" },
    "status": {
      "message_status": "delivered",
      "status_data": {
        "msg_time": 1701234560,
        "message_id": "1725407449772531712",
        "current_send_channel": "whatsapp",
        "template_key": "verify_code",
        "business_id": "1001"
      },
      "billing": { "cost": 0.005, "currency": "USD" },
      "error_code": 0
    }
  }]
}
```

SDK exposes as:

```ts
{
  kind: "message_status",
  message_id, to, server, channel,
  timestamp,         // itime, unix seconds
  custom_args,
  status,            // see enum below
  is_terminal,       // derived: true for delivered, delivered_failed, sent_failed, verified*, target_invalid
  current_send_channel,
  template_key, business_id,
  msg_time,
  billing,
  error_code,
  error_message,
  raw                // full original row
}
```

### message_status enum

```
plan                  message queued in send queue            (non-terminal)
target_valid          recipient validated                     (non-terminal)
target_invalid        recipient invalid (E.164 fail, etc.)    (terminal)
sent                  successfully submitted to carrier       (non-terminal)
sent_failed           submission failed                       (terminal)
delivered             user's device received the message      (terminal)
delivered_failed      sent, but undelivered                   (terminal — unless fallback follows)
verified              user successfully entered the code      (terminal)
verified_failed       user entered wrong code (no retries)    (terminal)
verified_timeout      user did not enter code in time          (terminal)
```

## 2. notification event

Account-level alerts, not tied to a specific message.

```json
{
  "rows": [{
    "server": "otp",
    "itime": 1712458844,
    "notification": {
      "event": "insufficient_balance",
      "notification_data": {
        "remain_balance": -0.005,
        "balance_threshold": 2,
        "currency": "USD"
      }
    }
  }]
}
```

### Notification events

| event                            | data fields |
|----------------------------------|-------------|
| `insufficient_balance`           | remain_balance, balance_threshold, currency |
| `insufficient_verification_rate` | rate, threshold |
| `template_audit_result`          | template_id, status, reason |

## 3. uplink event

User replied to a message (e.g. SMS "STOP").

```json
{
  "rows": [{
    "server": "otp",
    "itime": 1741083306,
    "message_id": "0",
    "business_id": "0",
    "response": {
      "event": "uplink_message",
      "response_data": {
        "message_sid": "SM1",
        "account_sid": "AC1",
        "from": "+6591234567",
        "to":   "+6580000000",
        "body": "STOP"
      }
    }
  }]
}
```

Common bodies: `STOP`, `UNSUBSCRIBE`, `HELP`, free-form responses.

## 4. system_event

Audit log of console actions. Usually just log these.

```json
{
  "rows": [{
    "server": "otp",
    "itime": 1694012345,
    "system_event": {
      "event": "account_login",
      "data": {
        "business_id": "123",
        "operator": { "email": "admin@example.com", "ip": "1.2.3.4" }
      }
    }
  }]
}
```

### system_event types

| event              | meaning |
|--------------------|---------|
| `account_login`    | user logged into console |
| `key_manage`       | API key created/rotated |
| `template_manage`  | template created/edited |
| `msg_history`      | message history exported |
| `api_call`         | high-volume API call audit |
