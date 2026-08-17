# Changelog

## Unreleased

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
