# Retail Electronics (ONDC:RET14 1.2.5)

## Overview

Retail Electronics (`ONDC:RET14`) covers the online sale of mobile phones, computers, audio devices, cameras, accessories and other consumer electronics.

Electronics are high-value products with detailed specifications, brand warranties and serial numbers. Buyers often use financing or EMI options, and returns and replacements usually involve a quality check of the product.

ONDC lets buyer apps and seller apps connect through a common, open protocol instead of building separate integrations with each other. A buyer app integrates once with the network and can show catalogs from many sellers. A seller app integrates once and can reach buyers on every participating buyer app.

The seller remains responsible for its catalog, prices, stock, packing, delivery and after-sales service. ONDC standardizes the network messages used to discover products, place orders, track delivery, and handle cancellations, returns and grievances.

This guide explains the electronics journey from a business point of view first, and then covers the API flows defined in this specification (version 1.2.5).

---

## What's New in 1.2.5

Compared with version 1.2.0, the Retail Electronics 1.2.5 specification adds:

| Area | What it adds |
|---|---|
| **Payments and commercials** | Payment collected by the seller app, purchase finance (EMI). |
| **Products and sellers** | Custom text input on items. |
| **Reverse flows** | Replacement with reverse QC, forced cancellation, seller cancellation with RTO. |

---

## Participants

| Participant | What This Means |
|---|---|
| **Buyer App (BAP)** | The consumer-facing application where the buyer searches for electronics, builds a cart, pays, tracks the order and raises complaints. It can be a shopping app, payments app, or any app that offers electronics to its users. |
| **Seller App (BPP)** | The application representing sellers such as electronics retailers, brand stores and authorised dealers. It publishes the catalog, checks stock and serviceability, prices the order, confirms it, manages packing and delivery, and handles cancellations, returns and replacements. |
| **Logistics Provider** | The delivery partner that picks up and delivers the order. Usually the seller arranges delivery. |
| **Gateway** | Network infrastructure that forwards a buyer app's `/search` to all relevant seller apps based on domain and city. |

---

## The User Journey

A buyer discovers sellers, builds a cart, pays, and receives the order with status updates along the way. The journey has the following stages.

### 1. Discovery and Catalog

The buyer searches for an item, a category (for example "Smartphone" or "Bluetooth headphones") or a seller. The buyer app sends the search with the buyer's location.

Seller apps that serve that location reply with their catalog, which can include:

- Store details and serviceable area
- Products with images, specifications, brand and model
- Variants such as colour, storage or memory
- MRP, selling price and stock availability
- Warranty, return and replacement terms, and statutory details such as manufacturer and country of origin

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
| **Order-picked-up** | The agent has collected the order from the seller. |
| **Out-for-delivery** | The agent is on the way to the buyer. |

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

### 1. SNP Collecting Payment

Instead of the buyer app collecting payment, the seller app (SNP) collects it. The seller shares the payment terms at `/on_init` and can send an unsolicited `/on_init` with the payable amount; the buyer app then confirms the order.

### 2. Purchase Finance

The buyer buys using a purchase finance option such as EMI. The finance and offer terms are carried in `/init` and `/on_init` and fixed at `/confirm`, after which the order is fulfilled as usual.

### 3. Customization Input Text

For items that support it, the buyer enters custom text (for example a name to print or engrave) during `/select`. The seller includes the customisation in the quote and fulfills the customised item.

### 4. Out of Stock

If an item is not available when the buyer selects it, the seller can handle it in two ways:

- **Cart correction**: The seller marks the unavailable item in `/on_select`. The buyer changes the cart, selects again, and continues with the order.
- **Error response**: The seller rejects the selection in `/on_select` with an out-of-stock error code (for example `40002`).

### 5. Buyer Cancellation

The buyer cancels the order before it is shipped, with a cancellation reason. The seller checks its cancellation terms, cancels the order, and shares any cancellation charges and the refund amount.

### 6. Seller Cancellation

After confirmation, the seller can cancel items it cannot supply through an unsolicited `/on_update`, and the buyer app updates the settlement. If delivery of the rest then fails, the seller cancels the order as Return to Origin (RTO), the buyer app updates the settlement again, and the seller confirms the returned order is back or disposed of.

### 7. Forced Cancellation

Once an order is out for delivery, a normal cancellation request can be refused. The buyer app can then raise a forced cancellation, the seller cancels the order, and the buyer app updates the settlement for the refund.

### 8. Return

After delivery, the buyer can return eligible items (the full order or some items) within the return window. The seller approves the return, the items are picked up, and the refund is settled once the seller receives them.

### 9. Replacement

The buyer asks for a replacement of a damaged or wrong item. The seller approves it, a reverse quality check (QC) is done at pickup, the item goes back to the seller, and a replacement is packed and delivered.

### 10. Incremental Catalog Updates

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
| 2 | SNP Collecting Payment Flow | Mandatory | The seller app, instead of the buyer app, collects the payment |
| 3 | Purchase Finance Flow | Mandatory | The buyer pays using a purchase finance (EMI or credit) option |
| 4 | Customization Input Text Flow | Mandatory | The buyer adds custom text to an item (for example a name to print or engrave) while selecting it, and the seller prices and fulfills the customised item |
| 5 | Out of Stock Flow | Mandatory | An item turns out to be out of stock when the buyer selects it |
| 6 | Out of Stock Flow (Error Code) | Mandatory | The seller rejects the selection straight away because an item is out of stock |
| 7 | Buyer Cancellation Flow | Mandatory | The buyer cancels a confirmed order before it is shipped |
| 8 | Seller Cancellation Flow | Mandatory | The seller cancels part of the order after confirmation, and later the remaining order is cancelled as Return to Origin (RTO) when delivery fails |
| 9 | Forced Cancellation Flow | Mandatory | The order is already out for delivery when the buyer wants to cancel |
| 10 | Return Flow | Mandatory | After delivery, the buyer returns an item |
| 11 | Replacement Flow | Optional | After delivery, the buyer asks for a replacement |
| 12 | Discovery Flow – Full Catalog | Mandatory | The buyer app searches a city and the seller returns its full catalog |
| 13 | Discovery Flow – Incremental Catalog (Push) | Mandatory | After the first search, the seller pushes catalog changes as they happen, without resending the whole catalog |
| 14 | Discovery Flow – Incremental Catalog Refresh (Pull) | Optional | The buyer app asks the seller for only the items that changed after a given time |
| 15 | Delivery Flow with IGM 1.0.0 | Mandatory | A delivered order followed by a complaint handled under IGM 1.0.0 |
| 16 | Delivery Flow with IGM 2.0.0 – No Action | Mandatory | A complaint under IGM 2.0.0 that the seller resolves with an explanation and no further action, which the buyer accepts |
| 17 | Delivery Flow with IGM 2.0.0 – Rejection and Escalation | Mandatory | The buyer rejects the seller's resolution and escalates the complaint |
| 18 | Delivery Flow with IGM 2.0.0 | Mandatory | A complaint under IGM 2.0.0 where the seller asks for more information, proposes a replacement, and the buyer accepts |

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
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `Order-picked-up`.
- **`/track` & `/on_track` (BAP <-> BPP)**: The buyer app requests and receives live tracking details.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Out-for-delivery` -> `Order-delivered`.

---

### Flow 2: SNP Collecting Payment Flow

#### 1. Short Description
The seller app, instead of the buyer app, collects the payment. The seller shares the payment details at `/on_init`, the buyer pays, and the order continues as usual.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote (`quote.breakup`) with item prices, delivery and packing charges, taxes, and the expected delivery time (`@ondc/org/TAT`).
- **`/init` (BAP -> BPP)**: Sends the delivery address, contact details and billing details.
- **`/on_init` (BPP -> BAP)**: Returns the final quote with payment terms where the seller collects the payment.
- **`/on_init` [unsolicited] (BPP -> BAP)**: Sends the payable amount for the seller-collected payment.
- **`/confirm` (BAP -> BPP)**: Places the order after the payment collected by the seller.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `Order-picked-up` -> `Out-for-delivery` -> `Order-delivered`.

---

### Flow 3: Purchase Finance Flow

#### 1. Short Description
The buyer pays using a purchase finance (EMI or credit) option. The finance terms are shared during `/init` and `/on_init`, and the order is then fulfilled as usual.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote (`quote.breakup`) with item prices, delivery and packing charges, taxes, and the expected delivery time (`@ondc/org/TAT`).
- **`/init` (BAP -> BPP)**: Sends the order details with the purchase finance option.
- **`/on_init` (BPP -> BAP)**: Returns the final quote with the finance and offer terms.
- **`/confirm` (BAP -> BPP)**: Places the order with the purchase finance option.
- **`/on_confirm` (BPP -> BAP)**: Confirms the financed order.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `Order-picked-up` -> `Out-for-delivery` -> `Order-delivered`.

---

### Flow 4: Customization Input Text Flow

#### 1. Short Description
The buyer adds custom text to an item (for example a name to print or engrave) while selecting it, and the seller prices and fulfills the customised item.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends the item with the buyer's custom text.
- **`/on_select` (BPP -> BAP)**: Returns the quote including the customisation.
- **`/init` (BAP -> BPP)**: Sends the delivery address, contact details and billing details.
- **`/on_init` (BPP -> BAP)**: Returns the final quote, order terms, and payment and settlement details.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `Order-picked-up` -> `Out-for-delivery` -> `Order-delivered`.

---

### Flow 5: Out of Stock Flow

#### 1. Short Description
An item turns out to be out of stock when the buyer selects it. The buyer corrects the cart and completes the order.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends a cart that includes an out-of-stock item.
- **`/on_select` (BPP -> BAP)**: Reports the unavailable item with an out-of-stock error (for example `40002`).
- **`/select` (BAP -> BPP)**: Sends the selected items, quantities, store location (`location_id`) and delivery location (GPS and area code).
- **`/on_select` (BPP -> BAP)**: Returns the quote (`quote.breakup`) with item prices, delivery and packing charges, taxes, and the expected delivery time (`@ondc/org/TAT`).
- **`/init` (BAP -> BPP)**: Sends the delivery address, contact details and billing details.
- **`/on_init` (BPP -> BAP)**: Returns the final quote, order terms, and payment and settlement details.
- **`/confirm` (BAP -> BPP)**: Places the order with payment status `PAID`.
- **`/on_confirm` (BPP -> BAP)**: Confirms the order (`Created`, fulfillment `Pending`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `Order-picked-up` -> `Out-for-delivery` -> `Order-delivered`.
- **`/track` & `/on_track` (BAP <-> BPP)**: The buyer app requests and receives live tracking details.

---

### Flow 6: Out of Stock Flow (Error Code)

#### 1. Short Description
The seller rejects the selection straight away because an item is out of stock.

#### 2. API Breakdown & Technical Actions
- **`/select` (BAP -> BPP)**: Sends a cart that includes an out-of-stock item.
- **`/on_select` (BPP -> BAP)**: Reports the unavailable item with an out-of-stock error (for example `40002`).

---

### Flow 7: Buyer Cancellation Flow

#### 1. Short Description
The buyer cancels a confirmed order before it is shipped.

#### 2. API Breakdown & Technical Actions
- **`/select` to `/on_confirm`**: As in Flow 1.
- **`/cancel` (BAP -> BPP)**: Sends the order ID and the cancellation reason code.
- **`/on_cancel` (BPP -> BAP)**: Cancels the order (`Cancelled`) and returns any cancellation charges and the refund amount.

---

### Flow 8: Seller Cancellation Flow

#### 1. Short Description
The seller cancels part of the order after confirmation, and later the remaining order is cancelled as Return to Origin (RTO) when delivery fails.

#### 2. API Breakdown & Technical Actions
- **`/select` to `/on_confirm`**: As in Flow 1.
- **`/on_update` [unsolicited] (BPP -> BAP)**: The seller cancels the unavailable items.
- **`/update` (BAP -> BPP)**: The buyer app updates the settlement for the partial refund.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Pending` -> `Packed` -> `Agent-assigned` -> `Order-picked-up` -> `Out-for-delivery`.
- **`/on_cancel` [unsolicited] (BPP -> BAP)**: Delivery fails and the seller cancels the order as RTO (reason code `013`).
- **`/on_status` [unsolicited] (BPP -> BAP)**: `RTO-Delivered / RTO-Disposed`.

---

### Flow 9: Forced Cancellation Flow

#### 1. Short Description
The order is already out for delivery when the buyer wants to cancel. The buyer app forces the cancellation and the refund is settled.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Out-for-delivery`.
- **`/cancel` (BAP -> BPP)**: A normal cancellation request, which is not accepted at this stage.
- **`/cancel` (BAP -> BPP)**: A forced cancellation request.
- **`/on_cancel` (BPP -> BAP)**: The seller cancels the order.
- **`/update` [unsolicited] (BAP -> BPP)**: The buyer app updates the settlement for the refund.

---

### Flow 10: Return Flow

#### 1. Short Description
After delivery, the buyer returns an item. The seller approves the return, the item is picked up, and the refund is settled.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Out-for-delivery` -> `Order-delivered`.
- **`/update` (BAP -> BPP)**: Sends the return request with the item, quantity and return reason.
- **`/on_update` (BPP -> BAP)**: The seller records the return request.
- **`/on_update` [unsolicited] (BPP -> BAP)**: Return approved.
- **`/on_update` [unsolicited] (BPP -> BAP)**: Return picked up.
- **`/update` (BAP -> BPP)**: The buyer app updates the settlement for the refund.
- **`/on_update` [unsolicited] (BPP -> BAP)**: Return received by the seller, which completes the return.

---

### Flow 11: Replacement Flow

#### 1. Short Description
After delivery, the buyer asks for a replacement. The item is checked and picked up (reverse QC), returned to the seller, and a replacement is delivered.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Out-for-delivery` -> `Order-delivered`.
- **`/update` (BAP -> BPP)**: Sends the replacement request with reverse QC details.
- **`/on_update` (BPP -> BAP)**: The seller records the request.
- **`/on_update` [unsolicited] (BPP -> BAP)**: Return approved.
- **`/on_update` [unsolicited] (BPP -> BAP)**: Item picked up after the reverse QC.
- **`/on_update` [unsolicited] (BPP -> BAP)**: Item received by the seller.
- **`/on_status` [unsolicited] (BPP -> BAP)**: replacement `Packed` -> replacement `Order-picked-up` -> replacement `Out-for-delivery` -> replacement `Order-delivered`.

---

### Flow 12: Discovery Flow – Full Catalog

#### 1. Short Description
The buyer app searches a city and the seller returns its full catalog.

#### 2. API Breakdown & Technical Actions
- **`/search` (BAP -> Gateway)**: Sends the search for the buyer's city and the domain.
- **`/on_search` (BPP -> BAP)**: Returns the catalog: store details and serviceability, categories, items with prices and statutory details, and any offers.

---

### Flow 13: Discovery Flow – Incremental Catalog (Push)

#### 1. Short Description
After the first search, the seller pushes catalog changes as they happen, without resending the whole catalog.

#### 2. API Breakdown & Technical Actions
- **`/search` (BAP -> Gateway)**: Starts the catalog session for incremental updates.
- **`/on_search` [unsolicited] (BPP -> BAP)**: Pushes changed items, for example new prices or stock.
- **`/on_search` [unsolicited] (BPP -> BAP)**: Disables items or the store (for example when it closes).
- **`/search` (BAP -> Gateway)**: Starts the catalog session for incremental updates.

---

### Flow 14: Discovery Flow – Incremental Catalog Refresh (Pull)

#### 1. Short Description
The buyer app asks the seller for only the items that changed after a given time.

#### 2. API Breakdown & Technical Actions
- **`/search` (BAP -> Gateway)**: Sends the search for the buyer's city and the domain.
- **`/on_search` (BPP -> BAP)**: Returns the catalog: store details and serviceability, categories, items with prices and statutory details, and any offers.
- **`/search` (BAP -> BPP)**: Asks for only the items changed after a given time.
- **`/on_search` (BPP -> BAP)**: Returns only the changed items.

---

### Flow 15: Delivery Flow with IGM 1.0.0

#### 1. Short Description
A delivered order followed by a complaint handled under IGM 1.0.0.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Out-for-delivery` -> `Order-delivered`.
- **`/issue` (BAP -> BPP)**: The buyer opens a complaint.
- **`/on_issue` (BPP -> BAP)**: The seller acknowledges it and starts processing.
- **`/on_issue_status` [unsolicited] (BPP -> BAP)**: The seller sends the resolution.
- **`/issue` (BAP -> BPP)**: The buyer closes the issue.

---

### Flow 16: Delivery Flow with IGM 2.0.0 – No Action

#### 1. Short Description
A complaint under IGM 2.0.0 that the seller resolves with an explanation and no further action, which the buyer accepts.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Out-for-delivery` -> `Order-delivered`.
- **`/issue` (BAP -> BPP)**: The buyer opens a complaint.
- **`/on_issue` (BPP -> BAP)**: The seller acknowledges it and starts processing.
- **`/on_issue` [unsolicited] (BPP -> BAP)**: The seller proposes a resolution with no further action.
- **`/issue` (BAP -> BPP)**: The buyer accepts the resolution.
- **`/on_issue` (BPP -> BAP)**: The issue is marked resolved.
- **`/issue` (BAP -> BPP)**: The buyer closes the issue.
- **`/on_status` [unsolicited] (BPP -> BAP)**: final order status after the complaint is closed.

---

### Flow 17: Delivery Flow with IGM 2.0.0 – Rejection and Escalation

#### 1. Short Description
The buyer rejects the seller's resolution and escalates the complaint. The final resolution leads to a return.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Out-for-delivery` -> `Order-delivered`.
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

### Flow 18: Delivery Flow with IGM 2.0.0

#### 1. Short Description
A complaint under IGM 2.0.0 where the seller asks for more information, proposes a replacement, and the buyer accepts.

#### 2. API Breakdown & Technical Actions
- **`/select` to `Order-picked-up`**: As in Flow 1.
- **`/on_status` [unsolicited] (BPP -> BAP)**: `Out-for-delivery` -> `Order-delivered`.
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
