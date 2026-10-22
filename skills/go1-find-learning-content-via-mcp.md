---
name: go1-find-learning-content-via-mcp
description: Find Go1 learning content for a topic or role, through the official Go1 MCP server or the equivalent REST operations.
api:
  - openapi/go1-learning-objects-api-openapi.yml
  - openapi/go1-playlists-api-openapi.yml
mcp: https://mcp.go1.com/mcp
operations:
  - learning-objects-v3-search
  - getRetiringContentAlternatives
  - playlistController_getPlaylistDetails
  - playlistController_getPlaylistLos
---

# Find Go1 learning content for a topic or role

Source: https://help.go1.com/en/articles/15469776-connecting-to-the-go1-mcp-server (documented MCP tools) and the Go1 2025-01-01 OpenAPI.

## Auth
- MCP: `https://mcp.go1.com/mcp`, `Authorization: Bearer <Go1 user JWT>`. Unauthenticated calls return 401, and the server advertises `https://mcp.go1.com/.well-known/oauth-protected-resource`.
- REST: `https://gateway.go1.com`, `Authorization: Bearer {token}`, `Api-Version: 2025-01-01`, scope `lo.read`.
- Portal content entitlements apply on both surfaces, so you only see content the portal can access.

## Steps
1. **Understand the search space.** MCP: `get_content_facets`. REST: `GET /learning-objects` (`learning-objects-v3-search`) with `keyword` and `facets[]`.
2. **Search.** MCP: `search_content`, limit 10-20. REST: `learning-objects-v3-search` with `keyword`, `language[]`, `providers[]`, `entry_level[]`, `duration[][]` and `skills[]`.
3. **For a role,** call the MCP tool `suggest_skills` first, then pass the returned skills as `skills[]` filters. This has no REST equivalent.
4. **Curated playlists.** MCP: `get_playlist_details`. REST: `GET /playlists/{id}` (`playlistController_getPlaylistDetails`) and `GET /playlists/{id}/learning-objects` (`playlistController_getPlaylistLos`).
5. **Replace decommissioned content.** MCP: `get_content_alternatives`. REST: `GET /learning-objects/{id}/alternatives` (`getRetiringContentAlternatives`).

## Rules
- All of these calls are read-only, so retries are safe.
- When a response is cut off or times out, lower `limit` and narrow the filters (per the MCP troubleshooting table).
- On a 401, refresh the token. On a 403, check the portal's entitlements.
