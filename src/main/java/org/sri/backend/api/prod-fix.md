# 🛠️ Production-Level Bug Fixes — Interview-Ready Real Examples

This document outlines real-world production bugs I’ve resolved, framed for behavioral and technical interview questions such as:

> **"Tell me about a production-level issue you fixed recently?"**

---

## 🅰️ Race Condition in Booking System (Concurrency Bug)

### 🧩 Problem
User bookings were **randomly failing or double-booking** in our Parking Spot app.

### 🔍 Root Cause
Race condition: Multiple users could `findById()` and `update()` the same parking spot simultaneously without locking, causing **duplicate bookings**.

### 🛠️ Fix

- **Pessimistic Locking in Spring JPA:**
    ```
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    Optional<ParkingSpot> findById(Long id);
    ```
- Later scaled to use **Redis distributed lock (Redisson)** for non-blocking, high-concurrency protection.

#### ⚙️ Tools
- Spring Boot
- JPA Locking
- Redis (Redisson)

#### 💡 Takeaway
> _"Even simple CRUD flows need concurrency control in multi-user systems."_

---

## 🅱️ Integration Failure Between Microservices (Silent Email Failures)

### 🧩 Problem
Email notifications weren’t being sent post-payment, with no error logged by the Payment Service.

### 🔍 Root Cause
REST endpoint path in Email Service changed from `/send` → `/sendEmail`, but Payment Service wasn’t updated. No contract validation existed.

### 🛠️ Fix

- Added **Resilience4j** for:
    - Circuit breaking
    - Retry logic
    - Fallback to queue emails
- Implemented **Spring Cloud Contract** for API contract enforcement.
- Built **Postman collections** and CI tests for end-to-end validation.

#### ⚙️ Tools
- Spring Boot + Feign Client
- Resilience4j
- Spring Cloud Contract
- ELK, Prometheus, Zipkin

#### 💡 Takeaway
> _"Not all integration failures throw exceptions. Observability and contract testing saved us."_

---

## 🆎 Memory Leak in API Response Caching

### 🧩 Problem
Spring Boot service was crashing periodically with `OutOfMemoryError`.

### 🔍 Root Cause
Used an unbounded `ConcurrentHashMap` to cache responses — it grew indefinitely with each unique request.

### 🛠️ Fix

- Replaced with **Caffeine Cache** with size and TTL limits:
    ```
    Caffeine.newBuilder()
        .maximumSize(10000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .build();
    ```
- Used heap dump analysis (Eclipse MAT, VisualVM)
- Set alerts when heap > 80%

#### ⚙️ Tools
- Caffeine Cache
- jmap, Eclipse MAT
- Java Flight Recorder, VisualVM

#### 💡 Takeaway
> _"Every in-memory store must have an expiration or size policy — unbounded collections are time bombs."_

---

## 🆑 Database Deadlock in Transaction Processing

### 🧩 Problem
App hung when processing transactions due to database-level deadlocks.

### 🔍 Root Cause
Thread A: locked Table 1 → then tried Table 2  
Thread B: locked Table 2 → then tried Table 1  
Classic circular wait deadlock.

### 🛠️ Fix

- Standardized lock acquisition order across the app
- Used **Spring Retry** for retrying failed transactions
- Monitored using MySQL’s `SHOW ENGINE INNODB STATUS`

#### ⚙️ Tools
- MySQL logs
- Spring Retry
- Spring `@Transactional`

#### 💡 Takeaway
> _"Deadlocks happen when locking order isn’t controlled. Always lock tables in the same sequence."_

---

## 🔁 Bonus: Logging + Fallback Logic with Resilience4j

### ✨ Use Case
Calling `EmailService` from `PaymentService` using RestTemplate or Feign

### 🧪 Problem
`EmailService` was flaky — sometimes down or slow

### 🔧 Fix

Used **Resilience4j**:
```

@Retry(name = "emailRetry", fallbackMethod = "fallbackEmail")
@CircuitBreaker(name = "emailCircuitBreaker", fallbackMethod = "fallbackEmail")
public String sendEmail(String msg) {
return restTemplate.postForEntity("http://email-service/send", msg, String.class).getBody();
}

public String fallbackEmail(String msg, Throwable t) {
log.error("Fallback triggered. Reason: {}", t.getMessage());
return "Email queued";
}

```

**Config (`application.yml`):**
```

resilience4j:
retry:
instances:
emailRetry:
max-attempts: 3
wait-duration: 1s
circuitbreaker:
instances:
emailCircuitBreaker:
failure-rate-threshold: 50
wait-duration-in-open-state: 10s

```

#### 💬 Interview Gold Quote
> _“In distributed systems, retries + fallback + circuit breaker protect us from cascading failures. I always add these for downstream calls like payments and emails.”_

---

## 📚 Summary of Lessons

| Category         | Key Lesson                                                      |
|------------------|-----------------------------------------------------------------|
| Concurrency      | Use locking or atomic operations for shared resources           |
| Integration      | Add observability + contract testing                            |
| Memory Mgmt      | Always expire caches or bound their size                        |
| DB Transactions  | Enforce lock order + retry logic                                |
| Resilience       | Combine retry, fallback, and circuit breaker patterns           |

---

## ✅ Prepared With

- Spring Boot, Spring Data JPA
- Redis, Redisson
- Caffeine Cache
- Resilience4j
- Spring Cloud Contract
- Prometheus, Grafana, Zipkin, ELK
- Java Flight Recorder, VisualVM, Eclipse MAT
- Postman, CI/CD pipelines
