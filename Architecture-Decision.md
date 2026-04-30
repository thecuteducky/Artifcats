<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/d1cd1a45-4cec-45e5-91fb-af1178fe1e5e" />

# S7 – Architecture Decision Record (ADR)

## Campus Lab Booking System

### Title

**ADR-01: Authentication Method for Campus Lab Booking System**

### Status

**Accepted**

---

## Context

The Campus Lab Booking System needs a secure way for students and staff to log in and access their bookings.

This decision is important because authentication affects:

* system security
* ease of maintenance
* user experience
* scalability
* development complexity

The two possible options are:

* **Option A:** Server-side session lookup
* **Option B:** Stateless JWT authentication

The system will be used by many students during busy lab hours, so the chosen solution should be secure and easy to manage.

---

## Options Comparison

---

### Option A: Server-side Session Lookup

### Pros

1. **More secure control**

   * Sessions are stored on the server, so they can be invalidated easily.

2. **Easy logout and session management**

   * The admin can directly terminate active sessions.

3. **Simple for smaller systems**

   * Good for university projects and internal systems.

### Cons

1. **Requires server memory/storage**

   * The server must store all active sessions.

2. **Less scalable**

   * If many students use the system at once, session storage may become heavier.

---

### Option B: Stateless JWT

### Pros

1. **Highly scalable**

   * No need to store session data on the server.

2. **Fast performance**

   * Each request contains its own authentication token.

3. **Good for distributed systems**

   * Works well if the system later grows into microservices.

### Cons

1. **Harder logout control**

   * Tokens remain valid until expiration.

2. **Higher security risk if token is stolen**

   * A leaked token can be reused.

---

## Decision

For the Campus Lab Booking System, I choose **Option A: Server-side session lookup**.

This option is better because the system is a university internal platform and security plus easy management are more important than large-scale scalability.

Since students may use shared university computers, having the ability to quickly terminate sessions is very useful.

Also, this option is simpler for the development team and easier to maintain.

---

## Consequences

As a result of this decision:

* the server must store session information
* memory usage may increase during peak hours
* logout and session expiration are easier to manage
* future scaling may require session replication

### Reversibility Note

This is a **two-way door decision**, because the authentication method can later be changed to JWT if the system grows.

---

# Trade-off Card

| Criterion       | Option A (Session) | Option B (JWT) | Why this criterion mattered here                                                          |
| --------------- | -----------------: | -------------: | ----------------------------------------------------------------------------------------- |
| Security        |                  5 |              3 | Student accounts and booking data must be protected, especially on shared campus devices. |
| Complexity      |                  4 |              3 | The team needs a solution that is simple to implement and debug.                          |
| Scalability     |                  3 |              5 | The system may grow later, but current usage is moderate.                                 |
| Maintainability |                  4 |              3 | Easy session control helps future maintenance and issue fixing.                           |
| Reversibility   |                  4 |              4 | Authentication can be changed later if system requirements change.                        |

