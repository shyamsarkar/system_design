# 📩 Notification System Design (Interview Ready)

## 🔹 1. Requirements

### Functional Requirements

* Send notifications via:

  * Email
  * SMS
  * Push notifications
* Support **bulk sending** (e.g., 1 lakh users)
* Retry failed notifications
* Respect user preferences (opt-in / opt-out)

### Non-Functional Requirements

* High scalability 📈
* High reliability (no message loss)
* Low latency (for real-time notifications)

---

## 🔹 2. High-Level Architecture

```id="a1b2c3"
Client / Admin
      ↓
Notification API (Rails)
      ↓
Queue (Sidekiq + Redis)
      ↓
Workers
      ↓
External Providers (Email/SMS/Push)
```

---

## 🔹 3. Core Components

### 1. Notification Service (Backend)

* Accepts request
* Validates data
* Pushes jobs to queue

### 2. Queue (Very Important)

* Use background processing instead of direct sending
* Prevents server overload

### 3. Workers

* Consume jobs from queue
* Send notifications

### 4. External Providers

* Email → SMTP / SendGrid
* SMS → Twilio
* Push → Firebase

---

## 🔹 4. Bulk Notification Strategy

### ❌ Bad Approach

```ruby id="bad1"
users.each do |user|
  send_email(user)
end
```

### ✅ Good Approach

```ruby id="good1"
users.each do |user|
  NotificationJob.perform_async(user.id)
end
```

---

## 🔹 5. Retry Mechanism

* Use automatic retries (e.g., Sidekiq retries)
* Apply exponential backoff

```id="retry1"
Retry: 1s → 5s → 30s → 2min
```

---

## 🔹 6. Idempotency

### Problem:

Duplicate notifications

### Solution:

* Store notification records
* Check before sending

---

## 🔹 7. User Preferences

```id="prefs1"
user_id
email_enabled
sms_enabled
push_enabled
```

* Only send if user has enabled that channel

---

## 🔹 8. Rate Limiting

* Prevent spam
* Avoid provider blocking

Example:

* Max 5 SMS/min per user

---

## 🔹 9. Scaling Strategy

### Horizontal Scaling

* Add multiple workers

### Queue Separation

```yaml id="queues1"
:queues:
  - critical   # OTP
  - default    # normal notifications
  - low        # marketing
```

---

## 🔹 10. Advanced Concepts

### Fan-out Problem

* Sending 1 message to 1M users

**Solutions:**

* Batch processing
* Pub/Sub model

---

### Notification Types

| Type          | Example             |
| ------------- | ------------------- |
| Transactional | OTP, payment alerts |
| Marketing     | Offers, promotions  |

---

### Delivery Tracking

Store:

* sent
* delivered
* failed

---

## 🔹 11. Final Architecture

```id="final1"
API Server (Rails)
   ↓
Queue (Redis)
   ↓
Workers (Sidekiq)
   ↓
Email/SMS/Push Providers
```

---

## 🎯 One-Line Answer

> A notification system uses background queues like Sidekiq to asynchronously send messages via external providers, ensuring scalability, retries, and reliability.
