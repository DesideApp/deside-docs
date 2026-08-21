# Changelog

## Unreleased

- documented the public product surface: `GET /api/v1/public/agents/stats-summary`
  now has a page, with what each counter counts and in which unit. The headline
  number changed meaning on 2026-08-21: `indexed` counts AGENTS in the Deside
  catalogue, not entries across registries, so it dropped from 10,825 to 10,690
  and now matches the `total` of the public list. `registered` is kept as a
  deprecated alias of the same number and will be removed
- stated the two rules that make the counters readable: `endpoints` is counted
  in URLs and never converts to agents (one URL can be declared by many
  agents), and an absent key means not measured, never `0`

- documented how liveness and verification are actually measured (Trust): the
  census handshake with its two-strike anti-flapping versus the paid daily
  check that invokes safe tools with rotation; the five failure-semantics
  rules (a failing tool never touches liveness, never removes the badge, a
  single failure is not a verdict, passes are sealed to the URL they probed,
  and the owner is notified on change)

- Ask moved to its own address: `POST /api/v1/ask` is now canonical. The old
  `POST /api/v1/directory/agents/ask` keeps answering as an alias and will be
  retired. Rationale: Ask has its own regime (no key, no quota, its own rate
  limits, and an announced pay-per-question machine lane), so it gets its own
  street instead of living inside the key-protected read surface's path

- corrected the webhook auth line in the boundary page: it said owner/session
  and the routes use the console proof. The authentication page already said it
  right, so the two pages contradicted each other
- named the collision that will confuse everyone sooner or later:
  `/directory/subscription` is the API plan of a project, and
  `/directory/agents/:catalogId/subscription` is the Verified subscription of
  one agent. Same word, same prefix, different person and different credential
- listed the agent-owner rail as deliberately out of scope, so that finding its
  eleven routes does not read as an undocumented part of this API

- corrected the auth of the owner routes: keys, usage and webhooks use the
  console proof, not a session cookie. The previous text said `protectRoute`,
  which has not been true since the console got its own signature login
- documented the console proof itself: nonce, signature, short-lived bearer,
  its own audience, and why it is a separate credential family
- documented the subscription and billing routes: the five endpoints, the
  unsigned-transaction flow, the 30-day cycle, why the tier only changes once
  the first charge settles, why cancelling stays open when selling is closed,
  that changing tier means cancel and subscribe again, and the actionable
  error codes
- documented `POST /directory/agents/ask`: no API key, no quota, human and
  headless lanes with their own rate limits, never a `500`, and the announced
  `402` for the paid machine lane
- added the two filters that the list route really accepts, `collection` and
  `collectionCase`, and stated that there is no free-text search on it
- documented the list ordering and what invalidates a cursor
- added the five item fields that were exposed but undocumented: `connected`,
  `primaryWalletSource`, `collectionBadges`, `channels` and `curationPublic`,
  the last one being where the measured facts live
- documented the verified fields in the trust response (`verified`,
  `verifiedCheck`, `verifiedCheckedAt`, `verifiedFailed`) and how to read
  them: a positive fact when present, no information when absent

- added `socialLinks` to `DirectoryAgentListItemV1` and the profile contract:
  the agent's own declared website, X, and GitHub links, resolved and
  sanitized by the backend

- aligned the capability filter vocabulary with the accepted server set
  (removed `support` and `automation` until the server accepts them)
- marked Pro webhooks and Pro bulk export as pre-rollout (documented, not yet
  enabled in production)
- corrected the trust `fairscale` example (`scoreKind` values such as
  `fairscore`, added `walletClassification`)
- reworded the public catalog auth row: it is an open product surface, not a
  legacy surface

- published the initial Directory API doc set
- documented the API-key read surface, pagination, errors, limits, data
  product fields, and boundary notes
- reconciled `api_key_revoked` to the `403` status contract
- added Pro documentation for webhook subscription management, signed
  deliveries, and `jsonl.gz` bulk exports
- documented the trust facts endpoint, including `connected`, receipt
  families, declared services, and FairScale attribution rules

## Notes

This changelog records public Directory API documentation changes.
