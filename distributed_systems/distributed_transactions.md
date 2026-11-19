# Distributed Payment Transactions (Interview Ready)

**Question: How would you design a payment system where money is debited from one bank, but crediting another bank fails, considering both banks have separate databases?**

## 1. Approach

Since the sender's bank and receiver's bank maintain separate databases, we cannot use a single ACID database transaction across both systems.

I would implement the **Saga Pattern** with a central Payment Service to coordinate the transaction.

**Technology Stack:**
- **Ruby on Rails:** Payment APIs, business logic and bank integrations.
- **PostgreSQL:** Payment records, transaction states, ledger and Outbox.
- **Apache Kafka:** Asynchronous communication between services.
- **Karafka:** Kafka integration and event consumption in Ruby.

## 2. Payment Flow

1. **Initiate Payment:** User sends a payment request through the frontend to the Rails API.
2. **Create Record:** Rails creates a payment record with a unique payment ID and `INITIATED` status in PostgreSQL.
3. **Transactional Outbox:** In the same database transaction, Rails creates an Outbox event to ensure reliable event publishing.
4. **Publish Event:** An Outbox Publisher sends the payment event to Kafka.
5. **Debit Sender:** A payment consumer processes the event and initiates the debit operation at the sender's bank.
6. **Credit Receiver:** If debit succeeds, initiate the credit operation at the receiver's bank.
7. **Success:** If credit succeeds, update the payment status to `COMPLETED` in PostgreSQL.
8. **Failure:** If credit definitively fails, initiate a compensating transaction to reverse the sender's debit.
9. **Refund:** If reversal succeeds, mark the payment as `REFUNDED`. Otherwise, keep it as `REVERSAL_PENDING` and retry.
10. **Unknown Outcome:** If the receiver's bank times out, verify its actual transaction status before retrying or initiating a reversal.

## 3. Architecture Flow

```
             Frontend (User 1)
                     |
                     | POST /api/payments
                     v
             Rails Payment API
                     |
                     v
                PostgreSQL
              /             \\
      Payments Table     Outbox Table
              |             |
              |        Outbox Publisher
              |             |
              |             v
              |           Kafka
              |             |
              |             v
              |       Payment Consumer
              |             |
              |       Debit Sender Bank
              |             |
              |       Debit Successful?
              |          /       \\
              |        No         Yes
              |        |           |
              |      FAILED        v
              |             Credit Receiver Bank
              |                    |
              |             Credit Successful?
              |                /       \\
              |              Yes        No
              |               |          |
              |               v          v
              |          COMPLETED   Compensating
              |                       Transaction
              |                            |
              |                       Refund Sender
              |                            |
              |                     REFUNDED or
              |                  REVERSAL_PENDING
              |                            |
              +----------------------------+
                           |
                           v
                      PostgreSQL
                           |
                           v
                    Rails Status API
                           |
                           v
                     Frontend User
                  (Final Payment Status)
```

## 4. How Does the Frontend Get the Result?

The initial API should not wait for the complete bank-to-bank transfer.

**Initial Request:**

`POST /api/payments`

Rails immediately responds with `202 Accepted`:

```json
{
  "payment_id": "PAY-123",
  "amount": 10000,
  "status": "PROCESSING",
  "message": "Your payment is being processed."
}
```

The frontend can obtain the final status in two ways:

- **Polling:** Frontend periodically calls `GET /api/payments/PAY-123`. Rails fetches the latest status from PostgreSQL.
- **WebSockets:** Rails uses Action Cable to push real-time payment status updates to the frontend.

**Example: Final Successful Response**

```json
{
  "payment_id": "PAY-123",
  "amount": 10000,
  "status": "COMPLETED"
}
```

**Example: Refund Pending Response**

```json
{
  "payment_id": "PAY-123",
  "amount": 10000,
  "status": "REVERSAL_PENDING",
  "message": "Your payment reversal is being processed."
}
```

## 5. Important Mechanisms

- **Saga Pattern:** Coordinates distributed transactions using local transactions and compensating actions.
- **Idempotency:** Prevents duplicate debits or credits during retries.
- **Transactional Outbox:** Ensures events are not lost between database commits and Kafka publishing.
- **Retry Mechanism:** Retries temporary failures using exponential backoff.
- **Reconciliation:** Periodically compares transaction records with both banks and resolves inconsistent or pending payments.
- **Double-entry Ledger:** Maintains accurate and auditable financial records.

## 6. Final Answer

Since both banks maintain independent databases, I cannot rely on a single ACID transaction. I would use the Saga Pattern to coordinate the payment workflow and compensating transactions to handle failures.

Rails would handle the payment APIs and business logic, PostgreSQL would maintain payment states and financial records, and Kafka would handle asynchronous event communication.

I would also implement idempotency, transactional outbox, retries and reconciliation to ensure reliable processing.

The frontend would initially receive a payment ID with a `PROCESSING` status and obtain the final result through polling or WebSockets.

**Goal:** Ensure that money is either successfully transferred or safely returned to the sender, without losing or duplicating funds.


The Transactional **Outbox Pattern** ensures reliable event publishing by storing business data and the corresponding event in the same database transaction. A separate publisher reads the Outbox table and sends events to a message broker. This prevents events from being lost if the message broker is temporarily unavailable.