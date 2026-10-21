🍔 Food Delivery Platform on ServiceNow

Phase 2 — Restaurant & Menu Management

«Building a Swiggy-inspired food delivery platform using ServiceNow, one phase at a time.»

Phase: 2 — Restaurant & Menu Management
Previous Phase: Phase 1 — Foundation
Status: Planned / In Progress

---

📌 Overview

Phase 2 focuses on building the restaurant discovery and menu management capabilities of the Food Delivery Platform.

Customers will be able to browse restaurants, search for food, explore menus, filter dishes by category, view prices, and check availability.

Restaurant managers will be able to manage their restaurant profiles, maintain menus, update dish prices, and control item availability.

The application will use ServiceNow tables, server-side business logic, role-based security, and a customer-facing interface.

🎯 Objectives

- [ ] Implement restaurant registration and profile management.
- [ ] Display active restaurants to customers.
- [ ] Create and manage food categories.
- [ ] Implement menu item creation, editing, and deactivation.
- [ ] Add restaurant-based menu filtering.
- [ ] Add food search functionality.
- [ ] Implement vegetarian and category filters.
- [ ] Display menu item prices and availability.
- [ ] Configure restaurant-manager permissions.
- [ ] Create customer-facing restaurant and menu pages.
- [ ] Test security, validation, and application behavior.

🏗️ Technology Stack

Component| Technology
Platform| ServiceNow
Application| Scoped Application
Data Storage| Custom ServiceNow Tables
User Interface| Service Portal or UI Builder
Client-side Logic| JavaScript
Server-side Logic| Script Includes and GlideRecord
Validation| Server-side validation and Business Rules
Security| Roles and ACLs
Workflow| Flow Designer, where required
Version Control| GitHub

🗂️ Data Model

Phase 2 builds on the tables created in Phase 1.

1. Food Restaurant

Purpose: Maintain restaurant profiles and publication status.

Field| Type| Purpose
Restaurant Number| Auto-number / String| Unique identifier
Name| String| Restaurant display name
Description| String| Restaurant description
Address| String| Restaurant location
Contact Number| Phone / String| Contact information
Status| Choice| Pending, Active, Suspended, Closed
Opening Time| Time| Opening time
Closing Time| Time| Closing time
Active| True/False| Record active flag

Business rules:

- Restaurant names must be provided.
- Only authorized users can activate or suspend a restaurant.
- Customers can browse only approved, active restaurants.
- Restaurant status and opening hours must be checked before allowing new orders.
- Existing orders must retain their restaurant reference even if the restaurant is subsequently suspended.

2. Food Category

Create a dedicated table named Food Category.

Field| Type| Purpose
Category Name| String| Category display name
Description| String| Category details
Display Order| Integer| Sort order
Active| True/False| Category availability

Example categories:

- Starters
- Main Course
- Biryani
- Pizza
- Burgers
- Desserts
- Beverages
- Snacks

A dedicated category table is preferable when administrators need to add or reorganize categories without modifying application code.

3. Food Menu Item

Purpose: Manage the dishes offered by each restaurant.

Field| Type| Purpose
Item Number| Auto-number / String| Unique item reference
Restaurant| Reference → Food Restaurant| Owning restaurant
Name| String| Dish name
Description| String| Dish details
Category| Reference → Food Category| Food category
Price| Decimal / Currency| Selling price
Vegetarian| True/False| Vegetarian classification
Available| True/False| Item availability
Active| True/False| Record active flag
Preparation Time| Integer| Estimated preparation time
Image URL| URL / String| Optional dish image reference

Business rules:

- Each menu item must belong to a valid restaurant.
- The price must be greater than or equal to zero; production ordering rules may require a strictly positive price.
- Inactive categories must not be selectable for new menu items.
- Only authorized restaurant managers can modify their restaurant's menu.
- Customers can view only active items belonging to active restaurants.
- Unavailable dishes must not be added to new orders.
- Historical order items must retain the original purchased name and price.

🔗 Relationships

Food Restaurant
       |
       | 1-to-many
       v
Food Menu Item
       |
       | many-to-one
       v
Food Category

A restaurant can have multiple menu items. A category can be associated with many menu items, and each menu item belongs to one restaurant and one category.

The menu item records from this phase will be referenced by the cart and order modules in later phases.

👨‍🍳 Restaurant Management Features

Restaurant Registration

Authorized restaurant managers or administrators can create restaurant profiles.

Required functionality:

- Enter restaurant details.
- Set opening and closing times.
- Save the restaurant as pending.
- Review the profile.
- Activate the restaurant after approval.

Restaurant Profile Management

Restaurant managers should be able to:

- View their restaurant details.
- Update permitted profile fields.
- Manage contact information.
- Review opening hours.
- View restaurant status.

Restaurant ownership must be validated on the server. A manager must not be able to modify a different restaurant by changing a URL, form value, or record identifier.

Restaurant Listing

The customer-facing page should display:

- Restaurant name.
- Restaurant description.
- Address or service area.
- Opening status.
- Menu availability.
- A link to view the menu.

The listing should support pagination or another bounded-result strategy to avoid loading every restaurant at once.

🍕 Menu Management Features

Add Menu Items

Restaurant managers can add a dish by entering:

- Dish name.
- Description.
- Food category.
- Price.
- Vegetarian classification.
- Preparation time.
- Optional image.
- Availability status.

The server must validate the restaurant reference, category, field values, and permissions before saving the record.

Edit Menu Items

Managers can update permitted fields, including descriptions, prices, preparation times, and availability.

Price changes must not rewrite the price of an already placed order.

Enable or Disable Items

Use the availability field to temporarily stop accepting new orders for a dish.

Use the active field to retire a menu item from normal use while preserving its historical references.

Menu Display

Customers should see:

- Dish name and description.
- Price.
- Vegetarian indicator.
- Availability status.
- Preparation-time estimate.
- Optional image.
- Add-to-cart action, implemented in Phase 3.

Unavailable dishes should be clearly marked and must not be orderable.

🔎 Search and Filtering

Implement the following customer-facing filters:

Feature| Expected Behavior
Restaurant Search| Search by restaurant name
Dish Search| Search dish names and descriptions
Restaurant Filter| Display dishes for a selected restaurant
Category Filter| Filter dishes by category
Vegetarian Filter| Show vegetarian dishes
Availability Filter| Show only available dishes
Price Sorting| Sort menu items by price
Name Sorting| Sort alphabetically

Search Implementation

Use server-side Script Includes or appropriately secured Service Portal data access to retrieve matching records.

Search requirements:

- Validate and normalize search input.
- Query only active and permitted records.
- Restrict the result set.
- Use pagination for large datasets.
- Avoid unrestricted queries and unnecessary database calls.
- Enforce ACLs and authorization regardless of client-side filters.

Client-side filtering may improve the user experience, but it must not be the security boundary.

🖥️ Customer-Facing Pages

Page 1: Restaurant Listing

Suggested route:

"/food-restaurants"

Components:

- Search input.
- Restaurant listing.
- Restaurant status.
- Menu navigation.
- Empty-state message.
- Loading and error states.

Page 2: Restaurant Menu

Suggested route:

"/food-menu?id=<restaurant_sys_id>"

Components:

- Restaurant name and details.
- Menu search.
- Category filters.
- Vegetarian filter.
- Dish cards or list rows.
- Price and availability.
- Add-to-cart action placeholder.

The route is illustrative; configure the actual Service Portal page or UI Builder route in your instance. Validate record identifiers and authorization server-side.

Page 3: Restaurant Manager Dashboard

Components:

- Assigned restaurant profile.
- Menu item listing.
- Add menu item form.
- Edit menu item form.
- Availability controls.
- Validation messages.
- Empty-state and error handling.

Use a suitable role-protected page and secure server-side operations.

🔐 Security and Access Control

Roles

Role| Access
"food_admin"| Manage approved restaurant and category configuration
"food_restaurant_manager"| Manage assigned restaurant profiles and menu items
"food_customer"| Browse published restaurants and available menus

Security Requirements

- Configure read, create, update, and delete ACLs as appropriate.
- Validate restaurant ownership before modifying restaurant or menu records.
- Prevent unauthorized price changes.
- Do not rely on hidden form fields or client-side role checks.
- Restrict restaurant approval and suspension to authorized users.
- Sanitize and validate user-supplied values.
- Validate image URLs and avoid rendering untrusted HTML.
- Return only the fields needed by the customer-facing interface.
- Test with separate administrator, manager, and customer accounts.

Where the platform supports it, use record-level security and explicit server-side checks together.

⚙️ Implementation Plan

Step 1: Verify Phase 1

Confirm that the scoped application, Food Restaurant table, Food Menu Item table, roles, and initial ACLs exist.

Step 2: Create Food Category

Create the category table, configure fields, and insert sample categories.

Step 3: Configure Restaurant Management

Create restaurant forms and list views. Configure manager access and administrator approval.

Step 4: Configure Menu Management

Create menu forms and list views. Configure category references, price validation, availability, and restaurant ownership checks.

Step 5: Implement Server-side Operations

Create reusable Script Includes for restaurant listing, menu retrieval, filtering, and authorized menu management.

Ensure that the scripts are appropriate for the chosen interface and scoped application.

Step 6: Build the Customer Pages

Create the restaurant listing and menu pages using Service Portal or UI Builder, depending on the selected interface.

Step 7: Build the Manager Dashboard

Create role-protected menu management pages for adding, editing, and deactivating menu items.

Step 8: Add Search and Filters

Implement search, category filtering, vegetarian filtering, availability filtering, sorting, and pagination.

Step 9: Add Sample Data

Create test restaurants, categories, and menu items using fictitious data.

Step 10: Test and Document

Test each user role, all important validation cases, and the complete restaurant-to-menu browsing flow.

🧪 Testing Checklist

Restaurant Tests

- [ ] Create a restaurant profile.
- [ ] Prevent saving a profile without required fields.
- [ ] Verify restaurant approval and activation.
- [ ] Hide suspended restaurants from customer listings.
- [ ] Prevent a manager from editing another restaurant.
- [ ] Verify opening-hours display and status behavior.

Menu Tests

- [ ] Create a menu item.
- [ ] Reject invalid prices and preparation times.
- [ ] Verify category and restaurant references.
- [ ] Edit permitted menu fields.
- [ ] Disable an item and confirm it cannot be ordered.
- [ ] Verify inactive items are excluded from customer listings.
- [ ] Preserve historical order information when menu details change.

Search Tests

- [ ] Search by restaurant name.
- [ ] Search by dish name.
- [ ] Filter by category.
- [ ] Filter vegetarian dishes.
- [ ] Exclude unavailable dishes.
- [ ] Verify sorting and pagination.
- [ ] Handle empty searches and no matching results.

Security Tests

- [ ] Customer cannot access manager operations.
- [ ] Manager cannot edit unrelated restaurants.
- [ ] Unauthorized users cannot activate restaurants.
- [ ] Unauthorized users cannot change menu prices.
- [ ] Server-side authorization works even when requests are modified.
- [ ] ACLs protect direct record access.

📦 Expected Deliverables

At the end of Phase 2, the project should include:

1. Food Category table.
2. Restaurant profile management.
3. Menu item management.
4. Restaurant and menu relationships.
5. Restaurant listing page.
6. Restaurant menu page.
7. Restaurant manager dashboard.
8. Search, filtering, sorting, and pagination.
9. Appropriate ACLs and server-side validation.
10. Sample data and test results.

🚀 Next Phase

Phase 3 — Cart, Checkout & Order Creation

The next phase will implement:

- Shopping cart management.
- Add, update, and remove cart items.
- Single-restaurant cart validation.
- Server-side price calculation.
- Coupon validation.
- Delivery fee calculation.
- Checkout and address validation.
- Order and order-item creation.
- Payment sandbox integration.

The objective is to let a customer browse restaurants, select dishes, and place a valid order.

---

Project: Food Delivery Platform on ServiceNow
Phase: 2 — Restaurant & Menu Management
Previous Phase: Phase 1 — Foundation
Next Phase: Phase 3 — Cart, Checkout & Order Creation
