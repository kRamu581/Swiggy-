# 🍔 Food Delivery Platform on ServiceNow

## Phase 4 — Restaurant Order Management

> Building a Swiggy-inspired food delivery platform using ServiceNow.

![Phase](https://img.shields.io/badge/Phase-04-orange)
![Platform](https://img.shields.io/badge/Platform-ServiceNow-green)
![Module](https://img.shields.io/badge/Module-Restaurant%20Orders-blue)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---

## 📌 Overview

Phase 4 connects customer checkout to the restaurant's operational workflow.

Once a customer places an order, the restaurant receives a notification, reviews the order, and accepts or rejects it. Accepted orders move through preparation and are marked ready for pickup. Customers are kept informed at every step.

The system validates every status change on the server using a defined order state model, so orders can only move through permitted transitions by authorized users.

This phase builds on:

- **Phase 1:** Application Foundation, Database Design and Security.
- **Phase 2:** Restaurant Management and Menu Management.
- **Phase 3:** Cart, Checkout and Order Creation.

## 🎯 Objectives

- [ ] Notify restaurants of new orders.
- [ ] Build a restaurant order dashboard.
- [ ] Allow restaurants to accept or reject orders.
- [ ] Capture rejection reasons.
- [ ] Capture estimated preparation times.
- [ ] Implement the preparation workflow (Preparing → Ready).
- [ ] Implement an order state model with validated transitions.
- [ ] Record an auditable status history.
- [ ] Handle unavailable items and restaurant rejection.
- [ ] Notify customers of order status changes.
- [ ] Handle restaurant response timeouts.
- [ ] Restrict managers to their own restaurant's orders.
- [ ] Test success, rejection, timeout and security scenarios.

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

Phase 4 extends the following tables from earlier phases:

- Food Order
- Food Order Item
- Food Restaurant
- Food Payment
- Food Customer

It also introduces two new tables for status auditing and notifications.

### 1. Food Order (Extended)

Add the following fields to the existing **Food Order** table.

| Field | Type | Description |
|---|---|---|
| Status | Choice | Expanded state model (see below) |
| Accepted At | Date/Time | When the restaurant accepted the order |
| Estimated Preparation Time | Integer | Estimated minutes, set on acceptance |
| Estimated Ready Time | Date/Time | Calculated ready time |
| Ready At | Date/Time | When the order was marked ready |
| Rejected At | Date/Time | When the restaurant rejected the order |
| Rejection Reason | Choice | Items unavailable, Too busy, Restaurant closing, Other |
| Rejection Notes | String | Optional additional detail |
| Response Deadline | Date/Time | Time by which the restaurant must respond |
| Cancelled By | Reference → User | User who cancelled, if applicable |

### 2. Order State Model

Phase 4 expands the order statuses. Later phases will add delivery states.

| Status | Meaning |
|---|---|
| Pending Payment | Order created, payment not yet resolved |
| Placed | Order placed and awaiting restaurant response |
| Confirmed | Accepted by the restaurant |
| Preparing | Restaurant is preparing the food |
| Ready | Food is ready for pickup |
| Rejected | Declined by the restaurant |
| Expired | Restaurant did not respond before the deadline |
| Cancelled | Cancelled before completion |

#### Permitted Transitions

```text
Pending Payment ──> Placed
Placed          ──> Confirmed | Rejected | Expired | Cancelled
Confirmed       ──> Preparing | Cancelled
Preparing       ──> Ready
Ready           ──> (Phase 5: Picked Up)
```

| From | To | Allowed Actor |
|---|---|---|
| Placed | Confirmed | Restaurant manager (own restaurant) |
| Placed | Rejected | Restaurant manager (own restaurant) |
| Placed | Expired | System (scheduled job) |
| Placed / Confirmed | Cancelled | Customer (within policy), admin |
| Confirmed | Preparing | Restaurant manager (own restaurant) |
| Preparing | Ready | Restaurant manager (own restaurant) |

Any transition not listed must be rejected by the server.

### 3. Food Order Status History (New)

**Purpose:** Maintain an immutable audit trail of order status changes.

| Field | Type | Description |
|---|---|---|
| Order | Reference → Food Order | Related order |
| From Status | String | Previous status |
| To Status | String | New status |
| Changed By | Reference → User | Actor, or empty for system changes |
| Changed At | Date/Time | Timestamp |
| Notes | String | Optional context, such as a rejection reason |

**Rules:**

- Records are created by server-side logic only.
- Records cannot be edited or deleted by regular users.
- Every status change must create a history record.

### 4. Food Notification (New)

**Purpose:** Track notifications sent to restaurants and customers.

| Field | Type | Description |
|---|---|---|
| Recipient | Reference → User | Notification recipient |
| Order | Reference → Food Order | Related order |
| Type | Choice | New Order, Order Confirmed, Order Rejected, Order Preparing, Order Ready, Order Expired |
| Channel | Choice | In-app, Email |
| Message | String | Notification text |
| Read | True/False | Whether the recipient has viewed it |
| Sent At | Date/Time | Delivery timestamp |

Keep messages free of sensitive data such as full addresses or payment details.

### 5. Food Restaurant (Extended)

Add the following fields to support order handling.

| Field | Type | Description |
|---|---|---|
| Manager | Reference → User | Primary manager for the restaurant |
| Default Preparation Time | Integer | Default minutes suggested on acceptance |
| Accepting Orders | True/False | Temporarily pause new orders |

If a restaurant supports several managers, use a mapping table between users and restaurants instead of a single manager reference.

## 🔔 New Order Notifications

When an order reaches **Placed**:

1. Set the response deadline according to the configured policy.
2. Create a **New Order** notification for the restaurant's authorized users.
3. Surface the order at the top of the restaurant dashboard.
4. Optionally send an email notification.

**Requirements:**

- Notify only users associated with the order's restaurant.
- Avoid duplicate notifications for the same order.
- Include the order number, item summary, and response deadline.
- Exclude payment details and unnecessary customer personal data.

## ✅ Accepting and Rejecting Orders

### Accept Order

1. Verify the user is an authorized manager of the order's restaurant.
2. Verify the order is in **Placed** status and has not expired.
3. Accept an estimated preparation time, defaulting to the restaurant's configured value.
4. Validate that the preparation time is a positive integer within configured limits.
5. Set the order to **Confirmed**.
6. Calculate and store the estimated ready time.
7. Create a status history record.
8. Notify the customer.

### Reject Order

1. Verify authorization and ownership.
2. Verify the order is in **Placed** status.
3. Require a rejection reason.
4. Set the order to **Rejected** and record the timestamp.
5. Create a status history record.
6. Trigger the payment handling process (see below).
7. Notify the customer with a clear reason.

### Handling Unavailable Items

If one or more items are unavailable, the restaurant should:

- Reject the order with the reason **Items unavailable**, or
- Contact the customer through a defined process, if partial fulfilment is supported.

Partial order modification is out of scope for this phase. Phase 4 supports full acceptance or full rejection.

### Payment Handling on Rejection or Expiry

- Orders paid through sandbox payment should move the payment to **Refunded**.
- Cash on Delivery orders should be closed with no collection.
- Never leave a rejected order with a payment shown as successful.
- Refund actions must be server-side and logged.

## 👨‍🍳 Preparation Workflow

### Start Preparing

- Allowed from **Confirmed** only.
- Moves the order to **Preparing**.
- Creates a history record and notifies the customer.

### Mark Ready

- Allowed from **Preparing** only.
- Moves the order to **Ready**.
- Records the ready timestamp.
- Creates a history record and notifies the customer.
- In Phase 5, this status triggers delivery partner assignment.

### Preparation Time Handling

- Display the estimated ready time on the dashboard and to the customer.
- Highlight orders approaching or exceeding the estimate.
- Allow a manager to revise the estimate, recording the change in history.

## ⏱️ Response Timeouts

Restaurants must respond within a defined time window.

- Configure a response window, for example a few minutes after placement.
- Run a scheduled job to find **Placed** orders past their response deadline.
- Move expired orders to **Expired** and create a history record.
- Trigger refund handling and notify the customer.
- Make the job idempotent so repeated runs do not duplicate actions.
- Guard against a race where a manager accepts an order at the same moment it expires.

The timeout duration is a configuration value, not a hardcoded assumption.

## 🖥️ User Interface Pages

### Page 1: Restaurant Order Dashboard

Suggested route:

`/food-restaurant-orders`

Components:

- New orders awaiting response, with a countdown to the deadline.
- Active orders grouped by status (Confirmed, Preparing, Ready).
- Order details with items and quantities.
- Accept and Reject actions.
- Start Preparing and Mark Ready actions.
- Pause or resume accepting orders.
- Empty-state, loading and error messages.

### Page 2: Order Details (Restaurant View)

Suggested route:

`/food-restaurant-order-detail?id=<order_sys_id>`

Components:

- Order number and status.
- Item list with notes.
- Estimated preparation time controls.
- Status history timeline.
- Allowed actions for the current status.

### Page 3: Order Tracking (Customer View)

Suggested route:

`/food-order-tracking?id=<order_sys_id>`

Components:

- Current order status.
- Estimated ready time.
- Status timeline.
- Rejection reason, if applicable.
- Cancel option, where policy allows.

### Page 4: Notifications

Suggested route:

`/food-notifications`

Components:

- Notification list.
- Unread indicator.
- Mark-as-read action.
- Link to the related order.

These routes are illustrative. Configure the actual pages and routing in the selected Service Portal or UI Builder experience.

## 🔐 Security Requirements

### Roles

| Role | Permissions |
|---|---|
| `food_restaurant_manager` | View and manage orders for assigned restaurants only |
| `food_customer` | View own orders and notifications; cancel within policy |
| `food_support_agent` | View permitted order details for support |
| `food_admin` | Manage configuration and authorized records |

### Mandatory Security Controls

- Validate restaurant ownership on the server for every order action.
- Enforce the state transition table on the server, not in the UI.
- Never trust client-supplied status values.
- Prevent managers from viewing or changing another restaurant's orders.
- Restrict customers to their own orders and notifications.
- Make status history records read-only to regular users.
- Limit customer data exposed to restaurants to what is needed for fulfilment.
- Protect refund and cancellation operations with explicit authorization checks.
- Use optimistic checks to prevent two actors changing the same order simultaneously.
- Avoid placing personal data in logs or notification text.

## ⚙️ Implementation Plan

### Step 1: Verify Phases 1 to 3

Confirm the order, payment, restaurant, and notification-related tables, roles, and ACLs are in place.

### Step 2: Extend the Data Model

Add the new fields to Food Order and Food Restaurant. Create Food Order Status History and Food Notification.

### Step 3: Define the State Model

Update the Status choices and document the permitted transitions.

### Step 4: Implement the State Transition Engine

Create a Script Include that validates and applies status changes, checking the current state, actor, ownership, and required inputs, and writing the history record.

### Step 5: Implement Order Actions

Create server-side functions for accept, reject, start preparing, mark ready, and revise estimate.

### Step 6: Implement Notifications

Configure events, notifications, and in-app notification records for the restaurant and customer.

### Step 7: Implement Timeout Handling

Create a scheduled job that expires unanswered orders and triggers refund handling.

### Step 8: Implement Payment Handling

Handle refunds or closure for rejected, expired, and cancelled orders.

### Step 9: Build the Restaurant Dashboard

Create the order dashboard and order detail pages with role-protected access.

### Step 10: Build Customer Tracking

Create the order tracking and notification pages.

### Step 11: Configure ACLs

Restrict order access by role and ownership at the table and field levels.

### Step 12: Test and Document

Run the testing checklist below, fix issues, and update the documentation.

## 🧪 Testing Checklist

### Restaurant Order Tests

- [ ] New order appears on the correct restaurant's dashboard.
- [ ] Other restaurants do not see the order.
- [ ] Accept an order with a valid preparation time.
- [ ] Reject an order with a reason.
- [ ] Reject without a reason is refused.
- [ ] Move an order through Preparing and Ready.
- [ ] Estimated ready time is calculated correctly.

### State Transition Tests

- [ ] Invalid transitions are rejected.
- [ ] Skipping states is rejected, for example Placed directly to Ready.
- [ ] Completed or rejected orders cannot be reopened.
- [ ] Status history is created for every change.
- [ ] Status history cannot be edited or deleted by regular users.

### Notification Tests

- [ ] Restaurant is notified of a new order exactly once.
- [ ] Customer is notified on confirmation, rejection, preparing and ready.
- [ ] Notifications contain no payment or unnecessary personal data.
- [ ] Mark-as-read works for the recipient only.

### Timeout Tests

- [ ] Unanswered orders expire after the configured window.
- [ ] Expired orders trigger refund handling and customer notification.
- [ ] Repeated job runs do not duplicate actions.
- [ ] Accepting an order at the moment of expiry is handled consistently.

### Payment Tests

- [ ] Rejected sandbox-paid orders are marked Refunded.
- [ ] Rejected COD orders are closed without collection.
- [ ] No rejected or expired order shows a successful payment.

### Security Tests

- [ ] Manager cannot act on another restaurant's order.
- [ ] Customer cannot call restaurant actions.
- [ ] Modified requests cannot force an invalid status.
- [ ] Customers cannot view other customers' orders or notifications.
- [ ] Direct record access is blocked by ACLs.

## 📦 Expected Deliverables

At the end of Phase 4, the application should contain:

1. An expanded order state model with validated transitions.
2. Food Order Status History table.
3. Food Notification table and notification rules.
4. Restaurant order dashboard.
5. Accept and reject workflows with reasons.
6. Preparation time and ready-time handling.
7. Preparation workflow (Preparing and Ready).
8. Response timeout handling.
9. Refund and closure handling for rejected or expired orders.
10. Customer order tracking and notifications.
11. ACLs and server-side authorization checks.
12. Updated documentation and test results.

## 🚀 Next Phase

### Phase 5 — Delivery Management

The next phase will implement:

- Delivery record creation when an order is Ready.
- Delivery partner assignment.
- Accepting or declining delivery requests.
- Pickup confirmation.
- Delivery status updates.
- Secure delivery verification using a hashed OTP.
- Failed delivery handling.
- Delivery partner dashboard.
- Customer delivery tracking.

The goal is to carry each ready order from the restaurant to the customer.

---

**Project:** Food Delivery Platform on ServiceNow  
**Phase:** 4 — Restaurant Order Management  
**Previous Phase:** Phase 3 — Cart, Checkout & Order Creation  
**Next Phase:** Phase 5 — Delivery Management
