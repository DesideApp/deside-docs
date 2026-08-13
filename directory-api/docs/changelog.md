# Changelog

## Unreleased

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
