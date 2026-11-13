# Punchout Suite

Three tools for testing supplier-side procurement integrations end to end: **cXML and SAP OCI punchout**, purchase orders, ship notices, invoices, and payment. One tool plays the buyer, one plays the supplier, and one acts as a full Coupa, Ariba, or SAP procurement system.

> **Source code is private.** This repo is a showcase of what the tools do. Happy to walk through the code or the architecture on a call.

| Tool | What it is | Live |
|---|---|---|
| **Punchout Headquarters** | Buyer-side tester for cXML punchout, SAP OCI, and cXML orders | [punchouthq.com](https://punchouthq.com) |
| **Punchout Supply** | Magento 2 / Mage-OS extension that turns a store into a punchout supplier | [supply.punchouthq.com](https://supply.punchouthq.com) |
| **Punchout Procure** | Mock Coupa, SAP Ariba, and SAP S/4HANA buyer that runs full procure-to-pay | [procure.punchouthq.com](https://procure.punchouthq.com) |

## Why I built it

Suppliers usually can't test a punchout connection until a customer's procurement team is ready, and when something breaks, the error is buried in someone else's system. These tools let a supplier run every step on their own schedule and see the raw message behind each one.

## How the pieces fit

```mermaid
sequenceDiagram
    participant B as Buyer (Procure / Headquarters)
    participant S as Supplier store (Punchout Supply)
    B->>S: PunchOutSetupRequest (cXML) or OCI login
    S-->>B: Start page URL
    Note over S: Buyer shops with contract pricing
    S-->>B: PunchOutOrderMessage / OCI cart
    Note over B: Requisition, approval chain
    B->>S: OrderRequest (cXML PO)
    S-->>B: ConfirmationRequest
    S-->>B: ShipNoticeRequest
    S-->>B: InvoiceDetailRequest
    Note over B: Goods receipt, 3-way match
    B->>S: PaymentRemittanceRequest
```

---

## Punchout Headquarters

Tests procurement connections from the buyer's side. Sends cXML PunchOutSetupRequests and SAP OCI launches, opens the supplier's catalog, and captures the returned cart. One click turns that cart into a cXML OrderRequest. A built-in mock supplier lets you run every flow without a live endpoint.

![Punchout Headquarters home](images/hq-home.png)

A full punchout session: request, supplier response, and returned cart, with every message saved.

![Session trace](images/hq-session-trace.png)

The cXML tester. Basic mode builds the request from a form; advanced mode edits the raw XML.

![cXML tester](images/hq-cxml-tester.png)

**Stack:** Node.js, Express, Vercel

---

## Punchout Supply

A Magento 2 / Mage-OS extension (`PunchoutGateway_Supplier`) that makes any store a punchout supplier. The demo store runs in production mode on an Oracle Cloud server: [supply.punchouthq.com](https://supply.punchouthq.com).

- Buyers punch in from Coupa, Ariba, or SAP over cXML or OCI
- Contract pricing per buyer, cart sent back to their system
- POs become real Magento orders
- Confirmations, ship notices, invoices, and credit memos sent back as cXML
- Per-buyer setup: encrypted shared secrets, UOM and UNSPSC mapping
- Session timelines and logs of every message

![Demo supplier store](images/supply-store.png)

One PO from received to paid, with the cXML at each step.

![Procure-to-pay](images/supply-procure-to-pay.png)

Every punchout session is logged step by step with the raw message, so it's easy to see where it breaks.

![Session timeline](images/supply-session-timeline.png)

Per-buyer cXML credentials and OCI logins. Shared secrets are stored encrypted and never shown again.

![Buyer setup](images/supply-buyer-setup.png)

**Stack:** PHP, Magento 2 / Mage-OS, Hyvä, Oracle Cloud (Ubuntu, arm64)

---

## Punchout Procure

A mock procurement system that behaves like the buyer platforms suppliers have to work with. It runs the full cycle against a supplier store: punchout requisition, approval chain, cXML PO, confirmation, ship notice, invoice, goods receipt, three-way match with price tolerance, and payment.

A Coupa PO from sent to paid, with the cXML for each step.

![Order complete](images/procure-order-complete.png)

Cart returned by cXML punchout, landing on a requisition with commodity codes, UOM, and account coding.

![Requisition](images/procure-requisition.png)

Approval chain by amount, with a preview of the OrderRequest the supplier will receive.

![Approval chain](images/procure-approval.png)

Three-way match catches an invoice billed before goods were received, with options to request a credit, short pay, or reject.

![Three-way match](images/procure-three-way-match.png)

Three buyer environments, each with its own rules: Coupa and SAP Ariba over cXML, SAP S/4HANA over OCI.

![Environments](images/procure-environments.png)

**Stack:** Node.js, Express, EJS, Redis, Vercel

---

## About me

I build supplier-side procurement integrations: punchout, catalogs, POs, and invoicing with Coupa, SAP Ariba, Oracle, and Zycus, plus the NetSuite scripting behind them. Available for consulting on Upwork.
