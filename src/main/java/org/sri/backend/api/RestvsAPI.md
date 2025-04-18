# REST vs SOAP

## ✅ Key Differences

| Feature           | REST                                  | SOAP                                     |
|------------------|---------------------------------------|------------------------------------------|
| Protocol         | Architectural style                   | Protocol                                 |
| Format           | JSON, XML (typically JSON)            | XML only                                 |
| Lightweight      | Yes                                   | No                                       |
| Contract         | Optional, flexible                    | Strict (via WSDL)                        |
| Security         | HTTPS, OAuth                          | WS-Security, built-in mechanisms         |
| Use Case         | Web/mobile APIs                       | Enterprise systems (banking, insurance)  |

---

## 🧠 Analogy

- **REST** is like ordering fast food—quick, simple, flexible.
- **SOAP** is like a formal dinner reservation—strict process, more guarantees.

---

## 🎯 When to Use

- **REST**: Public APIs, mobile/web apps (e.g., mobile banking)
- **SOAP**: Enterprise integrations, legacy systems, or when security/reliability is critical

---

## 📚 Sample Interview Answer

> REST is a lightweight, stateless architecture using HTTP verbs like GET, POST, PUT, DELETE. It commonly uses JSON and is easy to work with in modern applications.
>
> SOAP is a protocol with a strict XML structure and predefined WSDL contracts. It supports advanced features like WS-Security and is preferred in systems requiring strong guarantees, like core banking.
>
> I’d use REST for flexibility in mobile apps and SOAP for guaranteed delivery or enterprise system contracts.
