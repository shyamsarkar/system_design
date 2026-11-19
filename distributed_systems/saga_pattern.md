
## Saga Pattern

**Definition:** The Saga Pattern is a distributed transaction management pattern that breaks a large business transaction into multiple independent local transactions. If any step fails, compensating transactions are executed to reverse the effects of previously completed steps.

### Saga Transaction Flow

**Example: Bank Transfer (₹1,000)**

1. **Debit ₹1,000 from Bank A** → ✅ Success
2. **Credit ₹1,000 to Bank B** → ❌ Failure
3. **Refund ₹1,000 to Bank A** → 🔄 Compensation

**Flow:**

```text
Debit Bank A
     |
     v
Credit Bank B
     |
     v
  Failed
     |
     v
Refund Bank A (Compensation)
     |
     v
Transaction Failed / Refunded
```

**Note:** Compensation is a new business transaction, not a database rollback.

### Types of Saga

1. **Choreography:** Services coordinate with each other through events. There is no central controller.
2. **Orchestration:** A central Saga Orchestrator controls the workflow and triggers compensating actions on failure.

### Important Concepts

- **Local Transaction:** Transaction committed within an individual service/database.
- **Compensation:** Reverses the business effect of a previously completed transaction.
- **Idempotency:** Prevents duplicate operations when requests are retried.
- **Eventual Consistency:** Services eventually reach a consistent state.

### Payment Considerations

- Use unique transaction IDs and idempotency keys.
- Retry temporary failures.
- Verify transaction status before initiating a refund.
- Use reconciliation jobs to recover uncertain transactions.
- Maintain an audit trail.

### Interview Takeaway

Saga manages distributed transactions without requiring a single ACID transaction across multiple databases. It uses local transactions, retries, and compensating transactions to achieve eventual consistency.
