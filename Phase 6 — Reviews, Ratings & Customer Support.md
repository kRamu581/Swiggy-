# 🍔 Food Delivery Platform on ServiceNow

## Phase 6 — Reviews, Ratings & Customer Support

> Building a Swiggy-inspired food delivery platform using ServiceNow.

![Phase](https://img.shields.io/badge/Phase-06-orange)
![Platform](https://img.shields.io/badge/Platform-ServiceNow-green)
![Module](https://img.shields.io/badge/Module-Reviews%20%26%20Support-blue)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---

## 📌 Overview

Phase 6 closes the loop after delivery with customer feedback and issue resolution.

Customers can rate and review restaurants for orders they received. Ratings are aggregated and shown on restaurant listings. When something goes wrong, customers can raise a support case against an order, and support agents work the case through a defined lifecycle with SLAs, escalation, and resolution outcomes such as refunds.

This phase builds on:

- **Phase 1:** Application Foundation, Database Design and Security.
- **Phase 2:** Restaurant Management and Menu Management.
- **Phase 3:** Cart, Checkout and Order Creation.
- **Phase 4:** Restaurant Order Management.
- **Phase 5:** Delivery Management.

## 🎯 Objectives

- [ ] Allow customers to review delivered orders.
- [ ] Enforce one review per order.
- [ ] Validate ratings and review content.
- [ ] Aggregate restaurant ratings.
- [ ] Display ratings on restaurant listings and menu pages.
- [ ] Implement review moderation.
- [ ] Allow restaurants to respond to reviews.
- [ ] Allow customers to create support cases for their orders.
- [ ] Implement a support case lifecycle with validated transitions.
- [ ] Implement priority, assignment and SLA tracking.
- [ ] Implement refund and complaint resolution.
- [ ] Build customer, agent and restaurant interfaces.
- [ ] Restrict access to reviews and cases by role and ownership.
- [ ] Test success, abuse and security scenarios.

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
| SLAs | ServiceNow SLA Definitions |
| Security | Roles and ACLs |
| Version Control | GitHub |

## 🗂️ Data Model

Phase 6 uses and extends the following tables:

- Food Review
- Food Support Case
- Food Order
- Food Restaurant
- Food Customer
- Food Payment
- Food Notification

It also introduces tables for review responses and case activity.

### 1. Food Review (Extended)

Extend the Phase 1 **Food Review** table.

| Field | Type | Description |
|---|---|---|
| Order | Reference → Food Order | Completed order |
| Customer | Reference → Food Customer | Reviewer |
| Restaurant | Reference → Food Restaurant | Reviewed restaurant |
| Rating | Integer | Rating from 1 to 5 |
| Comments | Multi-line String | Optional feedback |
| Delivery Rating | Integer | Optional rating from 1 to 5 for the delivery |
| Status | Choice | Published, Pending Moderation, Hidden, Removed |
| Moderation Reason | String | Reason for hiding or removal |
| Moderated By | Reference → User | Moderator |
| Created At | Date/Time | Submission time |
| Edited At | Date/Time | Last edit time, if edited |

**Rules:**

- A review can be created only for an order in **Delivered** status.
- The reviewer must be the customer who placed the order.
- Only one review is permitted per order, enforced by the server and a unique constraint.
- The restaurant is derived from the order and not trusted from the client.
- Rating must be an integer from 1 to 5.
- Reviews can be edited within a configured window, with the edit recorded.
- Hidden or removed reviews are excluded from public display and rating aggregation.

### 2. Food Review Response (New)

**Purpose:** Allow restaurants to respond publicly to reviews.

| Field | Type | Description |
|---|---|---|
| Review | Reference → Food Review | Related review |
| Restaurant | Reference → Food Restaurant | Responding restaurant |
| Responded By | Reference → User | Restaurant manager |
| Response Text | Multi-line String | Public response |
| Status | Choice | Published, Hidden |
| Created At | Date/Time | Response time |

**Rules:**

- One response per review.
- Only a manager of the reviewed restaurant can respond.
- Responses are subject to the same content validation as reviews.

### 3. Food Restaurant (Extended)

Add rating summary fields.

| Field | Type | Description |
|---|---|---|
| Average Rating | Decimal | Average of published review ratings |
| Review Count | Integer | Number of published reviews |

These are derived values maintained by server-side logic, not editable by users.

### 4. Food Support Case (Extended)

Extend the Phase 1 **Food Support Case** table.

| Field | Type | Description |
|---|---|---|
| Case Number | Auto-number / String | Unique case reference |
| Customer | Reference → Food Customer | Requesting customer |
| Order | Reference → Food Order | Related order |
| Category | Choice | Payment, Delivery, Food Quality, Cancellation, Other |
| Priority | Choice | Low, Medium, High, Critical |
| Status | Choice | New, In Progress, Awaiting Customer, Resolved, Closed |
| Description | Multi-line String | Issue details |
| Assigned To | Reference → User | Support agent |
| Resolution Type | Choice | Refund, Partial Refund, Redelivery, Apology, No Action, Other |
| Resolution Notes | Multi-line String | Outcome description |
| Refund Amount | Decimal / Currency | Approved refund amount |
| Resolved At | Date/Time | Resolution time |
| Source | Choice | Customer, Auto-created (failed delivery), Agent |
| Escalated | True/False | Escalation indicator |

**Rules:**

- A customer can create a case only for their own orders.
- The customer is derived from the authenticated session.
- Reasonable limits apply to open cases per order to prevent abuse.
- Resolved cases require a resolution type and notes.
- Refund amounts cannot exceed the amount paid for the order.

### 5. Food Support Case Activity (New)

**Purpose:** Maintain the conversation and audit trail for each case.

| Field | Type | Description |
|---|---|---|
| Case | Reference → Food Support Case | Related case |
| Type | Choice | Customer Comment, Agent Comment, Internal Note, Status Change, Assignment, Refund |
| Author | Reference → User | Author, or empty for system |
| Visibility | Choice | Customer Visible, Internal |
| Message | Multi-line String | Content |
| Created At | Date/Time | Timestamp |

**Rules:**

- Internal notes are never visible to customers.
- Activity records cannot be edited or deleted by regular users.
- Status changes and refund decisions always create an activity record.

## ⭐ Reviews and Ratings

### Submit a Review

1. Verify the authenticated customer owns the order.
2. Verify the order is **Delivered**.
3. Verify that no review exists for the order.
4. Validate the rating and optional delivery rating.
5. Sanitize and length-check the comments.
6. Derive the restaurant from the order.
7. Create the review as **Published**, or **Pending Moderation** if automated checks flag it.
8. Recalculate the restaurant's rating summary.
9. Notify the restaurant.

### Rating Aggregation

- Include only **Published** reviews.
- Recalculate after review creation, edit, hide, removal and restoration.
- Update the summary atomically or by recomputing from source records to avoid drift.
- Round consistently, for example to one decimal place.
- Display a restaurant with no reviews as "New" or "No ratings yet", not zero.

For large volumes, use incremental updates with a periodic reconciliation job.

### Review Moderation

Moderators can hide or remove reviews that violate content policy, including abusive language, personal data, spam, or irrelevant content.

- Moderation requires a recorded reason.
- Moderation actions are logged.
- Moderators can restore incorrectly hidden reviews.
- Customers can report reviews flagged as inappropriate, and flagged reviews enter a moderation queue.
- Ratings are not removed merely because a restaurant disagrees with them. Genuine negative reviews remain published.

### Restaurant Responses

Managers can post one public response per review about their restaurant. Responses should be professional, and moderators can hide responses that violate policy.

### Review Validation Rules

- Comment length limits apply.
- Escape or sanitize all user-supplied text on output and never render untrusted HTML.
- Detect and flag obvious personal data, such as phone numbers or email addresses, for moderation.
- Apply rate limits to prevent spam.

## 🎧 Customer Support Cases

### Create a Case

1. Verify the customer owns the order.
2. Validate category and description.
3. Check open-case limits for the order.
4. Create the case as **New** with a default priority based on category and order state.
5. Create an initial activity record.
6. Start the SLA.
7. Notify the customer and the support queue.

### Auto-created Cases

Cases may be created automatically, for example:

- When a delivery fails (Phase 5).
- When an order is rejected or expires after payment.
- When repeated OTP failures lock a delivery.

Mark these with Source **Auto-created** and link them to the relevant order.

### Case State Model

| Status | Meaning |
|---|---|
| New | Created, not yet picked up |
| In Progress | An agent is working the case |
| Awaiting Customer | Waiting for information from the customer |
| Resolved | Resolution applied, awaiting confirmation |
| Closed | Finalized |

#### Permitted Transitions

```text
New               ──> In Progress
In Progress       ──> Awaiting Customer | Resolved
Awaiting Customer ──> In Progress | Resolved
Resolved          ──> Closed | In Progress (customer reopens within window)
```

| From | To | Allowed Actor |
|---|---|---|
| New | In Progress | Support agent |
| In Progress | Awaiting Customer | Support agent |
| Awaiting Customer | In Progress | Customer reply or agent |
| In Progress / Awaiting Customer | Resolved | Support agent, with resolution details |
| Resolved | Closed | System (after confirmation or timeout) or agent |
| Resolved | In Progress | Customer, within the reopen window |

Any transition not listed must be rejected by the server.

### Priority and SLA

| Priority | Typical Use |
|---|---|
| Critical | Safety concerns, large payment errors |
| High | Missing or failed delivery, payment problems |
| Medium | Food quality, wrong items |
| Low | General queries |

- Configure response and resolution targets for each priority.
- Track SLA breaches and warn agents before breach.
- Escalate breached or critical cases to a supervisor.
- Pause SLA clocks while the case is **Awaiting Customer**, if the policy allows.

Define the actual SLA durations in configuration, not in code.

### Agent Workflow

1. Review the case and linked order, delivery and payment details.
2. Contact the customer or restaurant through case activity if needed.
3. Decide a resolution.
4. Apply the resolution with required notes.
5. Mark the case **Resolved**.

### Refunds and Resolutions

- Refunds are performed only by authorized roles through server-side logic.
- The refund amount cannot exceed the amount paid.
- Large refunds can require supervisor approval, based on a configured threshold.
- A refund updates the payment record consistently (**Refunded**) and creates a case activity record.
- Make refund actions idempotent so retried requests do not refund twice.
- For sandbox payments, simulate the refund and record a sandbox reference.
- For Cash on Delivery, handle refunds according to the defined policy.

### Customer Satisfaction

After a case is resolved, optionally ask the customer to confirm the resolution or rate the support experience. Store this separately from restaurant reviews.

## 🖥️ User Interface Pages

### Page 1: Write a Review

Suggested route:

`/food-review?order=<order_sys_id>`

Components:

- Order summary.
- Star rating input.
- Optional delivery rating.
- Comment box with character limit.
- Submit button and validation messages.
- Message when the order is not eligible or already reviewed.

### Page 2: Restaurant Reviews

Suggested route:

`/food-restaurant-reviews?id=<restaurant_sys_id>`

Components:

- Average rating and review count.
- Rating distribution.
- Review list with pagination.
- Restaurant responses.
- Report review action.

### Page 3: Manager Review Dashboard

Suggested route:

`/food-manager-reviews`

Components:

- Recent reviews for the manager's restaurant.
- Respond action.
- Rating trend summary.

### Page 4: Moderation Queue

Suggested route:

`/food-review-moderation`

Components:

- Flagged and pending reviews.
- Hide, remove and restore actions with required reasons.
- Moderation history.

### Page 5: Customer Support

Suggested route:

`/food-support`

Components:

- Create case form linked to the customer's orders.
- My cases list with status.
- Case detail with customer-visible activity.
- Reply and reopen actions.

### Page 6: Support Agent Workspace

Suggested route:

`/food-support-agent`

Components:

- Case queue filtered by status, priority and SLA.
- Case detail with order, delivery and payment context.
- Internal notes and customer replies.
- Resolution and refund controls.
- SLA indicators and escalation action.

These routes are illustrative. Configure the actual pages and routing in the selected Service Portal or UI Builder experience.

## 🔐 Security Requirements

### Roles

| Role | Permissions |
|---|---|
| `food_customer` | Create and edit own reviews; create and view own cases |
| `food_restaurant_manager` | View reviews and respond for own restaurant |
| `food_support_agent` | Work assigned and queued cases; apply authorized resolutions |
| `food_admin` | Moderate reviews, manage configuration and approve large refunds |

### Mandatory Security Controls

- Derive customer identity from the authenticated session and never trust client-supplied IDs.
- Verify order ownership and Delivered status for reviews on the server.
- Enforce one review per order on the server.
- Enforce the case state transition table on the server.
- Keep internal notes hidden from customers at the ACL and query levels.
- Prevent managers from viewing support cases beyond what their role allows.
- Restrict refund actions to authorized roles and enforce amount limits.
- Make refunds idempotent.
- Sanitize all user-generated text and prevent script injection.
- Rate-limit review and case creation.
- Make activity and moderation records read-only to regular users.
- Protect logs and notifications from personal and payment data.

## ⚙️ Implementation Plan

### Step 1: Verify Phases 1 to 5

Confirm the order, delivery, payment and notification tables, roles and ACLs are in place.

### Step 2: Extend the Review Table

Add the new fields, status choices and a uniqueness rule per order.

### Step 3: Create the Review Response Table

Create Food Review Response and link it to reviews and restaurants.

### Step 4: Add Rating Summary Fields

Add rating fields to Food Restaurant and make them server-maintained.

### Step 5: Implement Review Logic

Create Script Includes for submission, validation, editing, moderation and rating aggregation.

### Step 6: Extend the Support Case Table

Add fields for assignment, resolution, refund and source. Update status choices.

### Step 7: Create the Case Activity Table

Create Food Support Case Activity with visibility controls.

### Step 8: Implement the Case State Engine

Create server-side logic to validate transitions, record activity and notify participants.

### Step 9: Configure Priority and SLAs

Define SLA targets, breach warnings and escalation rules.

### Step 10: Implement Refund Handling

Create idempotent, role-protected refund logic that updates payment and case records consistently.

### Step 11: Implement Auto-created Cases

Hook into failed deliveries, rejected paid orders and OTP lockouts.

### Step 12: Build the Interfaces

Create the review, moderation, support and agent pages.

### Step 13: Configure ACLs and Notifications

Restrict access by role and ownership. Configure customer, restaurant and agent notifications.

### Step 14: Test and Document

Run the testing checklist below, fix issues, and update the documentation.

## 🧪 Testing Checklist

### Review Tests

- [ ] Review is allowed for a delivered order.
- [ ] Review is rejected for an undelivered or cancelled order.
- [ ] A second review for the same order is rejected.
- [ ] Customer cannot review another customer's order.
- [ ] Ratings outside 1 to 5 are rejected.
- [ ] Restaurant is derived from the order, not the client.
- [ ] Comment length limits are enforced.
- [ ] Script and HTML in comments are neutralized.

### Rating Aggregation Tests

- [ ] Average rating and count update after a new review.
- [ ] Hidden and removed reviews are excluded.
- [ ] Restoring a review restores the aggregate.
- [ ] A restaurant with no reviews shows no rating rather than zero.
- [ ] Rounding is consistent.

### Moderation and Response Tests

- [ ] Moderator can hide a review with a reason.
- [ ] Hiding without a reason is refused.
- [ ] Reported reviews enter the moderation queue.
- [ ] Manager can respond once to a review of their restaurant.
- [ ] Manager cannot respond to another restaurant's review.

### Support Case Tests

- [ ] Customer creates a case for their own order.
- [ ] Customer cannot create a case for another customer's order.
- [ ] Open-case limits are enforced.
- [ ] Invalid case transitions are rejected.
- [ ] Resolution without required details is refused.
- [ ] Customer can reopen a resolved case within the window only.
- [ ] Internal notes are never visible to customers.
- [ ] Auto-created cases are generated for failed deliveries.

### SLA Tests

- [ ] SLA starts on case creation.
- [ ] Breach warnings and escalation fire correctly.
- [ ] The SLA clock pauses while awaiting the customer, if configured.

### Refund Tests

- [ ] Refund updates the payment status to Refunded.
- [ ] Refund cannot exceed the amount paid.
- [ ] Repeated refund requests do not refund twice.
- [ ] Large refunds require supervisor approval.
- [ ] Unauthorized roles cannot issue refunds.
- [ ] Case activity records every refund decision.

### Security Tests

- [ ] Customers see only their own reviews, cases and activity.
- [ ] Managers cannot view unrelated cases or internal notes.
- [ ] Modified requests cannot force an invalid status.
- [ ] Direct record access is blocked by ACLs.
- [ ] Rate limits block review and case spam.

## 📦 Expected Deliverables

At the end of Phase 6, the application should contain:

1. An extended Food Review table with moderation support.
2. Food Review Response table.
3. Restaurant rating aggregation.
4. Review submission, editing and moderation workflows.
5. An extended Food Support Case table.
6. Food Support Case Activity table with visibility controls.
7. A case state model with validated transitions.
8. Priority, SLA and escalation configuration.
9. Idempotent, role-protected refund handling.
10. Auto-created cases from delivery and order failures.
11. Customer, manager, moderator and support agent interfaces.
12. ACLs, server-side authorization and updated test results.

## 🚀 Next Phase

### Phase 7 — Coupons, Analytics & Automation

The next phase will implement:

- Coupon management and campaign administration.
- Redemption limits and per-customer controls.
- Operational dashboards and reports.
- Restaurant, delivery and support analytics.
- Scheduled reports.
- Automation with Flow Designer.
- Optional AI-assisted features, such as support case triage and summarization.
- Final hardening, performance review and documentation.

The goal is to complete the platform with promotions, insights and intelligent automation.

---

**Project:** Food Delivery Platform on ServiceNow  
**Phase:** 6 — Reviews, Ratings & Customer Support  
**Previous Phase:** Phase 5 — Delivery Management  
**Next Phase:** Phase 7 — Coupons, Analytics & Automation
