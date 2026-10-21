# 🍔 Food Delivery Platform on ServiceNow

## Phase 3 — Cart, Checkout & Order Creation

> Building a Swiggy-inspired food delivery platform using ServiceNow.

![Phase](https://img.shields.io/badge/Phase-03-orange)
![Platform](https://img.shields.io/badge/Platform-ServiceNow-green)
![Module](https://img.shields.io/badge/Module-Cart%20%26%20Checkout-blue)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---

## 📌 Overview

Phase 3 introduces the complete ordering workflow to the Food Delivery Platform.

Customers will be able to add dishes to their cart, update quantities, remove items, apply coupons, calculate delivery charges, select a delivery address, review the final bill, and place an order.

The system will validate every order on the server before creating order records. It will also support a simulated payment workflow for development and testing.

This phase builds on:

- **Phase 1:** Application Foundation, Database Design and Security.
- **Phase 2:** Restaurant Management and Menu Management.

## 🎯 Objectives

- [ ] Implement cart creation and retrieval.
- [ ] Add menu items to a cart.
- [ ] Update item quantities.
- [ ] Remove items from a cart.
- [ ] Validate restaurant and menu item availability.
- [ ] Prevent items from multiple restaurants being combined in one cart.
- [ ] Calculate the order subtotal on the server.
- [ ] Implement coupon validation.
- [ ] Calculate discounts and delivery charges.
- [ ] Create a checkout page.
- [ ] Validate delivery address and customer details.
- [ ] Create Food Order and Food Order Item records.
- [ ] Implement payment simulation.
- [ ] Prevent duplicate order submissions.
- [ ] Secure all cart, checkout and order operations.
- [ ] Test success, failure and recovery scenarios.

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
| Security | Roles and ACLs |
| Automation | Flow Designer, where appropriate |
| Payment | Simulated payment provider |
| Version Control | GitHub |

## 🗂️ Data Model

Phase 3 uses the following tables from Phase 1:

- Food Customer
- Food Restaurant
- Food Menu Item
- Food Cart
- Food Cart Item
- Food Order
- Food Order Item
- Food Payment
- Food Coupon

Verify that these tables exist before starting implementation.

### 1. Food Cart

**Purpose:** Maintain the customer's active cart.

| Field | Type | Description |
|---|---|---|
| Cart Number | Auto-number / String | Unique cart identifier |
| Customer | Reference → Food Customer | Cart owner |
| Restaurant | Reference → Food Restaurant | Selected restaurant |
| Status | Choice | Active, Converted, Abandoned |

**Rules:**

- Each customer should have at most one active cart per restaurant, unless the application intentionally supports multiple carts.
- A cart belongs to one customer.
- A cart contains items from one restaurant only.
- A converted cart cannot be modified.
- Customers cannot access another customer's cart.

### 2. Food Cart Item

**Purpose:** Store cart line items.

| Field | Type | Description |
|---|---|---|
| Cart | Reference → Food Cart | Parent cart |
| Menu Item | Reference → Food Menu Item | Selected dish |
| Quantity | Integer | Requested quantity |
| Unit Price | Decimal / Currency | Display snapshot, not authoritative |
| Line Total | Decimal / Currency | Display snapshot, not authoritative |

**Rules:**

- Quantity must be a positive integer.
- The menu item must belong to the cart's restaurant.
- Unavailable dishes cannot be added.
- Duplicate cart lines for the same menu item should be merged.
- The server must re-read the current menu price during checkout.

Cart prices are convenience snapshots. They must never be trusted for final billing.

### 3. Food Order

**Purpose:** Store the confirmed order.

| Field | Type | Description |
|---|---|---|
| Order Number | Auto-number / String | Unique order number |
| Customer | Reference → Food Customer | Ordering customer |
| Restaurant | Reference → Food Restaurant | Selected restaurant |
| Status | Choice | Pending Payment, Placed, Confirmed, Cancelled |
| Subtotal | Decimal / Currency | Server-calculated item subtotal |
| Delivery Fee | Decimal / Currency | Server-calculated delivery fee |
| Discount | Decimal / Currency | Validated discount |
| Tax | Decimal / Currency | Applicable tax, if configured |
| Total Amount | Decimal / Currency | Final payable amount |
| Delivery Address | Multi-line String | Address snapshot |
| Coupon | Reference → Food Coupon | Applied coupon, if any |
| Placed At | Date/Time | Order placement time |
| Idempotency Key | String | Unique submission key |

Use a documented order-state model. Restaurant preparation and delivery states will be expanded in later phases.

### 4. Food Order Item

**Purpose:** Preserve the exact items purchased.

| Field | Type | Description |
|---|---|---|
| Order | Reference → Food Order | Parent order |
| Menu Item | Reference → Food Menu Item | Original menu reference |
| Item Name | String | Name snapshot |
| Quantity | Integer | Purchased quantity |
| Unit Price | Decimal / Currency | Price at checkout |
| Line Total | Decimal / Currency | Calculated line amount |

Order items preserve their original names and prices even if the restaurant changes the menu later.

### 5. Food Payment

**Purpose:** Track simulated payment attempts.

| Field | Type | Description |
|---|---|---|
| Payment Reference | Auto-number / String | Unique payment reference |
| Order | Reference → Food Order | Associated order |
| Amount | Decimal / Currency | Amount to pay |
| Method | Choice | Sandbox Payment, Cash on Delivery |
| Status | Choice | Pending, Successful, Failed, Refunded |
| Provider Reference | String | Sandbox transaction reference |
| Attempt Time | Date/Time | Payment attempt timestamp |

Do not store card numbers, CVVs, banking passwords, or real payment credentials.

### 6. Food Coupon

**Purpose:** Apply eligible discounts.

| Field | Type | Description |
|---|---|---|
| Coupon Code | String | Unique coupon code |
| Discount Type | Choice | Fixed, Percentage |
| Discount Value | Decimal | Fixed amount or percentage |
| Minimum Order | Decimal / Currency | Minimum eligible subtotal |
| Maximum Discount | Decimal / Currency | Optional discount ceiling |
| Valid From | Date/Time | Start time |
| Valid Until | Date/Time | Expiration time |
| Active | True/False | Coupon status |

Optional future fields include total redemption limits, per-customer redemption limits, and restaurant-specific restrictions.

## 🛒 Cart Functionality

### Add to Cart

When a customer selects a dish:

1. Validate the authenticated customer.
2. Retrieve the selected menu item.
3. Verify that the restaurant is active.
4. Verify that the dish is active and available.
5. Verify that the requested quantity is valid.
6. Find or create the customer's active cart.
7. Reject or explicitly switch restaurants if the cart belongs to another restaurant.
8. Add the item or increment its existing quantity.
9. Recalculate the displayed cart summary.

All validation must be performed on the server.

### Update Quantity

Customers can increase or decrease a dish quantity.

- Quantity must be a positive integer.
- Quantity must remain within configured limits.
- Revalidate item availability.
- Recalculate the cart subtotal.
- Remove the line if the customer explicitly removes it.

### Remove Item

When an item is removed:

1. Verify cart ownership.
2. Verify that the cart is still active.
3. Delete the selected cart line.
4. Recalculate the subtotal.
5. Handle an empty cart gracefully.

### Clear Cart

Allow customers to remove all items from their active cart.

The cart can remain active and empty or be marked abandoned, depending on the chosen business rules.

### Cart Summary

Display:

- Restaurant name.
- Dish names.
- Quantity controls.
- Unit prices.
- Line totals.
- Item subtotal.
- Proceed to checkout button.

The UI should not calculate authoritative prices. Displayed totals must be refreshed from server-validated data.

## 💰 Checkout and Bill Calculation

### Checkout Fields

The checkout page should include:

- Selected restaurant.
- Ordered dishes and quantities.
- Delivery address.
- Contact details, if required.
- Coupon code.
- Item subtotal.
- Delivery fee.
- Discount.
- Tax, where applicable.
- Final amount.
- Payment method.
- Place Order button.

### Pricing Formula

```text
Item Subtotal = Sum of (Current Unit Price × Quantity)

Eligible Subtotal = Subtotal eligible for discount

Discount = Validated Coupon Discount

Delivery Fee = Configured Delivery Charge

Tax = Applicable Tax Calculated According to Rules

Final Total = Subtotal + Delivery Fee + Tax - Discount
```

The precise tax and fee rules must be configured for the intended deployment. Do not hardcode assumptions about local taxes.

### Price Validation

During checkout, the server must:

- Retrieve current menu prices.
- Revalidate item availability.
- Confirm that every item belongs to the selected restaurant.
- Recalculate line totals.
- Validate the coupon against the current subtotal and validity period.
- Apply maximum-discount and rounding rules.
- Calculate the final amount.
- Reject invalid or manipulated values from the client.

Use a consistent currency and decimal precision. Avoid binary floating-point arithmetic for monetary calculations where it could cause rounding errors; use suitable decimal or integer-minor-unit calculations.

## 🎟️ Coupon Validation

Implement a reusable coupon validation function.

### Validation Rules

1. Coupon code exists.
2. Coupon is active.
3. Current time is within the validity period.
4. Cart subtotal satisfies the minimum order value.
5. Discount configuration is valid.
6. Percentage discounts respect any maximum discount.
7. Fixed discounts do not exceed the eligible subtotal.
8. Coupon usage limits are enforced, if configured.

### Example

Assume the following demonstration values:

| Item | Amount |
|---|---:|
| Item Subtotal | ₹500 |
| Delivery Fee | ₹40 |
| Coupon Discount | ₹50 |
| Tax | ₹0 in this example |
| Final Total | ₹490 |

This example is illustrative, not a production tax or delivery-fee policy.

Coupon redemption must be validated and recorded server-side. If limited-use coupons are supported, prevent concurrent checkouts from exceeding the redemption limit.

## 📦 Order Creation Workflow

```text
Customer
   |
   v
Active Cart
   |
   v
Checkout Request
   |
   v
Validate Customer & Cart Ownership
   |
   v
Validate Restaurant & Menu Availability
   |
   v
Recalculate Prices and Coupon
   |
   v
Calculate Final Total
   |
   v
Create Order and Order Items
   |
   v
Create Payment Attempt
   |
   v
Process Selected Payment Method
   |
   +---- Payment Successful
   |          |
   |          v
   |      Mark Order Placed
   |          |
   |          v
   |      Convert Cart
   |
   +---- Payment Failed
              |
              v
       Record Failure Safely
```

This is a logical workflow. The actual implementation must use transaction-safe state changes and recovery logic.

### Order Creation Requirements

- Generate the order number on the server.
- Use an idempotency key to prevent duplicate orders.
- Revalidate the cart and pricing at submission time.
- Create immutable order-item snapshots.
- Create a payment record.
- Handle partial failures without creating duplicate orders.
- Clear or convert the cart only after the order reaches the appropriate state.
- Log failures without exposing sensitive information.

If an operation spans multiple records and cannot be committed atomically, use explicit states and recovery logic rather than assuming all writes succeed together.

## 💳 Payment Simulation

Phase 3 uses a sandbox payment mechanism instead of a real payment gateway.

### Supported Methods

**Sandbox Payment**

- Generate a test payment reference.
- Simulate success or failure.
- Update the payment record.
- Update the order state consistently.
- Provide a confirmation message.

**Cash on Delivery**

- Record the selected payment method.
- Mark the payment as pending collection.
- Place the order after validating the COD rules.
- Do not mark the payment as successful before collection.

### Payment Failure Handling

- Keep an auditable payment attempt record.
- Do not mark an unpaid order as paid.
- Prevent duplicate charges or duplicate order records.
- Allow a safe retry when permitted.
- Expire abandoned pending orders according to a defined policy.

A production integration will require a payment provider, secure credentials, verified callbacks, and appropriate payment compliance controls.

## 🖥️ User Interface Pages

### Page 1: Shopping Cart

Suggested route:

`/food-cart`

Components:

- Cart item list.
- Quantity controls.
- Remove-item action.
- Item subtotal.
- Empty-cart message.
- Proceed to checkout button.

### Page 2: Checkout

Suggested route:

`/food-checkout`

Components:

- Delivery address form.
- Order summary.
- Coupon input.
- Delivery fee and discount breakdown.
- Final total.
- Payment method selection.
- Place Order button.
- Validation and error messages.

### Page 3: Order Confirmation

Suggested route:

`/food-order-confirmation`

Components:

- Order number.
- Order status.
- Restaurant name.
- Purchased items.
- Final total.
- Payment status.
- Link to order history.

### Page 4: My Orders

Suggested route:

`/food-my-orders`

Components:

- Order history.
- Order number.
- Order date.
- Restaurant.
- Order total.
- Order status.
- View order details.

These routes are illustrative. Configure the actual pages and routing in the selected Service Portal or UI Builder experience.

## 🔐 Security Requirements

### Roles

| Role | Permissions |
|---|---|
| `food_customer` | Manage own cart and place orders for themselves |
| `food_restaurant_manager` | Access permitted restaurant and order information |
| `food_delivery_partner` | Access assigned delivery records |
| `food_admin` | Manage application configuration and authorized records |

### Mandatory Security Controls

- Enforce cart ownership on the server.
- Derive the customer identity from the authenticated session.
- Never trust a client-supplied customer ID.
- Recalculate prices and discounts on the server.
- Restrict direct access to order and payment records.
- Validate every quantity and menu reference.
- Use ACLs and server-side authorization checks.
- Prevent duplicate submissions through idempotency.
- Avoid exposing internal record identifiers or sensitive fields unnecessarily.
- Never store real payment credentials in custom tables.
- Protect logs from personal data and secrets.

## ⚙️ Implementation Plan

### Step 1: Verify Phase 1 and Phase 2

Confirm that the required tables, roles, restaurant records, menu items, and ACLs exist.

### Step 2: Configure Cart Tables

Validate Food Cart and Food Cart Item fields, reference relationships, and ownership rules.

### Step 3: Implement Cart Operations

Create server-side functions for retrieving a cart, adding an item, updating quantity, removing an item, and clearing a cart.

### Step 4: Implement Pricing Logic

Create reusable server-side logic for item subtotals, delivery fees, discounts, tax, and final totals.

### Step 5: Implement Coupon Validation

Validate coupon codes, dates, eligibility, discount rules, and redemption limits.

### Step 6: Build the Cart Page

Display the current cart and allow permitted cart modifications.

### Step 7: Build the Checkout Page

Collect the delivery address, display the server-calculated bill, and allow a payment method to be selected.

### Step 8: Implement Order Creation

Validate the cart and create the Food Order and Food Order Item records.

### Step 9: Implement Payment Simulation

Create payment attempts and test both successful and failed outcomes.

### Step 10: Add Idempotency and Recovery

Prevent repeated submissions from creating duplicate orders. Test recovery from failures during order or payment processing.

### Step 11: Build Order Confirmation and History

Display the final order status and allow customers to view their own orders.

### Step 12: Test and Document

Run the testing checklist below, fix issues, and update the project documentation.

## 🧪 Testing Checklist

### Cart Tests

- [ ] Add an available dish.
- [ ] Increase and decrease quantity.
- [ ] Remove a dish.
- [ ] Handle an empty cart.
- [ ] Reject zero and negative quantities.
- [ ] Reject unavailable dishes.
- [ ] Prevent mixing restaurants in one cart.
- [ ] Prevent unauthorized cart access.

### Checkout Tests

- [ ] Validate delivery address.
- [ ] Recalculate prices on the server.
- [ ] Detect a menu price change before checkout.
- [ ] Reject invalid coupon codes.
- [ ] Reject expired coupons.
- [ ] Apply eligible discounts correctly.
- [ ] Calculate the final total correctly.
- [ ] Prevent client-side price manipulation.

### Order Tests

- [ ] Create an order successfully.
- [ ] Generate a unique order number.
- [ ] Create all order-item snapshots.
- [ ] Preserve the purchased price.
- [ ] Prevent duplicate order submissions.
- [ ] Prevent customers from viewing other customers' orders.
- [ ] Recover safely from partial failures.

### Payment Tests

- [ ] Simulate a successful payment.
- [ ] Simulate a failed payment.
- [ ] Record the correct payment status.
- [ ] Prevent unpaid orders from being marked paid.
- [ ] Retry payment without duplicating the order.
- [ ] Record COD as pending collection.
- [ ] Avoid storing payment credentials.

## 📦 Expected Deliverables

At the end of Phase 3, the application should contain:

1. A functional cart.
2. Add, update, remove, and clear cart operations.
3. A server-side pricing engine.
4. Coupon validation.
5. A checkout interface.
6. Server-validated order creation.
7. Order-item snapshots.
8. A simulated payment workflow.
9. Order confirmation and order history.
10. Idempotency and failure recovery controls.
11. Security and authorization tests.
12. Updated documentation and sample test cases.

## 🚀 Next Phase

### Phase 4 — Restaurant Order Management

The next phase will implement:

- New order notifications for restaurants.
- Accepting or rejecting orders.
- Estimated preparation times.
- Food preparation workflow.
- Ready-for-pickup status.
- Restaurant order dashboard.
- Order state validation.
- Customer notifications.
- Handling unavailable items and restaurant rejection.

The goal is to connect customer checkout to the restaurant's operational workflow.

---

**Project:** Food Delivery Platform on ServiceNow  
**Phase:** 3 — Cart, Checkout & Order Creation  
**Previous Phase:** Phase 2 — Restaurant & Menu Management  
**Next Phase:** Phase 4 — Restaurant Order Management
