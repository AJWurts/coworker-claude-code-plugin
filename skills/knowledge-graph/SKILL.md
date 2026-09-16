---
name: knowledge-graph
description: Explore the company's OM2 organizational knowledge graph — the fastest way to understand people, projects, customers, and how they connect. Use for open-ended "what do we know about X", "how is A related to B", "what's the history of Y", or "what are the themes around Z" questions. Only applies when OM2 tools (om2_*) are available in this session; if they aren't, the network hasn't enabled OM2 — fall back to memory_retrieval and connector search.
---

# Organizational knowledge graph (OM2)

OM2 is a graph built from the company's connected data. It finds facts that are *indirectly* connected to your query, so it beats single-source search for understanding context. Use it before falling back to per-connector tools when the question is about organizational knowledge.

For the full tool-selection guide (completeness/pagination signals, search modes, anti-loop rules), retrieve the `mcp-om2-usage` skill from Coworker with `skill_retrieve`.

## Entry routing — pick the right tool by question shape

- **What happened / what do we know** → `om2_search` (set `time_start` for "recent", "yesterday", "this week")
- **Named account, deal, or contact** → `om2_entity_brief`
- **A person (including "me" / "my")** → `om2_search` with the person's NAME in the query plus a time window, never the literal "I"; exhaustive per-person record → `om2_identify_people` then `om2_user_activity`; collaborators → `om2_person_network`
- **"All / every / list / how many"** → `om2_enumerate` (page until `has_more=false`); enumerate before counting
- **"What's new" with no topic** → `om2_recent_activity`
- **One specific fact** → `om2_atomic_data`
- **Full history of one entity** → `om2_entity_timeline`
- **One node's full profile** → `om2_node_details`
- **Provenance / source documents** → `om2_source_trace` / `om2_document_explore` / `om2_report_explore`
- **Anything else** → `om2_graph_schema` then `om2_cypher` (~10s; try search/enumerate first)

### Live-data branch

Questions about "current status", "right now", "today", "open tickets", or "most recent X": **ONE** `om2_search` to orient (learn what entities exist and what the graph knows), then the owning connector tool for the live fact itself. Always.

## Reading the envelope — is one query enough?

Every OM2 result carries an envelope: `status`, `resolution`, `retrieval`, `warnings`.

- **`resolution.entities`** — what the query anchored to. Trust these over fuzzy matches.
- **`retrieval.has_more`** — `true` means top_k filled. Narrow (time window, source filter) or switch to `om2_enumerate`. Never report a `has_more=true` result as "complete".
- **`retrieval.activated_themes`** — sparse but relevant themes? Retry with `search_mode=theme_deep`.
- **`similarity`** on results — all low values means the graph has nothing on the query. Stop and try a connector.
- **`suggestions`** (on empty results) — nearest entities, matched themes, available connectors, and a hint about what to try next. Read these instead of rephrasing and retrying.

## When you've searched enough — stopping rules

- **Max 3 OM2 calls per sub-question.** If three queries on the same topic return nothing useful, the graph doesn't have it.
- **No near-duplicate re-queries.** Rephrasing a query that returned zero results is almost never productive — 104 such retries in benchmarking produced zero new data. Change the tool or the approach instead.
- **No dead-end graph hops.** Never call `om2_node_details` or `om2_source_trace` for IDs you won't reference again in your answer.

## When to go to connector tools

- **After 1 empty/thin OM2 result** on live-data or exact-aggregate questions.
- **After 2 empty results** on anything else.
- **Immediately** for JQL/SQL/CRM report shapes (the source computes these, not the graph).
- **Always name the source** you used ("according to Jira", "from the CRM").

## If OM2 tools are absent

The network hasn't enabled OM2 (the `enableOM2` flag is off). Don't claim graph results — use `memory_retrieval` and connector search instead.
