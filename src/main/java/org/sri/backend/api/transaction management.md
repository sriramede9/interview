# 📘 Spring Boot Transaction Management - Explained with Analogies & Use Cases

## 🧠 Purpose

This document helps Java/Spring Boot developers understand the powerful `@Transactional` annotation, isolation levels, propagation types, and row-level locking in a **simple, intuitive way**—with **real-world analogies**, **interview insights**, and **usage scenarios**.

---

## 🔁 @Transactional Overview

```java
@Transactional(
    isolation = Isolation.SERIALIZABLE,
    propagation = Propagation.REQUIRED,
    timeout = 30
)
```

The `@Transactional` annotation ensures that a method (or a block of code) executes **atomically**—all operations either succeed or fail together.

---

## 🔒 Isolation Levels (Concurrency Control)

**What it is:** Determines how transactions **see data modified by others** before they commit.

| Isolation Level | Prevents | Analogy | When to Use |
|-----------------|----------|---------|-------------|
| `READ_UNCOMMITTED` | Nothing | Reading someone’s draft before it's finished | Almost never, not safe |
| `READ_COMMITTED` | Dirty reads | You read a published article, not a draft | Most CRUD apps |
| `REPEATABLE_READ` | Dirty + non-repeatable reads | Taking a snapshot of a webpage | For consistent reads |
| `SERIALIZABLE` | Everything (incl. phantom reads) | Only one person allowed in the vault at a time | **Money transfers, bookings, orders** |

📌 **SERIALIZABLE** is the safest but most restrictive. Use for **critical business logic** (e.g., bank transfers, seat reservations).

---

## 🔁 Propagation Types (Nested Transactions)

**What it is:** Controls **how nested transactional methods behave**.

| Propagation | Behavior | Analogy | When to Use |
|-------------|----------|---------|-------------|
| `REQUIRED` | Join existing or start new | You join a meeting if one is ongoing | **Default**, most common |
| `REQUIRES_NEW` | Always start new | Start a new meeting even if you're in one | **Logging, audit, notifications** |
| `NESTED` | Create sub-transaction (savepoint) | Sub-meeting that can roll back separately | Partial rollback cases |

🧠 **Tip:** Use `REQUIRES_NEW` when you don’t want rollback of the main logic to affect side-logic like emails or logs.

---

## 🔐 Row-Level Locking with `findByNumberLocked`

**Goal:** Prevent concurrent modifications to the same row.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT a FROM Account a WHERE a.number = :number")
Optional<Account> findByNumberLocked(@Param("number") String number);
```

- Locks the selected row until the transaction completes.
- Prevents **race conditions** in operations like transferring funds or updating inventory.

### 🎭 Analogy:

Imagine putting a **"Do Not Touch" sign** on milk in a shared fridge. Others must wait till you’re done.

---

## ✅ Use Case Summary

| Scenario | Use This | Why |
|----------|----------|-----|
| Transfer money, book seat | `SERIALIZABLE`, `findByNumberLocked` | Prevent overdrawn balances, double-booking |
| Write to DB and log failure | `REQUIRES_NEW` for logger | Keep logs even if main flow fails |
| Audit trail, partial rollback | `NESTED` | Allow rollback of sub-tasks only |
| Regular form submission | `READ_COMMITTED`, `REQUIRED` | Safe and efficient default |

---

## 💬 Sample Interview Questions

- How does Spring handle concurrent transactions?
- What’s the difference between `REQUIRED` and `REQUIRES_NEW`?
- How do you prevent two users from over-drafting an account at the same time?

---

## 🧪 Sample Method with Best Practices

```java
@Service
@Transactional(isolation = Isolation.SERIALIZABLE, propagation = Propagation.REQUIRED)
public class TransferService {

    @Autowired
    private AccountRepository accountRepo;

    public TransferResponse processTransfer(TransferRequest request) {
        Account source = accountRepo.findByNumberLocked(request.sourceAccount())
            .orElseThrow(() -> new AccountNotFoundException());

        Account target = accountRepo.findByNumber(request.targetAccount())
            .orElseThrow(() -> new AccountNotFoundException());

        if (source.getBalance().compareTo(request.amount()) < 0) {
            throw new InsufficientFundsException();
        }

        source.debit(request.amount());
        target.credit(request.amount());

        accountRepo.saveAll(List.of(source, target));

        return new TransferResponse("Success");
    }
}
```

---

## 📎 Bonus Tips

- Always annotate service methods with `@Transactional` instead of repository layer.
- Be cautious with **timeouts**—long transactions block DB resources.
- Lock only **what you need** to avoid bottlenecks.
