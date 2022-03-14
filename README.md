# Datamodel
 
### Context
Returnly has two main product offerings:
1. Software-as-a-Service: a set of tools and APIs for all things returns: management, automation, analytics, routing, etc. More details about our SaaS offering can be found on our [website](https://returnly.com/automation/).
2. FinTech: we offer our merchants's shoppers an innovative new FinTech product called Instant Refunds™. Instead of waiting weeks for an order refund, eligible customers get returns credited instantly. All the details about our FinTech products can be found on our [website](https://returnly.com/credit/).

One of Returnly's merchants that uses both the SaaS and Instant Refunds™ products is soon due to renew their annual contract. Unfortunately, the merchant is not very data savvy and therefore does not understand the value Returnly adds to its business and the likelihood that the merchant will churn is high. You are part of a task force consisting of members from Sales, Customer Success and the Executive team trying to prevent the merchant from churning.

### Data dictionary and naming conventions

1. All table names are prefixed with either `r_` (Returnly) or `x_` (eXternal) indicating where does the main portion of the data come from / produced by
1. All table names are plural
1. All surrogate keys (typically primary key) are named `sk_id`
1. All foreign keys are named `fk_{parent-table}_id`; if there're multiple child columns, additional qualifiers are added as suffixes
1. All identifiers and other key columns are prefixed with `x_` or `r_` unless absolutely clear without a prefix
1. All monetary amounts are stored in cents, and the column names end with `_cents`, with the word _amount_ typically unnecessary (keep in mind that the min-max transformation of these columns will produce dimensionless fractional cent values between 0 and 1)
1. All timestamp columns end with `_at`
1. All date columns end with `_on`
1. All boolean columns start with `is_`
1. The order of columns is: primary key, foreign keys, any other ids, regular attributes, "metadata" columns (`created|updated_at`)

#### Table: `r_credit_application_decisions`
Credit application decisions

Column | Description
---|---
`fk_r_credit_applications_id` | 
`decision_module` | 

#### Table: `r_credit_applications`
This table stores data related to applications for Instant Refunds™ credit. 
When a shopper initiates a return, the system uses a list of rules as criteria for whether a shopper should be issued credit.
The results, whether approved or denied credit, are captured in this table as a credit application. If a shopper receives Instant Refund™ credit, this means that they have been issued a voucher that can be used towards a repurchase.

Column | Description
---|---
`sk_id` | Primary key
`fk_r_companies_id` | Foreign key
`fk_r_returns_id` | Foreign key
`status` | Credit application status
`repurch_count` | The number of times a voucher was used in different orders (repurchase transactions).
`credit_cents` | Total monetary value of the credit application in US cents. Sanitized!
`total_repurch_cents` | Total repurchase amount in US cents. Sanitized!
`first_repurch_at` | Timestamp in UTC of when the first repurchase associated with that credit application was made.
`last_repurch_at` | Timestamp in UTC of the last known repurchase associated with that credit application was made.
`mlu_model_id` | 
`created_at` | Timestamp in UTC that designates when the credit application was created.
`updated_at` | Timestamp in UTC of last time that the credit application was modified or updated.

#### Table: `r_return_lines`
The records in this table represent order line items that a shopper is returning to a merchant. 
There is a 1:n relationship between `r_returns` and `r_return_lines`.
In other words, a return is composed of one or more associated return line items.

Column | Description
---|---
`sk_id` | Primary key
`fk_r_returns_id` | Foreign key
`fk_x_order_lines_id` | Foreign key
`fk_x_orders_id` | Foreign key
`return_type` | 
`sku_title` | 
`units` | Units
`tax_cents` | Tax amount in US cents. Sanitized!
`discount_cents` | Discount amount in US cents. Sanitized!
`subtotal_cents` | Total cost of all the units in US cents. Sanitized!

#### Table: `r_return_statuses`
This tables stores the dates in UTC of the different transition a return has passed.

Column | Description
---|---
`fk_r_returns_id` | Foreign key
`return_authorized_at` | 
`return_in_transit_at` | 
`return_delivered_at` | 
`return_delivered_late_at` | 
`return_missed_delivery_deadline_at` | 
`return_canceled_at` | 
`return_refunded_at` | 
`days_to_missed_delivery_deadline` | 
`days_to_delivered` | 
`days_to_delivered_late` | 
`days_to_in_transit` | 
`days_to_refunded` | 

#### Table: `r_returns`
Processed returns

Column | Description
---|---
`sk_id` | Primary key
`fk_r_companies_id` | Foreign key
`status` | The status helps us determine where a given return is in the returns flow, and if that return has progressed far enough in the flow to be captured or counted as a completed return.
`created_at` | 
`updated_at` | 
`return_before_at` | 
`shopper_approved_at` | 
`authorized_at` | 
`in_transit_at` | 
`delivered_at` | 
`refunded_at` | 
`originator` | 
`tax_cents` | Tax amount in US cents. Sanitized!
`discount_cents` | Discount amount in US cents. Sanitized!
`restocking_fee_cents` | Sanitized!
`subtotal_cents` | Subtotal product amount in US cents. Sanitized!
`total_cents` | Total amount in US cents. Sanitized!
`is_gift` | 
`is_exempt_from_shipping` | 
`cancel_reason` | 
`credit_app_decision` | 
`is_credit_used` | 
`is_exchange` | 

#### Table: `x_order_lines`
This table stores data as it relates to line items for an external marketplace order, e.g. Shopify or Magento.
A line item represents the items associated with an order and contains information such as the SKU, unit price, and units.
There is a 1:n relationship between `x_orders` and `x_order_lines`.

Column | Description
---|---
`sk_id` | Primary key
`fk_x_orders_id` | Foreign key
`x_product_id` | External product identifier
`x_sku` | External SKU
`units` | 
`unit_price_cents` | Unit price in US cents. Sanitized!
`discount_cents` | Discount in US cents. Sanitized!
`tax_cents` | Tax amount in US cents. Sanitized!

#### Table: `x_orders`
This table stores all external order data from merchants originating from platforms like Shopify or Magento.

Orders are processed at import time to fit Returnly's internal data model. Orders are imported as an upsert to keep internal data in sync with external data each time the external order is updated.

An order must be present in this table to be eligible for return.

Column | Description
---|---
`sk_id` | Primary key
`fk_r_companies_id` | Foreign key
`shipping_fee_cents` | Shipping amount associated with the order in US cents. Sanitized!
`discount_cents` | Order discount amount in US cents. Sanitized!
`subtotal_cents` | The total amount in US cents excluding shipping and taxes but including discounts. Sanitized!
`shipping_tax_cents` | Shipping tax amount for the order in US cents. Sanitized!
`tax_cents` | Tax associated with order in US cents. Sanitized!
`total_cents` | The total amount of order in US cents. Sanitized!
`currency` | 
`status` | Status of the order.
`is_tax_included` | Denotes if tax_amount is included in the order_total.
`is_guest_checkout` | True if the order was placed with guest checkout.
`x_created_at` | Timestamp order was placed in the external platform.
`x_updated_at` | Timestamp order was updated in the external platform.
`x_created_on` | Date order was placed in the external platform.
`created_at` | Internal created at timestamp
`updated_at` | Internal updated at timestamp

#### Table: `x_repurchase_details`
Description

Column | Description
---|---
`fk_x_repurchases_id` | Foreign key
`fk_x_vouchers_id` | Foreign key
`voucher_sequence_no` | Sequence (within parent repurchase)
`credit_used_cents` | Credit amount used for repurchase in US cents. Sanitized!
`processed_at` | The timestamp of the repurchase payment
`created_at` | 

#### Table: `x_repurchases`
Description

Column | Description
---|---
`sk_id` | Primary key
`fk_r_companies_id` | Foreign key
`fk_x_orders_id` | Foreign key
`total_order_cents` | Total order amount in US cents. Sanitized!
`total_credit_used_cents` | Total credit amount used for repurchase in US cents. Sanitized!
`processed_at` | The timestamp of the repurchase payment
`created_at` | 
`updated_at` | 

#### Table: `x_vouchers_hist`
Table containing all transitions of vouchers. A transition happens if a voucher has been used for a repurchase and the balance needs to be decreased by the repurchase amount.

Column | Description
---|---
`fk_x_vouchers_id` | Foreign key
`current_balance_cents` | Current balance amount of the voucher in US cents. Sanitized!
`is_disabled` | True if voucher has been disabled
`created_at` | 

#### Table: `x_vouchers`
Table containing all granted vouchers as part of Instant Refunds™ credit.

Column | Description
---|---
`sk_id` | Primary key
`fk_r_returns_id` | Foreign key
`fk_r_credit_applications_id` | Foreign key
`type_code` | 
`initial_balance_cents` | Initial balance amount of the voucher in US cents. Sanitized!
`current_balance_cents` | Current balance amount of the voucher in US cents. Sanitized!
`is_disabled` | True if voucher has been disabled
`created_at` | 
`updated_at` | 
