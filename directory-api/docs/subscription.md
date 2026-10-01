# Subscription And Billing

A Directory API project moves to a paid tier, `developer` or `pro`, through a
recurring USDC authorization on Solana signed by the owner wallet. Deside
never signs and never holds the key: the backend returns an unsigned
transaction, your wallet signs it, and the backend reads the chain before it
stores anything.

All five routes use the console proof described on
[Owner console](console.md). The project is always resolved from the wallet
inside the proof.

## The flow

1. `POST /api/v1/directory/subscription/intent` with `{ "tier": "developer" }`
   returns an unsigned `transaction`, a `step`, the `delegation` it will create
   and the `plan` terms (`amount`, `mint`, `periodSeconds`). `step` is
   `init_authority` while a setup transaction is still missing, and
   `create_delegation` when this transaction grants the recurring charge.
   Calling it changes nothing and can be repeated.
2. Your wallet signs and sends the transaction.
3. `POST /api/v1/directory/subscription/accept` with the same `tier` confirms
   on chain what you signed and stores it.
4. **The tier changes when the first charge settles, not when you accept.**
   Accepting grants the charge; the first charge grants the tier.

## Routes

| Method and path | What it does |
| --- | --- |
| `GET /api/v1/directory/subscription` | Returns `enabled`, `tier`, `billingStatus`, `subscription` and `plan`. |
| `POST /api/v1/directory/subscription/intent` | Step 1 above. |
| `POST /api/v1/directory/subscription/accept` | Step 3 above. |
| `POST /api/v1/directory/subscription/cancel-intent` | Returns the unsigned transaction that revokes the recurring authorization. Available even when new subscriptions are closed. |
| `POST /api/v1/directory/subscription/cancel-confirm` | Confirms the revocation on chain. The paid tier stays until the paid period ends. |

`subscription.status` is `none`, `active`, `past_due` or `canceled`. `plan`
describes the on-chain plan (`planAddress`, `amount` in base units, `mint`,
`periodDays`) and is `null` when those terms cannot be read. When the mint is
one Deside knows, `plan` also carries `mintSymbol` and `mintDecimals`. The
signed figure is always `amount`, in base units.

## Billing rules

* One cycle is 30 days (`periodSeconds` 2,592,000).
* Changing tier is not done in place: cancel the current subscription and
  subscribe again. The new subscription charges on the day it settles and
  starts a new 30-day cycle.
* A project on a manual billing agreement cannot use this flow.

## Errors

Errors use the shape `{ "error": { code, message, requestId } }`.

| Status | Code | When |
| --- | --- | --- |
| `503` | `dapi_subs_disabled` | Subscriptions are not open. |
| `400` | `dapi_subs_tier_invalid` | `tier` is not `developer` or `pro`. |
| `400` | `dapi_subs_billing_manual` | The project is on a manual billing agreement. |
| `403` | `dapi_subs_payer_not_owner` | The signer is not the project owner. |
| `403` | `dapi_subs_project_blocked` | The project is blocked. |
| `404` | `dapi_subs_project_not_found` | The wallet has no project. Create a key first. |
| `409` | `dapi_subs_already_active` | The project already has that plan. |
| `409` | `dapi_subs_tier_change_not_supported` | Another tier is active. Cancel it first. |
| `409` | `dapi_subs_delegation_not_found` | No signed delegation was found on chain. |
| `409` | `dapi_subs_delegation_terms_mismatch` | The signed delegation does not match the plan. |
| `409` | `dapi_subs_delegation_already_used` | The delegation already pays for something else. |
| `409` | `dapi_subs_state_changed` | The subscription changed during the request. Retry. |
| `409` | `dapi_subs_not_cancelable` | There is no active subscription to cancel. |
| `409` | `dapi_subs_delegation_still_active` | The delegation is still live on chain. Sign the cancellation first. |
| `409` | `dapi_subs_delegation_already_revoked` | The delegation is already revoked. Confirm the cancellation. |
| `502` | `dapi_subs_unavailable` | Anything else. Retry later. |
