
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



