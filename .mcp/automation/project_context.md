# Product Context

<!--
  ⚙️ CONFIGURE: Fill in each section below with your product's details.
  This file is automatically loaded by the agent during test case generation.
  It provides domain knowledge that improves test case naming, preconditions,
  and coverage decisions.

  💡 TIP: The more detail you provide, the better the generated test cases
  will align with your product's terminology and user workflows.
-->

## Product Name

<!-- Replace with your product name -->

Your Product Name

## Description

<!-- One or two sentences describing what your product does and its primary domain -->

Describe your product's domain and primary function here.

---

## Key Features

<!-- List the main features of your product. Each feature should have a brief description.
     The agent uses this to understand scope and generate relevant test cases. -->

### Feature 1: [Feature Name]

Brief description of what this feature does and its key capabilities.

### Feature 2: [Feature Name]

Brief description of what this feature does and its key capabilities.

### Feature 3: [Feature Name]

Brief description of what this feature does and its key capabilities.

---

## User Roles

<!-- Define the user roles in your system. The agent uses these for preconditions
     and role-specific test scenarios. -->

| Role          | Description        | Key Permissions                  |
| ------------- | ------------------ | -------------------------------- |
| Admin         | Full system access | All operations                   |
| Standard User | Regular access     | View, create, edit own resources |
| Guest/Viewer  | Read-only access   | View only                        |

---

## Terminology

<!-- Define domain-specific terms used in your product. This ensures the agent
     uses correct terminology in test case titles and descriptions. -->

| Term     | Definition                            |
| -------- | ------------------------------------- |
| [Term 1] | What it means in your product context |
| [Term 2] | What it means in your product context |

---

<!--
  ============================================================================
  EXAMPLE: Below is a commented-out example showing what a filled-in version
  looks like for a fictional e-commerce platform. Delete this block once you've
  filled in your own details above.
  ============================================================================

  ## Product Name
  ShopFlow — E-Commerce Platform

  ## Description
  ShopFlow is a B2C e-commerce platform that enables merchants to manage product
  catalogs, process orders, and handle customer relationships.

  ## Knowledge Sources (RAG)
  - **Merchant Help Center:** 2,000+ articles covering store setup, payments, shipping
  - **API Documentation:** REST API for products, orders, customers, inventory
  - **Integration Guides:** Payment gateways (Stripe, PayPal), shipping providers

  ## Key Features

  ### Feature 1: Product Catalog
  Manage products with variants (size, color), pricing, inventory tracking,
  and bulk import/export via CSV.

  ### Feature 2: Order Management
  Process orders through fulfillment pipeline: pending → confirmed → shipped → delivered.
  Supports partial fulfillment, returns, and refunds.

  ### Feature 3: Customer Dashboard
  Customer-facing portal for order history, saved addresses, wishlists,
  and account management.

  ## User Roles
  | Role | Description | Key Permissions |
  |------|-------------|-----------------|
  | Merchant Admin | Store owner | Full store management |
  | Store Staff | Employee access | Orders, inventory |
  | Customer | Shopper | Browse, purchase, manage account |
  | Guest | Unauthenticated | Browse, add to cart |

  ## Terminology
  | Term | Definition |
  |------|-----------|
  | SKU | Stock Keeping Unit — unique identifier for each product variant |
  | Fulfillment | The process of picking, packing, and shipping an order |
  | Cart Abandonment | When a customer adds items to cart but doesn't complete checkout |
-->
