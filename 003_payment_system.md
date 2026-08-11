# 💳 Payment System Design (Interview Ready)

## 🔹 1. Requirements

### Functional Requirements

- User should be able to make a payment
- Prevent duplicate payments
- Support payment retries
- Maintain transaction history
- Notify users of payment status
- Support refunds (optional/advanced)

### Non-Functional Requirements

- High reliability (no money loss)
- Strong consistency for payment records
- High availability
- Secure communication
- Scalable architecture
- Fault tolerant

---

# 🔹 2. High-Level Architecture

```
                +----------------+
                |     Client     |
                +----------------+
                        |
                        v
                +----------------+
                |   API Server   |
                +----------------+
                        |
             +----------+----------+
             |                     |
             v                     v
      +-------------+      +---------------+
      | Payment DB  |      | Background Job|
      +-------------+      |   (Sidekiq)   |
             |             +---------------+
             |                     |
             +----------+----------+
                        |
                        v
              +--------------------+
              | Payment Gateway    |
              | (Stripe/Razorpay)  |
              +--------------------+
                        |
                        v
                    Bank/Card
                        |
                        v
                   Webhook Event
                        |
                        v
                +----------------+
                |   API Server   |
                +----------------+
                        |
                        v
                  Update Database
```

---

# 🔹 3. Payment Flow

```
User clicks Pay
        ↓
Backend creates payment
        ↓
Generate idempotency key
        ↓
Call Payment Gateway
        ↓
Gateway processes payment
        ↓
User may close browser
        ↓
Gateway sends webhook
        ↓
Backend updates payment status
        ↓
Client fetches latest status
```

---

# 🔹 4. Core Components

## API Server

Responsible for

- Creating payments
- Validating requests
- Calling payment gateway
- Handling webhooks
- Returning payment status

---

## Payment Gateway

Responsible for

- Processing payment
- Tokenizing card information
- Communicating with banks
- Sending webhook events

Examples

- Stripe
- Razorpay
- PayPal

---

## Database

Stores

- Payment information
- Transaction status
- Gateway transaction IDs
- Idempotency keys

---

## Background Workers (Sidekiq)

Used for

- Retry logic
- Notifications
- Reconciliation jobs
- Non-blocking tasks

---

# 🔹 5. Database Design

```sql
payments
--------

id
user_id
order_id
amount
currency
status
gateway_transaction_id
idempotency_key
failure_reason
created_at
updated_at
```

Indexes

```
UNIQUE(idempotency_key)

INDEX(user_id)

INDEX(order_id)

INDEX(status)
```

---

# 🔹 6. Payment State Machine

```
created
    ↓
processing
    ↓
success

or

created
    ↓
processing
    ↓
failed
```

Some gateways also support

```
authorized
      ↓
captured
      ↓
refunded
```

---

# 🔹 7. Idempotency (Most Important)

## Problem

User clicks Pay button multiple times.

Without protection

```
Request 1 → Charge ₹100

Request 2 → Charge ₹100

Result = ₹200 charged
```

## Solution

Generate an idempotency key.

```
abc123

↓

Store in database

↓

Unique index

↓

Same key returns same payment
```

Example

```ruby
payment = Payment.find_by(idempotency_key: key)

return payment if payment.present?

Payment.create!(
  idempotency_key: key,
  amount: amount
)
```

Benefits

- Prevent duplicate payments
- Safe retries
- Handles network failures

---

# 🔹 8. Webhooks

Webhooks are the source of truth.

Example

```
User pays

↓

Gateway processing

↓

User closes browser

↓

Gateway finishes payment

↓

Gateway sends webhook

↓

Backend updates payment

↓

Client sees SUCCESS
```

Always verify webhook signatures before processing.

---

# 🔹 9. Failure Handling

Possible failures

- Network timeout
- User refreshes page
- Gateway unavailable
- Response lost
- Duplicate requests

Solutions

- Retry with same idempotency key
- Webhook confirmation
- Background retry jobs
- Database constraints

---

# 🔹 10. Concurrency & Race Conditions

Possible issue

Two requests reach the server simultaneously.

Solutions

- Unique index
- Database transactions
- Row locking (SELECT ... FOR UPDATE)
- Optimistic locking where appropriate

---

# 🔹 11. Security

- HTTPS
- Never store raw card details
- Tokenization by payment gateway
- Verify webhook signatures
- Encrypt sensitive information
- Authentication & authorization

---

# 🔹 12. Scaling

- Multiple API servers
- Background workers (Sidekiq)
- Database indexing
- Read replicas (if needed)
- Horizontal scaling
- Load balancer

---

# 🔹 13. Reconciliation

Sometimes

Gateway = SUCCESS

Database = FAILED

Periodic reconciliation job

```
Database

↓

Gateway API

↓

Compare

↓

Fix mismatched records
```

This ensures financial accuracy.

---

# 🔹 14. Refunds (Advanced)

Flow

```
Client

↓

Refund Request

↓

Backend validates payment

↓

Gateway Refund API

↓

Webhook

↓

Refund Status Updated
```

Refunds should also use idempotency keys to avoid duplicate refunds.

Support

- Full refund
- Partial refund

Refund status

```
requested

↓

processing

↓

completed

or

failed
```

---

# 🔹 15. Interview Questions

Q. How do you prevent duplicate payments?

Answer:
Use idempotency keys with a unique database constraint.

---

Q. Why use webhooks?

Answer:
Users may close the browser before payment completes. Webhooks provide the final payment status.

---

Q. Why not trust the frontend response?

Answer:
The frontend can lose network connectivity or be closed. The webhook is the reliable source of truth.

---

Q. Why use Sidekiq?

Answer:
To process retries, notifications, reconciliation, and other asynchronous tasks without blocking API requests.

---

Q. What happens if the webhook is received twice?

Answer:
Webhook processing should be idempotent. Ignore duplicate events that have already been processed.

---

Q. How do you avoid double spending?

Answer:
Use idempotency keys, unique constraints, transactions, and row locking.

---

# 🔹 16. One-Line Summary

> A payment system ensures reliable and secure transactions using idempotency keys, database constraints, asynchronous webhooks, retries, reconciliation, and background workers while maintaining strong consistency for payment records.