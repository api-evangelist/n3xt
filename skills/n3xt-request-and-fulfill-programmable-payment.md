---
name: Create and fulfill a programmable payment with N3XT
description: Create a conditional (programmable) payment money-flow, track its status, and fulfill or cancel it on the N3XT network.
api: openapi/n3xt-openapi-original.json
operations:
  - Create Programmable Payment Request
  - List Programmable Payments
  - Fulfill Programmable Payment
  - Cancel Programmable Payment
---

# Create and fulfill a programmable payment with N3XT

Programmable payments are conditional money-flows: a payment is created, then
fulfilled (released) or cancelled once conditions are met.

## 1. Authenticate
Get a bearer token via OAuth 2.0 `client_credentials` at
`https://authn.n3xt.io/v1/public/{project}/oauth2/token` and send it as
`Authorization: Bearer <token>` (see the send-instant-payment skill / `authentication/`).

## 2. Create the programmable payment
Call **Create Programmable Payment Request**
(`POST /payments/programmable?initiator=payer` or `initiator=recipient`).

For a payer-initiated flow the body is a PayerRequestPayload:

    {
      "counterPartyId": "<counterparty business uuid>",
      "senderWalletId": "<your wallet uuid>",
      "amount": "1000",
      "description": "Milestone 1"
    }

The endpoint returns **202 Accepted** with the money-flow `id`.

## 3. Track it
Call **List Programmable Payments** (`GET /payments/programmable`) with
`limit` / `offset` / `orderBy` / `order` to page results, or
`GET /payments/programmable/{id}` for one money-flow. Watch `status`.

## 4. Fulfill or cancel
- **Fulfill Programmable Payment** (`PUT /payments/programmable/{id}/fulfillment`)
  releases the payment. Returns 202.
- **Cancel Programmable Payment** (`PUT /payments/programmable/{id}/cancellation`)
  cancels it. Returns 202.

## Rules
- Money-movement steps are asynchronous (202) and may require an approval
  workflow before they settle — check the audit trail / approvals surface.
- No idempotency key; verify state with a GET before retrying.
- Errors: `{ "error": "<message>" }` (see `errors/`).
