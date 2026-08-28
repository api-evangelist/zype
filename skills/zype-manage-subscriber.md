---
name: zype-manage-subscriber
description: Create a Zype consumer, entitle them to content, manage their subscription lifecycle, and cancel reversibly.
api: Zype Consumers + Monetization APIs
base_url: https://api.zype.com
spec: openapi/zype-consumers.json, openapi/zype-monetization.json
operations:
  - listConsumers
  - createConsumer
  - getConsumer
  - updateConsumer
  - deleteConsumer
  - resetPasswordFlow
  - updatePassword
  - listSubscriptions
  - createSubscription
  - getSubscription
  - updateSubscription
  - cancelSubscription
  - reactivateSubscription
  - deleteSubscription
  - listPlans
  - getPlan
  - addPlaylists
  - listTransactions
  - createTransaction
  - listRedemptionCodes
  - redeem
  - bulkCreate
  - checkVideoEntitlement
  - listVideoEntitlements
  - createVideoEntitlement
  - listPlaylistEntitlements
  - createPlaylistEntitlement
  - listSubscriptionEntitlements
  - listVideoFavorites
generated: '2026-08-28'
method: generated
source: openapi/zype-consumers.json, openapi/zype-monetization.json
---

# Manage a Zype subscriber

A **Consumer** is the end viewer. A **User** (`/site/users`) is an admin on the Zype
property — they are different objects; do not confuse them.

## Steps

1. **Create the consumer.** `POST /consumers` (`createConsumer`). Look one up first with
   `listConsumers` — there is no idempotency key, so a retried create makes a duplicate
   viewer account.
2. **Pick the plan.** `listPlans` / `getPlan`. A tiered plan is bound to content with
   `addPlaylists` / `removePlaylists`.
3. **Subscribe.** `POST /subscriptions` (`createSubscription`) with `consumer_id` and
   `plan_id`. Payment objects carry the external processor reference
   (`stripe_id`, `braintree_id`, `recurly_token_id`) — Zype does not process cards itself.
4. **Or entitle directly.** For TVOD or gifted access, use
   `createVideoEntitlement` / `createPlaylistEntitlement` instead of a subscription, and
   verify with `checkVideoEntitlement` before serving playback.
5. **Redemption codes.** `bulkCreate` mints a batch, `redeem` consumes one.
6. **Reconcile.** `listTransactions` and `listSubscriptionEntitlements`.

## Reversibility — the important part

Zype gives you two different ways to end a subscription and they are **not** equivalent:

- `cancelSubscription` — `PUT /subscriptions/{id}/cancel`. **Reversible** via
  `reactivateSubscription` (`PUT /subscriptions/{id}/reactivate`).
- `deleteSubscription` — `DELETE /subscriptions/{id}`. **No reversal exists.**

**Always cancel, never delete**, unless a human has explicitly asked for deletion and
understands it cannot be undone. Zype publishes **no reactivation window**, so do not tell
a user how long they have to change their mind — that number is not documented anywhere.

`deleteConsumer` is likewise terminal and takes the viewer's history with it.

## Password flows

`resetPasswordFlow` emails the consumer a reset; `updatePassword` completes it. Never
attempt to set a consumer password directly as part of an automated flow.
