<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/86bcbaa5-33c9-49ec-84b5-4c9287c88f7f" />

# 🧪 Feature Specification: Lab Booking & Cancellation (Contamination Control)

## 🔹 Feature Overview

This feature allows a **lab assistant** to create and cancel lab bookings to prevent contamination between students by enforcing cleaning gaps and access control.

---

## 🔹 Actors

* **Lab Assistant** → can create and cancel bookings
* **Student** → uses booked lab but cannot override safety rules

---

## 🔹 Inputs

* `lab_id`
* `time_slot_start`
* `time_slot_end`
* `requester_role` (assistant or student)
* `current_time`
* existing bookings list

---

## 🔹 Outputs

* `status`:

  * `"booked"`
  * `"cancelled"`
  * `"refused"`
* `reason` (if refused):

  * `"lab closed"`
  * `"conflict with existing booking"`
  * `"insufficient cleaning gap"`
  * `"not authorized"`
  * `"cannot cancel past booking"`

---

## 🔹 Rules

### 1. Booking Authorization Rule

* Only a **lab assistant** can create or cancel bookings for contamination control
* If requester is not assistant → refuse

---

### 2. Lab Availability Rule

* A lab marked as **closed** cannot be booked
* Applies to everyone (even assistant)

---

### 3. Conflict Rule

* Two bookings cannot overlap on the same lab
* If overlap detected → refuse

---

### 4. Cleaning Gap Rule (🔥 key contamination rule)

* There must be at least **15 minutes gap** between bookings
* This gap is used for cleaning and contamination prevention
* If gap < 15 minutes → refuse

---

### 5. Cancellation Rule

* A lab assistant can cancel any booking
* Cancellation must happen **before the booking start time**
* If current_time ≥ start_time → refuse

---

## 🔹 Normal Flow

### Booking

1. Assistant requests booking
2. System checks:

   * authorization ✔️
   * lab open ✔️
   * no conflict ✔️
   * cleaning gap ✔️
3. Booking is created
4. Return: `status = "booked"`

---

### Cancellation

1. Assistant requests cancellation
2. System checks:

   * authorization ✔️
   * booking not started ✔️
3. Booking is cancelled
4. Return: `status = "cancelled"`

---

## 🔹 Invalid / Refusal Flow

Booking is refused if:

* requester is not assistant → `"not authorized"`
* lab is closed → `"lab closed"`
* time overlaps → `"conflict with existing booking"`
* cleaning gap < 15 min → `"insufficient cleaning gap"`

Cancellation is refused if:

* requester is not assistant → `"not authorized"`
* booking already started → `"cannot cancel past booking"`

---

## 🔹 Edge Cases

* Booking starts **exactly 15 minutes after previous booking ends** → ✅ allowed
* Booking starts **14 min 59 sec after previous booking** → ❌ refused
* Cancellation at **exact start time** → ❌ refused
* Back-to-back bookings without gap → ❌ refused

---

## 🔥 Why this is a strong spec (important for your assignment)

* Clear **rules** → easy to test
* Defined **outputs + reasons** → perfect for oracles
* Includes:

  * normal cases
  * edge cases
  * invalid cases

---

Got it — now you’re moving to **S7 (decision / failures behind the design)**, not just listing rules like in S5.

In S7, you don’t just restate rules — you identify **“predictable failures” (unhappy paths)** and explain how your system **defends against them**.

---

# 🧪 S7 – Three Predictable Failures (Unhappy Paths)

**Feature: Lab Booking & Cancellation (Contamination Control)**

---

## 🔴 Failure 1: Contamination due to insufficient cleaning gap

### ❗ What can go wrong

A booking is scheduled too close to another booking (less than 15 minutes gap), so the lab is not properly cleaned → contamination risk between students.

### 🧠 Why it happens

* Users try to maximize lab usage
* System might allow near-overlapping bookings if not strictly validated
* Off-by-one errors at boundary (e.g. 14:59 vs 15:00)

### 🛡️ Defense (Design Decision)

* Enforce strict **Cleaning Gap Rule (≥ 15 minutes)**
* Validate both:

  * previous booking end → new start
  * new end → next booking start
* Treat **15:00 exactly as valid**, anything below → refused

### ✅ Result

System refuses unsafe bookings with:

* `status = "refused"`
* `reason = "insufficient cleaning gap"`

---

## 🔴 Failure 2: Unauthorized booking or cancellation

### ❗ What can go wrong

A student (or unauthorized user) tries to:

* create bookings
* cancel bookings

This could break contamination control policies.

### 🧠 Why it happens

* Missing role validation
* UI restrictions bypassed (API calls, etc.)

### 🛡️ Defense (Design Decision)

* Enforce **role-based access control at backend**
* Only allow `requester_role == "assistant"`
* Never rely only on frontend checks

### ✅ Result

System refuses with:

* `status = "refused"`
* `reason = "not authorized"`

---

## 🔴 Failure 3: Cancellation too late (lab already in use)

### ❗ What can go wrong

A booking is cancelled **after it has already started**, causing:

* confusion in lab usage
* possible contamination (students already inside)

### 🧠 Why it happens

* Time comparison errors
* Delayed requests
* System allows cancellation at or after start time

### 🛡️ Defense (Design Decision)

* Enforce strict time rule:

  * `current_time < start_time` → allowed
  * `current_time ≥ start_time` → refused
* Use consistent time source (server time)

### ✅ Result

System refuses with:

* `status = "refused"`
* `reason = "cannot cancel past booking"`





* ✅ Happy path 
* 📊 Category column
* 🧠 One **tautological test**
* 🔁 One **oracle mirroring implementation**
* 🚫 One **oracle-free assertion**

---

# 🧪 Test Design: Lab Booking & Cancellation System

| Test ID | Category                    | Scenario                                  | Input                                                            | Oracle Type       | Expected Result                                 |
| ------- | --------------------------- | ----------------------------------------- | ---------------------------------------------------------------- | ----------------- | ----------------------------------------------- |
| T1      | Happy Path (Boundary Gap)   | Booking with exact 15-minute cleaning gap | assistant, lab open, previous ends 10:00, new starts 10:15–11:00 | Spec-based oracle | `"booked"`                                      |
| T2      | Conflict Failure            | Overlapping booking attempt               | previous 10:00–11:00, new 10:30–11:30                            | Rule oracle       | `"refused"`, `"conflict with existing booking"` |
| T3      | Unauthorized Access         | Student tries to book                     | requester_role = student                                         | Rule oracle       | `"refused"`, `"not authorized"`                 |
| T4      | Cancellation Invalid Timing | Cancel after start time                   | current_time = 10:05, start_time = 10:00                         | Rule oracle       | `"refused"`, `"cannot cancel past booking"`     |

---

# 🔥 Required Special Test Types

## 1️⃣ 🟢 Happy Path ONLY GAP (Boundary Case)

### ✔ Test

T1 – Booking exactly at 15-minute cleaning gap

### Why it matters

This ensures the system correctly handles the **critical contamination boundary rule**:

> 15 minutes = allowed
> 14:59 = refused

### Expected behavior

```text
status = "booked"
```

---

## 2️⃣ 🧠 Tautological Test (⚠️ Weak but required)

This test *mirrors the system output without meaningful checking*.

### ✔ Test

T1 rewritten as tautological assertion:

```text
assert system.book(...) == system.book(...)
```

OR in API form:

```text
assert response.status == response.status
```

### Why it is tautological

* It does NOT validate correctness
* It only compares output to itself
* It would always pass even if system is broken

👉 This demonstrates a **bad oracle design on purpose**

---

## 3️⃣ 🔁 Oracle Mirroring Implementation Test

This uses a **second implementation identical to the system logic**.

### ✔ Idea

We re-run the same booking logic in a “reference function”:

```text
expected = bookingService.book(lab_id, time_slot, requester_role, current_time)
actual = system.book(...)
assert actual == expected
```

### Why this is special

* Oracle is **not independent**
* It mirrors the implementation logic
* It detects regressions but not logic errors shared by both

---

## 4️⃣ 🚫 Oracle-Free Assertion (Property-Based)

Instead of checking exact outputs, we check a **system invariant**.

### ✔ Test (after any successful booking)

```text
ASSERT:
for every booking in system:
    no overlap exists AND
    gap between consecutive bookings >= 15 minutes
```

### Why this is oracle-free

* No expected value is defined
* It checks a **global property of the system state**
* Works even if we don’t know exact outputs

### Example assertion

```text
assert all(
    gap(b1.end, b2.start) >= 15
    for all consecutive bookings
)
```

---

# 🧠 Summary of What You Demonstrated

| Type                    | What it proves                     |
| ----------------------- | ---------------------------------- |
| Happy Path (15-min gap) | Correct boundary handling          |
| Tautological Test       | Demonstrates weak/invalid testing  |
| Oracle Mirroring        | Shows dependency on implementation |
| Oracle-Free Assertion   | Strong system invariant validation |


