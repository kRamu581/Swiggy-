# 🍔 Food Delivery Platform on ServiceNow

## Phase 5 — Delivery Management

> Building a Swiggy-inspired food delivery platform using ServiceNow.

![Phase](https://img.shields.io/badge/Phase-05-orange)
![Platform](https://img.shields.io/badge/Platform-ServiceNow-green)
![Module](https://img.shields.io/badge/Module-Delivery-blue)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---

## 📌 Overview

Phase 5 carries each ready order from the restaurant to the customer.

When a restaurant marks an order **Ready**, the system creates a delivery record and assigns a delivery partner. The partner accepts the job, picks up the food, and delivers it. The customer confirms receipt with a one-time password (OTP) that is verified securely on the server. Failed deliveries are handled through a defined recovery process.

This phase builds on:

- **Phase 1:** Application Foundation, Database Design and Security.
- **Phase 2:** Restaurant Management and Menu Management.
- **Phase 3:** Cart, Checkout and Order Creation.
- **Phase 4:** Restaurant Order Management.

## 🎯 Objectives

- [ ] Create a delivery record when an order becomes Ready.
- [ ] Implement delivery partner availability.
- [ ] Implement delivery partner assignment.
- [ ] Allow partners to accept or decline delivery requests.
- [ ] Reassign deliveries that are declined or time out.
- [ ] Implement pickup confirmation.
- [ ] Implement a delivery state model with validated transitions.
- [ ] Implement secure OTP-based delivery verification.
- [ ] Handle failed deliveries.
- [ ] Synchronize delivery status with order status.
- [ ] Build the delivery partner dashboard.
- [ ] Build customer delivery tracking.
- [ ] Restrict partners to their assigned deliveries.
- [ ] Test success, failure, reassignment and security scenarios.

## 🏗️ Technology Stack

| Component | Technology |
|---|---|
| Platform | ServiceNow |
| Application | Scoped Application |
| Database | Custom ServiceNow Tables |
| UI | Service Portal or UI Builder |
| Client Logic | JavaScript |
| Server Logic | Script Includes and GlideRecord |
| Validation | Server-side validation and Business Rules |
| Notifications | ServiceNow Notifications and Events |
| Automation | Flow Designer and Scheduled Jobs |
| Security | Roles and ACLs |
| Version Control | GitHub |

## 🗂️ Data Model

Phase 5 uses and extends the following tables:

- Food Order
- Food Delivery
- Food Restaurant
- Food Customer
- Food Notification
- Food Order Status History

It also introduces two new tables for partner profiles and delivery history.

### 1. Food Delivery Partner (New)

**Purpose:** Maintain delivery partner profiles and availability.

| Field | Type | Description |
|---|---|---|
| Partner Number | Auto-number / String | Unique partner reference |
| User | Reference → User | Associated ServiceNow user |
| Name | String | Display name |
| Phone | Phone Number / String | Contact number |
| Vehicle Type | Choice | Bicycle, Scooter, Motorcycle, Other |
| Availability | Choice | Offline, Available, On Delivery |
| Active Deliveries | Integer | Current assignment count |
| Active | True/False | Account status |

**Rules:**

- A partner must be active and **Available** to receive assignments.
- A partner cannot exceed the configured maximum concurrent deliveries.
- Only the partner or an administrator can change availability.

### 2. Food Delivery (Extended)

Extend the Phase 1 **Food Delivery** table.

| Field | Type | Description |
|---|---|---|
| Delivery Number | Auto-number / String | Unique delivery reference |
| Order | Reference → Food Order | Related order |
| Delivery Partner | Reference → User | Assigned rider |
| Status | Choice | Unassigned, Assigned, Accepted, Picked Up, Delivered, Failed |
| Assigned At | Date/Time | Assignment timestamp |
| Accepted At | Date/Time | Partner acceptance timestamp |
| Pickup Time | Date/Time | Pickup timestamp |
| Delivery Time | Date/Time | Completion timestamp |
| Assignment Deadline | Date/Time | Time by which the partner must respond |
| Assignment Attempts | Integer | Number of assignment attempts |
| Delivery OTP Hash | String | Secure verification value; never store a plain OTP |
| OTP Expires At | Date/Time | OTP expiry time |
| OTP Failed Attempts | Integer | Count of incorrect OTP entries |
| Failure Reason | Choice | Customer unreachable, Wrong address, Customer refused, Vehicle issue, Other |
| Failure Notes | String | Optional detail |

### 3. Delivery State Model

| Status | Meaning |
|---|---|
| Unassigned | Delivery created, no partner assigned |
| Assigned | Partner offered the delivery, awaiting response |
| Accepted | Partner accepted and is heading to the restaurant |
| Picked Up | Partner collected the food |
| Delivered | Customer verified and received the order |
| Failed | Delivery could not be completed |

#### Permitted Transitions

```text
Unassigned ──> Assigned
Assigned   ──> Accepted | Unassigned (declined or timed out)
Accepted   ──> Picked Up | Unassigned (partner withdraws before pickup)
Picked Up  ──> Delivered | Failed
```

| From | To | Allowed Actor |
|---|---|---|
| Unassigned | Assigned | System or admin |
| Assigned | Accepted | Assigned partner |
| Assigned | Unassigned | Assigned partner (decline) or system (timeout) |
| Accepted | Picked Up | Assigned partner |
| Accepted | Unassigned | Assigned partner or admin |
| Picked Up | Delivered | Assigned partner, after OTP verification |
| Picked Up | Failed | Assigned partner or admin |

Any transition not listed must be rejected by the server.

### 4. Order and Delivery Status Mapping

Keep the order status synchronized with delivery progress through server-side logic.

| Delivery Event | Order Status |
|---|---|
| Delivery created | Ready |
| Partner picks up | Picked Up |
| Delivery verified | Delivered |
| Delivery failed | Delivery Failed (pending resolution) |

Add **Delivery Failed** to the order status choices if not already present. Every order status change must continue to create a Food Order Status History record.

### 5. Food Delivery Event (New)

**Purpose:** Maintain an auditable timeline of delivery events.

| Field | Type | Description |
|---|---|---|
| Delivery | Reference → Food Delivery | Related delivery |
| Event Type | Choice | Created, Offered, Accepted, Declined, Timed Out, Reassigned, Picked Up, OTP Failed, Delivered, Failed |
| Partner | Reference → User | Partner involved, if any |
| Occurred At | Date/Time | Timestamp |
| Notes | String | Optional context |

**Rules:**

- Records are created by server-side logic only.
- Records cannot be edited or deleted by regular users.

## 🚚 Delivery Creation

When an order reaches **Ready**:

1. Verify that no delivery record already exists for the order.
2. Create a Food Delivery record with status **Unassigned**.
3. Generate the delivery OTP (see the OTP section).
4. Create a **Created** delivery event.
5. Start the assignment process.

Creation must be idempotent. Repeated triggers must not create duplicate deliveries.

## 🧭 Partner Assignment

### Assignment Process

1. Find active partners with **Available** status and capacity.
2. Rank candidates using the configured strategy.
3. Offer the delivery to the selected partner, setting the delivery to **Assigned** with a response deadline.
4. Notify the partner.
5. If the partner declines or does not respond, return the delivery to **Unassigned** and offer it to the next candidate.
6. If no partner accepts after the configured number of attempts, escalate to an administrator.

### Selection Strategies

Choose a strategy appropriate to the project and keep it configurable.

| Strategy | Description |
|---|---|
| Manual | An administrator assigns partners |
| Round Robin | Offers rotate across available partners |
| Least Loaded | Prefers the partner with the fewest active deliveries |
| Nearest | Uses location data, if collected |

Location-based assignment requires collecting partner location data. If implemented, collect only what is needed, define a retention policy, and inform users. A simple strategy such as Least Loaded is recommended for this phase.

### Concurrency Controls

- Prevent two partners from accepting the same delivery.
- Apply the acceptance atomically, checking the delivery is still **Assigned** to that partner.
- Prevent a partner from exceeding the maximum concurrent deliveries.
- Make the assignment job idempotent.

## ✅ Delivery Partner Actions

### Accept Delivery

1. Verify the user is the assigned partner.
2. Verify the delivery is **Assigned** and the deadline has not passed.
3. Set the delivery to **Accepted** and record the time.
4. Update the partner's availability and active delivery count.
5. Create a delivery event and notify the restaurant and customer.

### Decline Delivery

1. Verify the user is the assigned partner.
2. Return the delivery to **Unassigned**.
3. Record the event and trigger reassignment.

### Confirm Pickup

1. Verify the user is the assigned partner and the delivery is **Accepted**.
2. Verify the order is **Ready**.
3. Set the delivery to **Picked Up** and record the pickup time.
4. Update the order to **Picked Up**.
5. Notify the customer.

## 🔑 Secure Delivery Verification (OTP)

The OTP confirms that the customer received the order.

### Requirements

- Generate the OTP on the server using a cryptographically secure source.
- Store only a salted hash of the OTP. Never store the plain OTP.
- Show the OTP to the customer through the secure customer interface or notification.
- Never expose the OTP or its hash to the delivery partner.
- Set an expiry time and define a policy for regenerating an expired OTP.
- Limit incorrect attempts and lock verification after the limit is reached.
- Use constant-time comparison where available.
- Log verification failures without logging the entered code.
- Invalidate the OTP after successful use.

### Verification Flow

```text
Partner arrives at customer
        |
        v
Customer shares OTP with partner
        |
        v
Partner enters OTP in the app
        |
        v
Server validates:
  - Partner is assigned to the delivery
  - Delivery is Picked Up
  - OTP has not expired
  - Attempt limit not exceeded
  - Hash matches
        |
   +----+----+
   |         |
 Valid     Invalid
   |         |
   v         v
Delivered   Increment attempts,
            record event, show error
```

### Lockout and Recovery

- After the maximum failed attempts, block further entries and alert an administrator.
- Provide an authorized recovery path, such as support-assisted verification.
- Record all recovery actions in the delivery event timeline.

## ❌ Failed Delivery Handling

A delivery may fail when the customer is unreachable, the address is wrong, or the customer refuses the order.

### Process

1. The partner records a failure reason and notes.
2. The delivery moves to **Failed** and the order to **Delivery Failed**.
3. The customer and support team are notified.
4. A support agent reviews the case and decides the outcome.

### Possible Outcomes

| Outcome | Action |
|---|---|
| Retry delivery | Create a new delivery attempt for the same order |
| Cancel order | Cancel the order and apply the refund policy |
| Partner error | Reassign to another partner |
| Customer at fault | Close under the defined policy |

Define refund and fee policy explicitly. For sandbox payments, update the payment status consistently and never leave an inconsistent payment and order state.

Optionally create a **Food Support Case** automatically for failed deliveries.

## 🖥️ User Interface Pages

### Page 1: Delivery Partner Dashboard

Suggested route:

`/food-partner-dashboard`

Components:

- Availability toggle.
- Pending delivery offers with a countdown.
- Active deliveries.
- Accept and Decline actions.
- Confirm Pickup action.
- OTP entry for completion.
- Report Failed Delivery action.
- Delivery history.
- Empty-state, loading and error messages.

### Page 2: Delivery Details (Partner View)

Suggested route:

`/food-partner-delivery?id=<delivery_sys_id>`

Components:

- Restaurant name and pickup address.
- Order items summary.
- Delivery address and permitted contact details.
- Current status and allowed actions.
- Delivery event timeline.

Show only the customer information needed to complete the delivery, and only for deliveries assigned to the partner.

### Page 3: Delivery Tracking (Customer View)

Suggested route:

`/food-delivery-tracking?id=<order_sys_id>`

Components:

- Delivery status timeline.
- Assigned partner's first name and permitted contact method.
- Delivery OTP display.
- Estimated delivery time, if available.
- Failure message and next steps, if applicable.

### Page 4: Delivery Administration

Suggested route:

`/food-delivery-admin`

Components:

- Unassigned and delayed deliveries.
- Manual assignment and reassignment.
- Partner availability overview.
- Failed delivery queue.
- Event timeline per delivery.

These routes are illustrative. Configure the actual pages and routing in the selected Service Portal or UI Builder experience.

## 🔐 Security Requirements

### Roles

| Role | Permissions |
|---|---|
| `food_delivery_partner` | View and update only assigned deliveries; manage own availability |
| `food_customer` | View own delivery status and OTP |
| `food_restaurant_manager` | View delivery status and partner details for own restaurant's orders |
| `food_support_agent` | Review failed deliveries and perform authorized recovery |
| `food_admin` | Manage assignments, partners and configuration |

### Mandatory Security Controls

- Validate on the server that the partner is assigned to the delivery for every action.
- Enforce the delivery state transition table on the server.
- Never trust client-supplied status or partner identifiers.
- Prevent partners from accessing unassigned or other partners' deliveries.
- Never expose the OTP or hash to partners.
- Apply attempt limits and expiry to OTP verification.
- Use ACLs to restrict direct record access.
- Share only the minimum customer data a partner needs, and only during an active delivery.
- Restrict partner contact details visible to customers to what is necessary.
- Make delivery event records read-only to regular users.
- Protect logs from OTPs, addresses and phone numbers.

## ⚙️ Implementation Plan

### Step 1: Verify Phases 1 to 4

Confirm the order, delivery, notification, status history tables, roles and ACLs are in place.

### Step 2: Create the Partner Table

Create Food Delivery Partner and link it to users and the `food_delivery_partner` role.

### Step 3: Extend the Delivery Table

Add the new fields to Food Delivery and update status choices.

### Step 4: Create the Delivery Event Table

Create Food Delivery Event and restrict writes to server-side logic.

### Step 5: Define the State Models

Document the delivery transitions and the delivery-to-order status mapping.

### Step 6: Implement Delivery Creation

Trigger idempotent delivery creation when an order becomes Ready.

### Step 7: Implement Assignment

Create the assignment Script Include, selection strategy, response deadlines, and reassignment logic.

### Step 8: Implement Partner Actions

Create server-side functions for accept, decline, confirm pickup, withdraw and report failure.

### Step 9: Implement OTP Handling

Generate, hash, expire, verify and invalidate the delivery OTP with attempt limits.

### Step 10: Implement Failed Delivery Handling

Create the failure workflow, support notification, and recovery outcomes.

### Step 11: Build the Interfaces

Create the partner dashboard, delivery details, customer tracking and admin pages.

### Step 12: Configure ACLs and Notifications

Restrict access by role and assignment. Configure partner, customer, restaurant and admin notifications.

### Step 13: Test and Document

Run the testing checklist below, fix issues, and update the documentation.

## 🧪 Testing Checklist

### Delivery Creation Tests

- [ ] Delivery is created when an order becomes Ready.
- [ ] Duplicate triggers do not create duplicate deliveries.
- [ ] OTP is generated and only its hash is stored.
- [ ] A delivery event is recorded.

### Assignment Tests

- [ ] An available partner receives the offer.
- [ ] Unavailable or inactive partners are skipped.
- [ ] Partners at capacity are skipped.
- [ ] Declining reassigns the delivery.
- [ ] Timeout reassigns the delivery.
- [ ] Two partners cannot accept the same delivery.
- [ ] Escalation occurs after the maximum attempts.

### Partner Action Tests

- [ ] Accept a delivery.
- [ ] Confirm pickup.
- [ ] Pickup is refused if the order is not Ready.
- [ ] Invalid transitions are rejected.
- [ ] Order status is synchronized with delivery status.
- [ ] Status history and delivery events are recorded.

### OTP Tests

- [ ] Correct OTP completes the delivery.
- [ ] Incorrect OTP is rejected and counted.
- [ ] Verification locks after the attempt limit.
- [ ] Expired OTP is rejected.
- [ ] Used OTP cannot be reused.
- [ ] OTP and hash are never visible to the partner.
- [ ] The plain OTP does not appear in logs.

### Failed Delivery Tests

- [ ] Partner can report a failed delivery with a reason.
- [ ] Order moves to Delivery Failed.
- [ ] Support team is notified.
- [ ] Retry, cancel and reassign outcomes work.
- [ ] Payment and order states stay consistent.

### Security Tests

- [ ] Partner cannot view or update unassigned deliveries.
- [ ] Partner cannot update another partner's delivery.
- [ ] Customer cannot call partner actions.
- [ ] Modified requests cannot force an invalid status.
- [ ] Customers cannot view other customers' deliveries.
- [ ] Direct record access is blocked by ACLs.

## 📦 Expected Deliverables

At the end of Phase 5, the application should contain:

1. Food Delivery Partner table and availability management.
2. An extended Food Delivery table.
3. Food Delivery Event table.
4. A delivery state model with validated transitions.
5. Idempotent delivery creation.
6. Partner assignment, reassignment and escalation.
7. Accept, decline and pickup workflows.
8. Secure hashed OTP verification with expiry and attempt limits.
9. Failed delivery handling and recovery outcomes.
10. Delivery partner dashboard and customer tracking.
11. ACLs and server-side authorization checks.
12. Updated documentation and test results.

## 🚀 Next Phase

### Phase 6 — Reviews, Ratings & Customer Support

The next phase will implement:

- Review and rating submission for delivered orders.
- Restaurant rating aggregation.
- Review validation and moderation.
- Support case creation and tracking.
- Support agent workflows and SLAs.
- Refund and complaint handling.
- Customer-facing support pages.

The goal is to close the loop after delivery with feedback and issue resolution.

---

**Project:** Food Delivery Platform on ServiceNow  
**Phase:** 5 — Delivery Management  
**Previous Phase:** Phase 4 — Restaurant Order Management  
**Next Phase:** Phase 6 — Reviews, Ratings & Customer Support
