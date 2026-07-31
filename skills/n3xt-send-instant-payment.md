---
name: Send an instant USD payment with N3XT
description: Authenticate, pick a funding wallet, and send an instant business-to-business USD payment on the N3XT network, then confirm settlement.
api: openapi/n3xt-openapi-original.json
operations:
  - Wallets
  - Instant Payment
  - Get Payment by Id
---

# Send an instant USD payment with N3XT

Use this skill to move real USD between businesses on the N3XT network in real time.

## Prerequisites
- A N3XT business account with API access. Get a `client_id` / `client_secret`
  under **Settings > API Keys** (per environment: production, beta, or omega).
- Test in **beta** or **omega** first (see `sandbox/`).

## 1. Get a bearer token
Exchange your client credentials at the N3XT authorization server (OAuth 2.0
`client_credentials`):

    POST https://authn.n3xt.io/v1/public/{project}/oauth2/token
    grant_type=client_credentials

The response returns `access_token` (a bearer JWT) and `expires_in: 3600`.
Send it on every API call as `Authorization: Bearer <token>`.

## 2. Choose the funding wallet
Call **Wallets** (`GET /wallets`) to list the wallets for your business and pick
the `id` of the wallet you will send from. Confirm its `balance` covers the amount.

## 3. Send the payment
Call **Instant Payment** (`POST /payments/instant`) with:

    {
      "amount": "1000",
      "senderWalletId": "<wallet id from step 2>",
      "recipientBusinessId": "<counterparty business id>",
      "description": "Payment for services"
    }

The endpoint returns **202 Accepted** — the payment is processed asynchronously.

## 4. Confirm settlement
Poll **Get Payment by Id** (`GET /payments/{id}`) and watch `status` advance
through the payment lifecycle (`queued` -> `initiated` -> `released` ->
`completed`). Treat `declined` or `failed` as terminal failures.

## Rules
- Auth: bearer JWT only; refresh before the 1-hour expiry.
- No idempotency key is supported — do not blindly retry a 202; check
  `GET /payments` first to avoid double-sends (see `conventions/`).
- Errors return `{ "error": "<message>" }` with standard HTTP status codes
  (see `errors/`).
