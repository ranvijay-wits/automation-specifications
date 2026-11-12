# Food & Beverage (ONDC:RET11 1.2.5)

## Overview

Food & Beverage (`ONDC:RET11`) covers ordering prepared food and drinks from restaurants, cloud kitchens, cafés, bakeries and sweet shops.

Food orders are prepared after the order is placed and must be delivered quickly while fresh. Menus change through the day, items often come with customisations (size, add-ons, toppings), and outlets have fixed operating hours and short delivery radii. Food sellers must also show FSSAI and food-safety details.

ONDC lets buyer apps and seller apps connect through a common, open protocol instead of building separate integrations with each other. A buyer app integrates once with the network and can show catalogs from many sellers. A seller app integrates once and can reach buyers on every participating buyer app.

The seller remains responsible for its catalog, prices, stock, packing, delivery and after-sales service. ONDC standardizes the network messages used to discover products, place orders, track delivery, and handle cancellations, returns and grievances.

This guide explains the food journey from a business point of view first, and then covers the API flows defined in this specification (version 1.2.5).

---

## What's New in 1.2.5

Compared with version 1.2.0, the Food & Beverage 1.2.5 specification adds:

| Area | What it adds |
|---|---|
| **Fulfillment choices** | Slotted delivery (buyer picks a date and time window), multiple delivery options in one quote, self-pickup from the store. |
| **Changes after the order** | Buyer instructions and delivery address change after confirmation, and a delivery OTP sent by the seller for handover. |
| **Payments and commercials** | A standard commercial model for BNP/SNP fees, seller offers applied to the quote. |
| **Reverse flows** | Forced cancellation. |

---

## Participants

| Participant | What This Means |
|---|---|
| **Buyer App (BAP)** | The consumer-facing application where the buyer searches for food, builds a cart, pays, tracks the order and raises complaints. It can be a shopping app, payments app, or any app that offers food to its users. |
| **Seller App (BPP)** | The application representing sellers such as restaurants, cloud kitchens, cafés and food brands. It publishes the catalog, checks stock and serviceability, prices the order, confirms it, manages packing and delivery, and handles cancellations, returns and replacements. |
| **Logistics Provider** | The delivery partner that picks up and delivers the order. Usually the seller arranges delivery. In the buyer-managed delivery flow, the buyer app arranges the courier and reports the delivery progress back to the seller. |
| **Gateway** | Network infrastructure that forwards a buyer app's `/search` to all relevant seller apps based on domain and city. |

---

## The User Journey

A buyer discovers sellers, builds a cart, pays, and receives the order with status updates along the way. The journey has the following stages.

### 1. Discovery and Catalog

The buyer searches for an item, a category (for example "Biryani" or "Pizza") or a seller. The buyer app sends the search with the buyer's location.

Seller apps that serve that location reply with their catalog, which can include:

- Outlet name, location, operating hours and delivery area
- Menu categories and items, with images and descriptions
- Price, and whether an item is veg or non-veg
- Customisation options such as size, add-ons and toppings, with their prices
- Item availability and preparation time

**Network interaction:**

- **`/search`**: The buyer app sends the buyer's search intent and location to the network.
- **`/on_search`**: Each seller app returns its catalog for the buyer's location.

### 2. Item Selection and Quote

The buyer adds items from one seller to the cart. The seller checks stock and whether it can deliver to the buyer's location, and returns a price quote.

**Network interaction:**

- **`/select`**: The buyer app sends the selected items, quantities, seller location and delivery location.
- **`/on_select`**: The seller returns the quote, broken down into item prices, delivery charges, packing charges and taxes, along with the expected delivery time.

### 3. Order Initialization

The buyer enters the delivery address, contact details and billing details.

**Network interaction:**

- **`/init`**: The buyer app sends the full delivery address, contact details and billing details.
- **`/on_init`**: The seller fixes the final quote and shares the order terms, such as cancellation and return terms, payment collection details and settlement details.

### 4. Order Confirmation and Payment

The buyer pays online.

**Network interaction:**

- **`/confirm`**: The buyer app places the order with the payment status (`PAID` for prepaid).
- **`/on_confirm`**: The seller confirms the order. The order is `Created` and its fulfillment is `Pending`.

### 5. Order Processing and Packing

The seller processes the order and packs it. The seller keeps the buyer app updated through unsolicited `/on_status` messages.

| Status update | What it means |
|---|---|
| **Pending** | The seller has received the order and it is waiting to be processed. |
| **Packed** | Items are packed and ready for pickup. |

### 6. Delivery Assignment and Tracking

A delivery agent is assigned and collects the order.

| Status update | What it means |
|---|---|
| **Agent-assigned** | A delivery agent is assigned. The agent's name and phone number are shared. |
| **At-pickup** | The agent has reached the seller's location. |
| **Order-picked-up** | The agent has collected the order from the seller. |
| **At-delivery** | The agent has reached the buyer's location. |

The buyer app can request live tracking at any point.

**Network interaction:**

- **`/track`**: The buyer app asks for tracking details.
- **`/on_track`**: The seller returns the tracking link or the agent's current location.

### 7. Delivery and Order Completion

The agent hands over the order to the buyer. The seller sends **`Order-delivered`** through `/on_status`, with the delivery time. The order is now `Completed`.

### 8. Settlement

After delivery, the buyer app settles the order amount with the seller as per the settlement terms agreed in `/on_init`, after deducting the agreed buyer app fee and any applicable taxes such as TCS or TDS.

---

## Additional Scenarios

Beyond the standard delivery journey, this specification supports the following scenarios.

### 1. Slotted Delivery

The seller offers delivery slots (a date and time window) in `/on_select`. The buyer picks a slot, which is carried through `/init` and `/confirm`, and the seller delivers within that slot.

### 2. Multi-Option Fulfillment

The seller offers more than one delivery option in the same quote, for example express or standard delivery, each with its own charge and delivery time. The buyer chooses one before placing the order.

### 3. Self-Pickup

The buyer collects the order from the store, so no delivery charge applies. The seller tells the buyer app when the order is ready for pickup, and again when the buyer has collected it.

### 4. Buyer Delivery

The buyer app arranges delivery instead of the seller. The seller packs the order and hands it over, and the buyer app keeps the seller informed of the delivery progress through `/update`.

### 5. Buyer Instructions and Delivery Update

After the order is confirmed and before dispatch, the buyer can send delivery instructions (for example "call on arrival") and correct the delivery address. The seller acknowledges each change through `/on_update`, and sends a delivery OTP through `/on_update` before handover.

### 6. Commercial Model for BNP/SNP

The buyer network participant (BNP) and seller network participant (SNP) agree on fees, commissions, tax deductions and settlement terms during `/on_select` and `/on_init`, and these are fixed when the order is confirmed.

### 7. Offers

Sellers publish offers in their catalog. The buyer applies an offer during `/select`, and the seller reflects the discount in the quote through to order confirmation.

### 8. Out of Stock

If an item is not available when the buyer selects it, the seller can handle it in two ways:

- **Cart correction**: The seller marks the unavailable item in `/on_select`. The buyer changes the cart, selects again, and continues with the order.
- **Error response**: The seller rejects the selection in `/on_select` with an out-of-stock error code (for example `40002`).

### 9. Buyer Cancellation

The buyer cancels the order before it is shipped, with a cancellation reason. The seller checks its cancellation terms, cancels the order, and shares any cancellation charges and the refund amount.

### 10. Forced Cancellation

Once an order is out for delivery, a normal cancellation request can be refused. The buyer app can then raise a forced cancellation, the seller cancels the order, and the buyer app updates the settlement for the refund.

### 11. RTO and Part-Cancellation

If some items are not available after confirmation, the seller cancels only those items through an unsolicited `/on_update`, and the buyer app updates the settlement for the partial refund. If delivery then fails (for example the buyer cannot be reached), the seller cancels the order as Return to Origin (RTO) through an unsolicited `/on_cancel`, the buyer app updates the settlement, and the seller confirms when the order is back.

### 12. Return

After delivery, the buyer can return eligible items (the full order or some items) within the return window. The seller approves the return, the items are picked up, and the refund is settled once the seller receives them.

### 13. Incremental Catalog Updates

Sellers do not need to resend the full catalog for every change:

- **Push**: The seller sends unsolicited `/on_search` updates for changes such as price or stock, and when a store closes or reopens.
- **Pull**: The buyer app asks for only the items changed after a given time, and the seller returns just those items.

---

## Issue and Grievance Management

A buyer can raise a complaint through the buyer app about an order, an item or the delivery. The buyer app sends it to the seller app, which investigates and resolves it.

The grievance process includes:

- **Registration**: The buyer raises the issue, with a description and photos where relevant.
- **Acknowledgement**: The seller confirms receipt and starts processing the issue.
- **Information exchange** (IGM 2.0.0): The seller can ask for more information, and the buyer can provide it.
- **Resolution**: The seller proposes a resolution, such as a refund, a replacement, or an explanation with no further action.
- **Acceptance, rejection or escalation** (IGM 2.0.0): The buyer can accept or reject the resolution, and can escalate an unresolved issue to the next level.
- **Closure**: The buyer app closes the issue once it is resolved.

This specification supports both IGM versions:

| Version | How it works |
|---|---|
| **IGM 1.0.0** | A simple sequence: the buyer opens an issue, the seller acknowledges it, sends the resolution through `/on_issue_status`, and the buyer closes it. |
| **IGM 2.0.0** | Adds information requests, resolution acceptance or rejection, and escalation. An accepted resolution can trigger a follow-up action on the order, such as a replacement or a return, through `/on_update`. |

---

## Flow Summary

| # | Flow | Requirement | What it covers |
|---|---|---|---|
| 1 | Delivery Flow | Mandatory | The standard order: the buyer selects items, gets a quote, pays online, and receives the order through seller-managed delivery |
| 2 | Slotted Delivery Flow | Optional | The buyer books a delivery date and time slot, and the seller delivers within it |
| 3 | Multi-Option Fulfillment Flow | Optional | The seller offers more than one delivery option (for example express or standard), each with its own charge and delivery time, and the buyer picks one |
| 4 | Self-Pickup Flow | Optional | The buyer orders online and collects the order from the store |
| 5 | Buyer Delivery Flow | Mandatory | The buyer app arranges delivery |
| 6 | Buyer Instructions and Delivery Update Flow | Optional | After confirming the order, the buyer adds delivery instructions and changes the delivery address before dispatch |
| 7 | Commercial Model for BNP/SNP Flow | Mandatory | Shows how the buyer network participant (BNP) and seller network participant (SNP) agree on fees, commissions, tax deductions and settlement terms as part of placing an order |
| 8 | Offers Flow | Mandatory | The buyer applies a seller offer (such as a discount) while selecting items, and the quote reflects the offer |
| 9 | Out of Stock Flow | Mandatory | An item turns out to be out of stock when the buyer selects it |
| 10 | Out of Stock Flow (Error Code) | Mandatory | The seller rejects the selection straight away because an item is out of stock |
| 11 | Buyer Cancellation Flow | Mandatory | The buyer cancels a confirmed order before it is shipped |
| 12 | Forced Cancellation Flow | Mandatory | The order is already out for delivery when the buyer wants to cancel |
| 13 | RTO and Part-Cancellation Flow | Mandatory | The seller cancels some items it cannot supply, then delivery of the rest fails and the order returns to the seller (Return to Origin) |
| 14 | Return Flow | Mandatory | After delivery, the buyer returns an item |
| 15 | Discovery Flow – Full Catalog | Mandatory | The buyer app searches a city and the seller returns its full catalog |
| 16 | Discovery Flow – Incremental Catalog (Push) | Mandatory | After the first search, the seller pushes catalog changes as they happen, without resending the whole catalog |
| 17 | Discovery Flow – Incremental Catalog Refresh (Pull) | Optional | The buyer app asks the seller for only the items that changed after a given time |
| 18 | Delivery Flow with IGM 1.0.0 | Mandatory | A delivered order followed by a complaint handled under IGM 1.0.0 |
| 19 | Delivery Flow with IGM 2.0.0 – No Action | Mandatory | A complaint under IGM 2.0.0 that the seller resolves with an explanation and no further action, which the buyer accepts |
| 20 | Delivery Flow with IGM 2.0.0 – Rejection and Escalation | Mandatory | The buyer rejects the seller's resolution and escalates the complaint |
| 21 | Delivery Flow with IGM 2.0.0 | Mandatory | A complaint under IGM 2.0.0 where the seller asks for more information, proposes a replacement, and the buyer accepts |

In the flows below, **[unsolicited]** marks a message the sender pushes on its own, not as a reply to a request.

---

## Detailed Protocol Flows

### Flow 1: Delivery Flow

#### 1. Short Description
The standard order: the buyer selects items, gets a quote, pays online, and receives the order through seller-managed delivery. Most other order flows build on these steps.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote (`quote.breakup`) with item prices, delivery and packing charges, taxes, and the expected delivery time (`@ondc/org/TAT`).
- **`/init` (BAP -> BPP)**: Sends the delivery address, contact details and billing details.
- **`/on_init` (BPP -> BAP)**: Returns the final quote, order terms, and payment and settlement details.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `At-pickup` -> `Order-picked-up`.
- **`/track` & `/on_track` (BAP <-> BPP)**: The buyer app requests and receives live tracking details.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `At-delivery` -> `Order-delivered`.

---

### Flow 2: Slotted Delivery Flow

#### 1. Short Description
The buyer books a delivery date and time slot, and the seller delivers within it.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote with the available delivery slots and their charges.
- **`/init` (BAP -> BPP)**: Sends the fulfillment option chosen by the buyer.
- **`/on_init` (BPP -> BAP)**: Returns the final quote, order terms, and payment and settlement details.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `At-pickup` -> `Order-picked-up` -> `At-delivery` -> `Order-delivered`.

---

### Flow 3: Multi-Option Fulfillment Flow

#### 1. Short Description
The seller offers more than one delivery option (for example express or standard), each with its own charge and delivery time, and the buyer picks one.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns several fulfillment options, each with its own ID, charge and delivery time.
- **`/init` (BAP -> BPP)**: Sends the fulfillment option chosen by the buyer.
- **`/on_init` (BPP -> BAP)**: Fixes the quote for the chosen option.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `At-pickup` -> `Order-picked-up` -> `At-delivery` -> `Order-delivered`.

---

### Flow 4: Self-Pickup Flow

#### 1. Short Description
The buyer orders online and collects the order from the store. There is no delivery charge and no delivery agent.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote with the available fulfillment options, each with its own ID, type and charge.
- **`/init` (BAP -> BPP)**: Sends the delivery address, contact details and billing details.
- **`/on_init` (BPP -> BAP)**: Returns the final quote, order terms, and payment and settlement details.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Order-picked-up` (by buyer's courier).

---

### Flow 5: Buyer Delivery Flow

#### 1. Short Description
The buyer app arranges delivery. The seller packs the order and hands it over, and the buyer app reports the delivery progress back to the seller.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote with the available fulfillment options, each with its own ID, type and charge.
- **`/init` (BAP -> BPP)**: Sends the delivery address, contact details and billing details.
- **`/on_init` (BPP -> BAP)**: Returns the final quote, order terms, and payment and settlement details.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed`.
- **`/update` (BAP -> BPP)**: Tells the seller that the buyer app's delivery agent has been assigned.
- **`/on_update` (BPP -> BAP)**: Acknowledges the agent assignment.
- **`/update` (BAP -> BPP)**: Shares an interim update on the buyer app's own delivery.
- **`/on_update` (BPP -> BAP)**: Acknowledges the interim delivery update.
- **`/update` (BAP -> BPP)**: Tells the seller that the order is out for delivery with the buyer app's courier.
- **`/on_update` (BPP -> BAP)**: Acknowledges the update.
- **`/update` (BAP -> BPP)**: Tells the seller that the buyer app's courier has delivered the order.
- **`/on_update` (BPP -> BAP)**: Acknowledges the delivery and completes the order.

---

### Flow 6: Buyer Instructions and Delivery Update Flow

#### 1. Short Description
After confirming the order, the buyer adds delivery instructions and changes the delivery address before dispatch. The seller sends a delivery OTP before handover.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Pending`**: As in Flow 1.
- **`/update` (BAP -> BPP)**: Sends the new delivery address.
- **`/on_update` (BPP -> BAP)**: Confirms the new address.
- **`/update` (BAP -> BPP)**: Sends the buyer's delivery instructions.
- **`/on_update` (BPP -> BAP)**: Acknowledges the instructions.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Packed` -> `Agent-assigned` -> `At-pickup` -> `Order-picked-up`.
- **`/on_update` [unsolicited] (BPP -> BAP)**: Sends the delivery OTP used to verify the handover.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `At-delivery` -> `Order-delivered`.

---

### Flow 7: Commercial Model for BNP/SNP Flow

#### 1. Short Description
Shows how the buyer network participant (BNP) and seller network participant (SNP) agree on fees, commissions, tax deductions and settlement terms as part of placing an order.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote with the commercial terms, such as the buyer app's fee.
- **`/init` (BAP -> BPP)**: Sends the settlement details for the commercial model.
- **`/on_init` (BPP -> BAP)**: Confirms the settlement window, tax deductions and bank details.
- **`/confirm` (BAP -> BPP)**: Places the order with the agreed commercial terms.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order and fixes the commercial terms.

---

### Flow 8: Offers Flow

#### 1. Short Description
The buyer applies a seller offer (such as a discount) while selecting items, and the quote reflects the offer.

#### 2. API Breakdown & Technical Actions
- **`/search` (BAP -> Gateway)**: Sends the search for the buyer's city and the domain.
- **`/on_search` (BPP -> BAP)**: Returns the catalog: store details and serviceability, categories, items with prices and statutory details, and any offers.
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote (`quote.breakup`) with item prices, delivery and packing charges, taxes, and the expected delivery time (`@ondc/org/TAT`).
- **`/init` (BAP -> BPP)**: Sends the delivery address, contact details and billing details.
- **`/on_init` (BPP -> BAP)**: Returns the final quote, order terms, and payment and settlement details.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).

---

### Flow 9: Out of Stock Flow

#### 1. Short Description
An item turns out to be out of stock when the buyer selects it. The buyer corrects the cart and completes the order.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the original cart.
- **`/on_select` (BPP -> BAP)**: Reports the unavailable item with an out-of-stock error (for example `40002`).
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote (`quote.breakup`) with item prices, delivery and packing charges, taxes, and the expected delivery time (`@ondc/org/TAT`).
- **`/init` (BAP -> BPP)**: Sends the delivery address, contact details and billing details.
- **`/on_init` (BPP -> BAP)**: Returns the final quote, order terms, and payment and settlement details.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `At-pickup` -> `Order-picked-up`.
- **`/track` & `/on_track` (BAP <-> BPP)**: The buyer app requests and receives live tracking details.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `At-delivery` -> `Order-delivered`.

---

### Flow 10: Out of Stock Flow (Error Code)

#### 1. Short Description
The seller rejects the selection straight away because an item is out of stock.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends a cart that includes an out-of-stock item.
- **`/on_select` (BPP -> BAP)**: Reports the unavailable item with an out-of-stock error (for example `40002`).

---

### Flow 11: Buyer Cancellation Flow

#### 1. Short Description
The buyer cancels a confirmed order before it is shipped.

#### 2. API Breakdown & Technical Actions
- **`/select` to `/on_confirm`**: As in Flow 1.
- **`/cancel` (BAP -> BPP)**: Sends the order ID and the cancellation reason code.
- **`/on_cancel` (BPP -> BAP)**: Cancels the order (`Cancelled`) and returns any cancellation charges and the refund amount.

---

### Flow 12: Forced Cancellation Flow

#### 1. Short Description
The order is already out for delivery when the buyer wants to cancel. The buyer app forces the cancellation and the refund is settled.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/cancel` (BAP -> BPP)**: A normal cancellation request, which is not accepted at this stage.
- **`/cancel` (BAP -> BPP)**: A forced cancellation request.
- **`/on_cancel` (BPP -> BAP)**: Cancels the order (`Cancelled`) and returns any cancellation charges and the refund amount.
- **`/update` (BAP -> BPP)**: The buyer app updates the settlement for the refund.

---

### Flow 13: RTO and Part-Cancellation Flow

#### 1. Short Description
The seller cancels some items it cannot supply, then delivery of the rest fails and the order returns to the seller (Return to Origin).

#### 2. API Breakdown & Technical Actions
- **`/select` to `/on_confirm`**: As in Flow 1.
- **`/on_update` (BPP -> BAP)**: The seller cancels the unavailable items.
- **`/update` (BAP -> BPP)**: The buyer app updates the settlement for the partial refund.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `At-pickup` -> `Order-picked-up` -> `At-delivery` -> `Order-delivered`.
- **`/on_cancel` [unsolicited] (BPP -> BAP)**: Delivery fails and the seller cancels the order as RTO (reason code `013`).
- **`/update` (BAP -> BPP)**: The buyer app updates the settlement after the RTO.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `RTO-Disposed`.

---

### Flow 14: Return Flow

#### 1. Short Description
After delivery, the buyer returns an item. The seller approves the return, the item is picked up, and the refund is settled.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `At-delivery` -> `Order-delivered`.
- **`/update` (BAP -> BPP)**: Sends the return request with the item, quantity and return reason.
- **`/on_update` (BPP -> BAP)**: The seller records the return request.
- **`/on_update` [unsolicited] (BPP -> BAP)**: Return received by the seller, which completes the return.
- **`/update` (BAP -> BPP)**: The buyer app updates the settlement for the return refund.

---

### Flow 15: Discovery Flow – Full Catalog

#### 1. Short Description
The buyer app searches a city and the seller returns its full catalog.

#### 2. API Breakdown & Technical Actions
- **`/search` (BAP -> Gateway)**: Sends the search for the buyer's city and the domain.
- **`/on_search` (BPP -> BAP)**: Returns the catalog: store details and serviceability, categories, items with prices and statutory details, and any offers.

---

### Flow 16: Discovery Flow – Incremental Catalog (Push)

#### 1. Short Description
After the first search, the seller pushes catalog changes as they happen, without resending the whole catalog.

#### 2. API Breakdown & Technical Actions
- **`/search` (BAP -> Gateway)**: Starts the catalog session for incremental updates.
- **`/on_search` (BPP -> BAP)**: Returns the catalog: store details and serviceability, categories, items with prices and statutory details, and any offers.
- **`/on_search` [unsolicited] (BPP -> BAP)**: Pushes an incremental catalog update, such as changed prices, stock or store timings.
- **`/on_search` [unsolicited] (BPP -> BAP)**: Pushes an incremental catalog update, such as changed prices, stock or store timings.
- **`/search` (BAP -> Gateway)**: A follow-up search to confirm the catalog is in sync.

---

### Flow 17: Discovery Flow – Incremental Catalog Refresh (Pull)

#### 1. Short Description
The buyer app asks the seller for only the items that changed after a given time.

#### 2. API Breakdown & Technical Actions
- **`/search` (BAP -> Gateway)**: Sends the search for the buyer's city and the domain.
- **`/on_search` (BPP -> BAP)**: Returns the catalog: store details and serviceability, categories, items with prices and statutory details, and any offers.
- **`/search` (BAP -> BPP)**: Asks for only the items changed after a given time.
- **`/on_search` (BPP -> BAP)**: Returns only the changed items.

---

### Flow 18: Delivery Flow with IGM 1.0.0

#### 1. Short Description
A delivered order followed by a complaint handled under IGM 1.0.0.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-delivered`**: As in Flow 1 (order placed and delivered).
- **`/issue` (BAP -> BPP)**: The buyer opens a complaint.
- **`/on_issue` (BPP -> BAP)**: The seller acknowledges it and starts processing.
- **`/on_issue` [unsolicited] (BPP -> BAP)**: The seller sends the resolution.
- **`/issue` (BAP -> BPP)**: The buyer closes the issue.

---

### Flow 19: Delivery Flow with IGM 2.0.0 – No Action

#### 1. Short Description
A complaint under IGM 2.0.0 that the seller resolves with an explanation and no further action, which the buyer accepts.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-delivered`**: As in Flow 1 (order placed and delivered).
- **`/issue` (BAP -> BPP)**: The buyer opens a complaint.
- **`/on_issue` (BPP -> BAP)**: The seller acknowledges it and starts processing.
- **`/on_issue` [unsolicited] (BPP -> BAP)**: The seller proposes a resolution with no further action.
- **`/issue` (BAP -> BPP)**: The buyer accepts the resolution.
- **`/on_issue` (BPP -> BAP)**: The issue is marked resolved.
- **`/issue` (BAP -> BPP)**: The buyer closes the issue.
- **`/on_status` [unsolicited] (BPP -> BAP)**: final order status after the complaint is closed.

---

### Flow 20: Delivery Flow with IGM 2.0.0 – Rejection and Escalation

#### 1. Short Description
The buyer rejects the seller's resolution and escalates the complaint. The final resolution leads to a return.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-delivered`**: As in Flow 1 (order placed and delivered).
- **`/issue` (BAP -> BPP)**: The buyer opens a complaint.
- **`/on_issue` (BPP -> BAP)**: The seller acknowledges it and starts processing.
- **`/on_issue` [unsolicited] (BPP -> BAP)**: The seller proposes a resolution.
- **`/issue` (BAP -> BPP)**: The buyer rejects the resolution.
- **`/on_issue` (BPP -> BAP)**: The seller reprocesses the issue.
- **`/on_issue` [unsolicited] (BPP -> BAP)**: The seller proposes a new resolution.
- **`/issue` (BAP -> BPP)**: The buyer escalates the issue.
- **`/on_issue` (BPP -> BAP)**: The escalated issue is being processed.
- **`/on_issue` [unsolicited] (BPP -> BAP)**: A final resolution is proposed.
- **`/issue` (BAP -> BPP)**: The buyer accepts the resolution.
- **`/on_issue` (BPP -> BAP)**: The issue is marked resolved.
- **`/issue` (BAP -> BPP)**: The buyer closes the issue.
- **`/on_update` [unsolicited] (BPP -> BAP)**: The seller starts the replacement on the order.

---

### Flow 21: Delivery Flow with IGM 2.0.0

#### 1. Short Description
A complaint under IGM 2.0.0 where the seller asks for more information, proposes a replacement, and the buyer accepts.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-delivered`**: As in Flow 1 (order placed and delivered).
- **`/issue` (BAP -> BPP)**: The buyer opens a complaint.
- **`/on_issue` (BPP -> BAP)**: The seller acknowledges it and starts processing.
- **`/on_issue` [unsolicited] (BPP -> BAP)**: The seller asks for more information.
- **`/issue` (BAP -> BPP)**: The buyer provides the information.
- **`/on_issue` (BPP -> BAP)**: The seller acknowledges the information.
- **`/on_issue` [unsolicited] (BPP -> BAP)**: The seller proposes a resolution.
- **`/issue` (BAP -> BPP)**: The buyer accepts the resolution.
- **`/on_issue` (BPP -> BAP)**: The issue is marked resolved.
- **`/issue` (BAP -> BPP)**: The buyer closes the issue.
- **`/on_update` [unsolicited] (BPP -> BAP)**: The seller starts the return on the order.
