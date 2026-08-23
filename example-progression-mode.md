---
layout: page
title: Complete Example
---

TODO: rewrite this page from scratch.

The previous example was based on an older generated API and is intentionally
removed from the public guide until it can be rebuilt around the current Lua API:

- `init`, `update`, and `draw`.
- `api.session.variant_id`.
- `api.session.selection`.
- `api.save`.
- `api.progress.unlock_milestone`.
- `api.progress.has_milestone`.
- `api.ranking.submit_score`.
- `api.session.end_game`.

A complete example should submit its deterministic score and duration before
`api.session.end_game()`. Ranking queries and the final player-facing ranking
display are outside the Lua API v1 contract.
