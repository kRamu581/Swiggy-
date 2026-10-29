# 🍔 Food Delivery Platform on ServiceNow

## Phase 7 — Coupons, Analytics & Automation

> Building a Swiggy-inspired food delivery platform using ServiceNow.

![Phase](https://img.shields.io/badge/Phase-07-orange)
![Platform](https://img.shields.io/badge/Platform-ServiceNow-green)
![Module](https://img.shields.io/badge/Module-Coupons%20%26%20Analytics-blue)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---

## 📌 Overview

Phase 7 completes the platform with promotions, operational insight and intelligent automation.

Administrators can create and manage coupon campaigns with strict redemption controls. Operational dashboards and reports give restaurants, delivery operations and support teams visibility into performance. Flow Designer automations reduce manual work, and optional AI-assisted features, such as support case triage and summarization, speed up resolution while keeping humans in control. The phase ends with security hardening, performance review and final documentation.

This phase builds on:

- **Phase 1:** Application Foundation, Database Design and Security.
- **Phase 2:** Restaurant Management and Menu Management.
- **Phase 3:** Cart, Checkout and Order Creation.
- **Phase 4:** Restaurant Order Management.
- **Phase 5:** Delivery Management.
- **Phase 6:** Reviews, Ratings and Customer Support.

## 🎯 Objectives

- [ ] Implement coupon campaign management.
- [ ] Enforce total and per-customer redemption limits.
- [ ] Record coupon redemptions reliably.
- [ ] Support restaurant-specific and first-order coupons.
- [ ] Build operational dashboards and reports.
- [ ] Provide restaurant, delivery and support analytics.
- [ ] Schedule recurring reports.
- [ ] Automate routine workflows with Flow Designer.
- [ ] Implement optional AI-assisted support triage and summarization.
- [ ] Protect data privacy in analytics and AI features.
- [ ] Perform security, performance and data-integrity hardening.
- [ ] Complete project documentation.
- [ ] Run end-to-end tests of the full ordering lifecycle.

## 🏗️ Technology Stack

| Component | Technology |
|---|---|
| Platform | ServiceNow |
| Application | Scoped Application |
| Database | Custom ServiceNow Tables |
| UI | Service Portal or UI Builder |
| Client Logic | JavaScript |
| Server Logic | Script Includes and GlideRecord |
| Analytics | Reports, Dashboards, Performance Analytics where licensed |
| Automation | Flow Designer and Scheduled Jobs |
| AI Features | Now Assist or another approved AI capability, where licensed |
| Security | Roles and ACLs |
| Version Control | GitHub |

Analytics and AI features depend on your instance's licensing and enabled plugins. Verify availability before designing around them, and use standard reports and dashboards as the baseline.

## 🗂️ Data Model

Phase 7 extends the following tables:

- Food Coupon
- Food Order
- Food Customer

It also introduces tables for redemptions, campaigns and automation logging.

### 1. Food Coupon (Extended)

Extend the Phase 1 and Phase 3 **Food Coupon** table.

| Field | Type | Description |
|---|---|---|
| Coupon Code | String | Unique coupon code |
| Campaign | Reference → Food Campaign | Parent campaign, if any |
| Discount Type | Choice | Fixed, Percentage |
| Discount Value | Decimal | Fixed amount or percentage |
| Minimum Order | Decimal / Currency | Minimum eligible subtotal |
| Maximum Discount | Decimal / Currency | Optional discount ceiling |
| Valid From | Date/Time | Start time |
| Valid Until | Date/Time | Expiration time |
| Total Redemption Limit | Integer | Maximum total uses, empty for unlimited |
| Per Customer Limit | Integer | Maximum uses per customer |
| Redemption Count | Integer | Server-maintained usage count |
| Restaurant | Reference → Food Restaurant | Restrict to one restaurant, if set |
| First Order Only | True/False | Eligible only for a customer's first order |
| Stackable | True/False | Whether it can combine with other offers |
| Status | Choice | Draft, Active, Paused, Expired, Exhausted |
| Active | True/False | Coupon availability |

**Rules:**

- Coupon codes are unique, case-normalized and validated for allowed characters.
- `Redemption Count` is maintained only by server-side logic.
- A coupon becomes **Exhausted** when its total limit is reached.
- A coupon moves to **Expired** after its validity period.
- Changes to active coupons are recorded for audit.

### 2. Food Campaign (New)

**Purpose:** Group coupons under a promotional campaign.

| Field | Type | Description |
|---|---|---|
| Campaign Number | Auto-number / String | Unique campaign reference |
| Name | String | Campaign name |
| Description | String | Campaign details |
| Start Date | Date/Time | Campaign start |
| End Date | Date/Time | Campaign end |
| Budget | Decimal / Currency | Optional maximum total discount |
| Spent | Decimal / Currency | Server-maintained discount total |
| Status | Choice | Draft, Active, Paused, Completed |

**Rules:**

- A campaign with a budget pauses its coupons when the budget is reached.
- `Spent` is calculated from redemption records, not entered manually.

### 3. Food Coupon Redemption (New)

**Purpose:** Record each coupon use for limits, audit and reporting.

| Field | Type | Description |
|---|---|---|
| Coupon | Reference → Food Coupon | Redeemed coupon |
| Customer | Reference → Food Customer | Customer who used it |
| Order | Reference → Food Order | Related order |
| Discount Amount | Decimal / Currency | Discount actually applied |
| Status | Choice | Reserved, Redeemed, Released |
| Redeemed At | Date/Time | Timestamp |

**Rules:**

- A redemption is **Reserved** when an order is created and **Redeemed** when the order is confirmed or placed according to the defined policy.
- A redemption is **Released** if the order is rejected, expired or cancelled before fulfilment, returning the use to the coupon's limits.
- One redemption record per coupon per order.
- Records are created and updated by server-side logic only.

### 4. Food Automation Log (New)

**Purpose:** Provide an audit trail of automated actions and scheduled jobs.

| Field | Type | Description |
|---|---|---|
| Automation | String | Flow or job name |
| Related Record | Document ID | Record the action affected |
| Action | String | What was performed |
| Result | Choice | Success, Skipped, Failed |
| Details | String | Brief non-sensitive detail |
| Run At | Date/Time | Timestamp |

Log only what is needed to diagnose problems. Never write personal data, payment data or AI prompts containing personal data to this table.

## 🎟️ Coupon Management

### Campaign and Coupon Administration

Administrators can:

- Create, edit, pause and retire campaigns.
- Create coupons individually or in bulk for a campaign.
- Define eligibility rules and limits.
- View redemption counts and spend.
- Pause or end a coupon immediately.

Restrict coupon editing after activation. Changes to discount values or limits on an active coupon should require a defined approval or be blocked.

### Eligibility Rules

Coupon validation, extended from Phase 3, must now check:

1. Coupon exists, is **Active**, and is within its validity period.
2. Campaign is active and within budget.
3. Minimum order value is met.
4. Restaurant restriction matches the cart's restaurant.
5. First-order restriction is satisfied.
6. Total redemption limit has not been reached.
7. Per-customer limit has not been reached.
8. Stacking rules are respected.
9. Discount does not exceed the maximum discount or the eligible subtotal.

### Concurrency and Integrity

- Reserve a redemption atomically when the order is created so concurrent checkouts cannot exceed the limit.
- Re-check the limit at reservation time, not only during cart display.
- Use database-level or locking approaches suitable for the platform to avoid race conditions.
- Release the reservation if the order does not proceed.
- Make reservation and release idempotent.
- Reconcile `Redemption Count` and `Spent` with redemption records periodically.

### Abuse Prevention

- Rate-limit repeated invalid coupon attempts per customer.
- Avoid revealing whether a code exists to repeated failed attempts.
- Flag unusual redemption patterns for review.
- Do not allow coupon values to be supplied or changed by the client.

## 📊 Analytics and Reporting

### Principles

- Aggregate data wherever possible and avoid exposing personal data in reports.
- Limit each dashboard to the roles that need it.
- Restrict restaurant users to their own restaurant's data.
- Define each metric clearly and document its calculation.

### Restaurant Metrics

| Metric | Description |
|---|---|
| Orders received | Count of placed orders |
| Acceptance rate | Confirmed orders divided by placed orders |
| Rejection and expiry rate | Rejected and expired orders divided by placed orders |
| Average preparation time | Average time from confirmation to ready |
| Revenue | Sum of order subtotals for completed orders |
| Average rating | From published reviews |
| Top-selling items | Highest quantity sold |

### Delivery Metrics

| Metric | Description |
|---|---|
| Deliveries completed | Count of delivered orders |
| Average delivery time | Average time from pickup to delivery |
| Assignment time | Average time from Ready to partner acceptance |
| Decline and timeout rate | Declined and timed-out offers divided by offers |
| Failed delivery rate | Failed deliveries divided by attempted deliveries |
| Partner utilization | Active delivery time against available time |

### Support Metrics

| Metric | Description |
|---|---|
| Cases opened and resolved | Volume over time |
| SLA compliance | Cases resolved within target |
| Average resolution time | Time from creation to resolution |
| Cases by category | Distribution across categories |
| Refund totals | Sum of approved refunds |
| Reopen rate | Resolved cases that were reopened |

### Business Metrics

| Metric | Description |
|---|---|
| Gross order value | Sum of order totals |
| Total discounts given | Sum of redeemed discount amounts |
| Coupon redemption rate | Orders using a coupon divided by total orders |
| Campaign performance | Orders, revenue and spend per campaign |
| Repeat customer rate | Customers with more than one order |
| Order funnel | Cart created, checkout started, order placed, delivered |

### Dashboards

| Dashboard | Audience |
|---|---|
| Operations Overview | Administrators |
| Restaurant Performance | Restaurant managers (own restaurant) |
| Delivery Operations | Administrators and dispatchers |
| Support Performance | Support supervisors |
| Campaign Performance | Administrators |

### Scheduled Reports

- Daily operations summary for administrators.
- Weekly restaurant performance summary for managers.
- Weekly support SLA report for supervisors.
- Campaign summary at campaign completion.

Scheduled report recipients must be limited to authorized roles, and report content must respect the same access rules as the dashboards.

### Performance Considerations

- Avoid heavy aggregate queries on large tables during peak hours.
- Use indexed fields for filtering and grouping.
- Consider summary tables updated by scheduled jobs for expensive metrics.
- Limit date ranges and result sizes.

## 🤖 Workflow Automation

Use Flow Designer for routine, well-defined workflows, with server-side Script Includes for complex logic.

### Candidate Automations

| Automation | Trigger | Action |
|---|---|---|
| Expire coupons | Scheduled | Move past-due coupons to Expired |
| Exhaust coupons | Redemption change | Mark coupons Exhausted at the limit |
| Pause campaign at budget | Redemption change | Pause the campaign and its coupons |
| Release stale reservations | Scheduled | Release reservations for abandoned orders |
| Stale order alert | Scheduled | Alert on orders stuck in a state beyond a threshold |
| Delayed delivery alert | Scheduled | Notify operations of delayed deliveries |
| SLA escalation | SLA warning or breach | Escalate and notify supervisors |
| Case auto-close | Scheduled | Close resolved cases after the confirmation window |
| Rating reconciliation | Scheduled | Recompute restaurant rating summaries |
| Redemption reconciliation | Scheduled | Reconcile coupon counts and campaign spend |
| Abandoned cart cleanup | Scheduled | Mark old carts Abandoned |

### Automation Standards

- Make every automation idempotent so repeated runs are safe.
- Write a Food Automation Log entry for each significant action.
- Handle errors explicitly and notify administrators of repeated failures.
- Run automations with the minimum required permissions.
- Avoid infinite trigger loops between flows and business rules.
- Test each automation with realistic data volumes.
- Document triggers, conditions, actions and owners.

## 🧠 Optional AI-Assisted Features

These features are optional and depend on licensing and organizational policy. Treat AI output as a suggestion that a human can review, not as an authoritative decision.

### Support Case Triage

- Suggest a category and priority from the case description.
- Surface similar previous cases and resolutions.
- Agents can accept or override suggestions.

### Case Summarization

- Summarize long case activity for handover or escalation.
- Summaries are labeled as AI-generated.

### Draft Replies

- Suggest a reply for the agent to edit before sending.
- Never send an AI-generated reply to a customer without agent review.

### Review Insights

- Summarize themes across a restaurant's reviews for the manager.
- Flag potentially abusive reviews for moderator attention, without automatically removing them.

### AI Safeguards

- Do not use AI to make final decisions on refunds, bans or moderation.
- Send the minimum data needed and exclude payment details and unnecessary personal data.
- Use only approved, contractually permitted AI services.
- Do not store prompts or outputs containing personal data in logs.
- Record whether a suggestion was accepted, edited or rejected for quality review.
- Provide a configuration switch to disable each AI feature.
- Test for incorrect, biased or unsafe output before enabling.
- Document the AI features, data flows and limitations.

## 🖥️ User Interface Pages

### Page 1: Campaign and Coupon Administration

Suggested route:

`/food-campaign-admin`

Components:

- Campaign list and status.
- Coupon creation and bulk generation.
- Limits, eligibility and validity settings.
- Redemption counts and spend.
- Pause and end actions.

### Page 2: Operations Dashboard

Suggested route:

`/food-ops-dashboard`

Components:

- Live order and delivery status counts.
- Delayed and stuck items.
- Key business metrics.
- Date range filters.

### Page 3: Restaurant Performance Dashboard

Suggested route:

`/food-restaurant-performance`

Components:

- Order volume, acceptance and preparation metrics.
- Revenue and top items.
- Ratings trend.
- Date range filters, limited to the manager's restaurant.

### Page 4: Support Performance Dashboard

Suggested route:

`/food-support-performance`

Components:

- SLA compliance.
- Resolution time and volume trends.
- Category distribution.
- Refund totals.

### Page 5: Customer Coupon Entry

Extends the checkout page from Phase 3.

Components:

- Coupon input with clear, non-revealing error messages.
- Server-validated discount breakdown.
- Eligible coupon suggestions, if implemented.

These routes are illustrative. Configure the actual pages and routing in the selected Service Portal or UI Builder experience.

## 🔐 Security Requirements

### Roles

| Role | Permissions |
|---|---|
| `food_admin` | Manage campaigns, coupons, dashboards, automations and AI configuration |
| `food_restaurant_manager` | View own restaurant analytics |
| `food_support_agent` | Use AI-assisted case features; view support metrics as permitted |
| `food_customer` | Apply eligible coupons at checkout |

Consider adding `food_analyst` or `food_support_supervisor` roles for reporting and escalation if the project requires separation.

### Mandatory Security Controls

- Validate and apply coupons only on the server.
- Maintain redemption counts and campaign spend only through server-side logic.
- Reserve redemptions atomically to prevent over-redemption.
- Rate-limit coupon attempts and avoid revealing coupon existence.
- Restrict analytics by role and restaurant ownership, including report and dashboard definitions.
- Exclude personal data from aggregate reports where not essential.
- Run automations with least privilege.
- Log automation outcomes without sensitive data.
- Restrict and audit changes to active coupons and campaigns.
- Apply data minimization and approved-service controls to AI features.
- Provide switches to disable AI and automation features.

## 🛡️ Final Hardening and Review

Before treating the platform as complete, perform a final review.

### Security Review

- [ ] Review all ACLs across tables, fields and operations.
- [ ] Confirm every client-callable function re-checks authorization on the server.
- [ ] Verify no sensitive data appears in logs, notifications or reports.
- [ ] Confirm no payment credentials or plain OTPs are stored anywhere.
- [ ] Test with separate accounts for every role.
- [ ] Review integrations and scheduled jobs for excess privilege.

### Data Integrity Review

- [ ] Verify required fields, references and choice values on all tables.
- [ ] Confirm unique constraints for orders, deliveries, reviews and coupon codes.
- [ ] Validate that derived values reconcile with source records.
- [ ] Verify audit tables are not editable by regular users.
- [ ] Confirm historical orders retain original prices and names.

### Performance Review

- [ ] Index fields used for filtering, joins and grouping.
- [ ] Review slow queries and large list loads.
- [ ] Ensure pagination on all customer-facing lists.
- [ ] Review scheduled job frequency and run times.
- [ ] Load-test checkout, assignment and dashboard queries.

### Operational Readiness

- [ ] Document configuration values such as fees, timeouts and SLAs.
- [ ] Document runbooks for stuck orders, failed deliveries and refund errors.
- [ ] Define data retention for orders, notifications, logs and AI outputs.
- [ ] Define backup, update-set or source-control promotion practices.
- [ ] Prepare production replacements for sandbox payment behavior.

## ⚙️ Implementation Plan

### Step 1: Verify Phases 1 to 6

Confirm tables, roles, ACLs and end-to-end flows from earlier phases work.

### Step 2: Extend the Coupon Table

Add limits, restrictions and status fields. Create Food Campaign and Food Coupon Redemption.

### Step 3: Implement Coupon Logic

Extend the coupon Script Include with eligibility, reservation, redemption and release.

### Step 4: Integrate with Checkout and Order Lifecycle

Reserve on order creation, redeem on placement, and release on rejection, expiry or cancellation.

### Step 5: Build Coupon Administration

Create campaign and coupon management pages with role protection.

### Step 6: Define Metrics

Document each metric, its source tables and calculation.

### Step 7: Build Reports and Dashboards

Create reports, then dashboards, with role and ownership restrictions.

### Step 8: Configure Scheduled Reports

Set up recurring reports for authorized recipients.

### Step 9: Build Automations

Create Flow Designer flows and scheduled jobs from the candidate list, with logging and idempotency.

### Step 10: Implement Optional AI Features

Enable approved AI features behind configuration switches, with safeguards and human review.

### Step 11: Final Hardening

Complete the security, data integrity, performance and operational reviews.

### Step 12: End-to-End Testing

Run the full lifecycle tests below.

### Step 13: Finalize Documentation

Update all phase READMEs, schema documentation, configuration guides and runbooks.

## 🧪 Testing Checklist

### Coupon Tests

- [ ] Valid coupon applies the correct discount.
- [ ] Expired, paused and inactive coupons are rejected.
- [ ] Minimum order, maximum discount and restaurant restrictions are enforced.
- [ ] First-order coupons work only for first orders.
- [ ] Per-customer limit is enforced.
- [ ] Total limit is enforced under concurrent checkouts.
- [ ] Coupon becomes Exhausted at the limit.
- [ ] Rejected, expired and cancelled orders release the redemption.
- [ ] Campaign pauses coupons when the budget is reached.
- [ ] Client-supplied discount values are ignored.
- [ ] Repeated invalid attempts are rate-limited.

### Analytics Tests

- [ ] Metrics match manually calculated values on sample data.
- [ ] Restaurant managers see only their own restaurant's data.
- [ ] Unauthorized roles cannot open restricted dashboards or reports.
- [ ] Reports exclude unnecessary personal data.
- [ ] Scheduled reports reach only authorized recipients.
- [ ] Dashboards perform acceptably with realistic data volumes.

### Automation Tests

- [ ] Each automation produces the expected result.
- [ ] Repeated runs do not duplicate actions.
- [ ] Failures are logged and reported.
- [ ] Automations do not trigger loops.
- [ ] Reconciliation jobs correct induced drift in counts.
- [ ] Automations run with least privilege.

### AI Feature Tests

- [ ] Suggestions are labeled and can be accepted, edited or rejected.
- [ ] No AI output is sent to a customer without agent review.
- [ ] Personal and payment data are excluded from AI inputs.
- [ ] Each AI feature can be disabled by configuration.
- [ ] Incorrect or low-quality suggestions are handled safely.

### End-to-End Lifecycle Tests

- [ ] Browse restaurants and menus.
- [ ] Add to cart, apply a coupon and check out.
- [ ] Restaurant accepts, prepares and marks ready.
- [ ] Delivery is assigned, picked up and verified with the OTP.
- [ ] Customer reviews the restaurant.
- [ ] Rating summary updates.
- [ ] Customer raises a support case and receives a refund.
- [ ] Dashboards reflect the full lifecycle.
- [ ] Rejection, expiry and failed delivery paths behave correctly.
- [ ] Every role can perform only its permitted actions.

## 📦 Expected Deliverables

At the end of Phase 7, the project should contain:

1. Extended coupon management with limits and restrictions.
2. Food Campaign table and campaign administration.
3. Food Coupon Redemption table with reservation and release logic.
4. Concurrency-safe, server-side coupon validation.
5. Restaurant, delivery, support and business metrics.
6. Role-restricted dashboards and scheduled reports.
7. Flow Designer automations with logging and idempotency.
8. Food Automation Log table.
9. Optional AI-assisted support and review features with safeguards.
10. Completed security, data integrity, performance and operational reviews.
11. End-to-end test results across the full lifecycle.
12. Complete project documentation, including schema, configuration and runbooks.

## 🏁 Project Completion

With Phase 7 complete, the Food Delivery Platform provides the full workflow:

```text
Restaurant Discovery
        |
        v
Menu Browsing and Cart
        |
        v
Checkout, Coupons and Payment
        |
        v
Restaurant Acceptance and Preparation
        |
        v
Delivery Assignment, Pickup and OTP Verification
        |
        v
Reviews, Ratings and Customer Support
        |
        v
Analytics, Automation and Continuous Improvement
```

### Suggested Future Enhancements

- Real payment gateway integration with verified callbacks and compliance controls.
- Live partner location tracking with a clear privacy policy.
- Native mobile experience.
- Scheduled and group orders.
- Loyalty programs and subscriptions.
- Multi-city and multi-language support.
- Advanced recommendations and demand forecasting.

---

**Project:** Food Delivery Platform on ServiceNow  
**Phase:** 7 — Coupons, Analytics & Automation  
**Previous Phase:** Phase 6 — Reviews, Ratings & Customer Support  
**Status:** Final planned phase
