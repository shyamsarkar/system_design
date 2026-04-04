# 💳 Payment System Design (Interview Ready)

## 🔹 1. Requirements

### Functional Requirements

* User should be able to **make payment**
* Prevent **duplicate payments**
* Support **success / failure / retry**
* Maintain **transaction history**

### Non-Functional Requirements

* High reliability (no money loss ❗)
* Strong consistency
* Secure system

---

## 🔹 2. High-Level Flow

```
Client → Backend → Payment Gateway → Bank
                    ↓
                 Webhook
                    ↓
                 Backend → DB
```

---

## 🔹 3. Key Components

### 1. Payment Service (Backend)

* Creates payment
* Stores status
* Communicates with payment gateway

### 2. Payment Gateway

* External service (e.g., Stripe)
* Handles actual money transfer

### 3. Database

Stores:

* transaction_id
* user_id
* amount
* status (pending, success, failed)

### 4. Webhooks

* Gateway notifies backend about payment status

---

## 🔹 4. Idempotency (🔥 Most Important)

### Problem:

User clicks "Pay" multiple times → multiple charges

### Solution:

Use **Idempotency Key**

```ruby
if payment_exists?(idempotency_key)
  return existing_payment
else
  create_payment
end
```

👉 Same request = same result (no duplicate payment)

---

## 🔹 5. Database Design

```sql
payments
--------
id
user_id
amount
status
idempotency_key (unique)
created_at
```

👉 Add **unique index on idempotency_key**

---

## 🔹 6. Payment States

```
created → pending → success / failed
```

---

## 🔹 7. Failure Handling

### Cases:

* Payment success but response lost
* Payment failed but retry needed

### Solution:

* Use **webhooks as source of truth**
* Retry failed payments safely

---

## 🔹 8. Security

* Use HTTPS
* Do not store raw card details
* Use tokenization via gateway

---

## 🔹 9. Scaling

* Use background jobs (Sidekiq)
* Use DB indexing
* Cache where needed

---

## 🔹 10. Advanced Concepts

### Race Conditions

* Use DB constraints (unique index)

### Double Spending

* Prevent using idempotency + locking

### Reconciliation

* Match DB records with bank records

---

## 🔹 11. Final Architecture

```
Client
  ↓
API Server (Rails)
  ↓
DB (payments)
  ↓
Payment Gateway (Stripe)
  ↓
Webhook → API → Update DB
```

---

## 🎯 One-Line Answer

> A payment system ensures reliable and consistent transactions using idempotency keys, webhooks, and strong database constraints to prevent duplicate or failed payments.
