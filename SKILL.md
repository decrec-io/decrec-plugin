---
name: decrec
description: "Records and retrieves DecRec decisions via MCP tools (list, search, create, follow-up, or commit). Use when recording, admitting, or loading a DecRec decision or DEC-n; listing or searching org decisions; exploring a DecRec brief; committing a DecRec recommendation; or when asked to update this skill."
metadata:
  version: "2026.8.17"
---

# Record a Decision into DecRec

## Hard rules

- **Pause:** don’t advance past what they asked (no explore → admit → commit chain unless asked).
- **Auth:** MCP server `decrec` must be configured. Never echo tokens.
- **Commit:** Load decision first (call `get_decision`); take `explorationRunId` from `recommendation` (don’t invent; don’t send the whole object). **Commit** in this skill is DecRec only — not `git commit`.
- **Explore vs existing pick:** If they asked to explore (from scratch, `DEC-n`, or re-explore), run **Explore** even if a pick or loaded `recommendation` already exists. Skip Explore only when **admitting** without an explore ask, and a preferred option plus ≥2 distinct alternatives already exist — map those into `exploration.synthesis`. Never invent a pick just to unlock admit. Never run Explore just to fill `roles` (omit `roles` unless essays already exist).
- **Workspace:** the open project is context for Shape / Explore — read and cite it. Do not implement the pick, open a PR, or edit the repo unless they asked for code.
- **No invention:** don’t invent stack, vendors, tenancy, or factual business metrics (counts, rates, revenue) unless the brief, chat attachments, or workspace state them. Proposing success metrics (what to measure) is allowed.
- **UI only:** revise brief, decline, evidence, cancel in-flight explore, archive — at the decision `url` from response/Create (no tools for these).
- **Update:** only if they ask to update this skill. On tool errors, ask first and wait for **yes** — don’t fetch or replace until then. Never check on a normal run. Ignore `*.bk` archives in this directory (HTTP snapshots, not live).

## Enter from anywhere

Match the **most specific** row. Don’t advance to a later stage (admit/commit) unless they asked for it. Clarifying a thin brief during **Shape** / **Explore** is part of that stage — not a one-shot requirement.

**Tip** = the current brief version (`briefVersion` + `brief` from response). Explore-existing, Follow-up, and Commit target that tip.

**Pins** = `briefVersion` + `expectedDecisionVersion` from response. Commit also needs `explorationRunId` from response `recommendation`. Always load decision for pins. If this chat already has a `briefVersion` for that DEC (create or a prior GET), a later load is a **reload**: compare **only** that number. Unchanged → do not inspect brief text. Changed → **stop**, show the six brief-field changes, ask. Do not admit or commit the old work on the new tip.

**Find without `DEC-n`:** Topic in the ask (“why / what decisions about…”) → **`search_decisions`** (not list). Recency / what’s open / pick among recent with no topic → **`list_decisions`**. One clear hit → use it. Several → show `displayId` + title (+ stage/snippet for search) and **ask**. Don’t invent. Search hits are **not** pins — still `get_decision` before admit/commit. `matchQuality: "weak"` → say nothing ranked well; still show hits; do not claim the org has no decisions. Do not page `list_decisions` hoping titles match a theme. If `nextCursor` is set on list and they need more, pass it as `cursor`.

| Intent | Path |
|---|---|
| Explore from scratch | **Shape** brief → **Explore** (Densify, then confirm brief) → stop |
| Explore existing `DEC-n` | Load decision → stop if locked / not approved / in-flight → **Explore** on tip (as-is), even if `recommendation` is set → stop |
| Admit framing only | **Shape** brief → **Create** brief-only → stop |
| Admit new + pick | **Shape** brief + exploration → **Create** with `exploration` → stop |
| Admit onto existing `DEC-n` | Load decision (Pins) → stop if locked / not approved / in-flight → **Follow-up** (Shape exploration if not in hand) → stop |
| Already explored in chat | **Shape** brief + exploration → **Create** / **Follow-up** (skip Explore) → stop |
| Load / discuss `DEC-n` | Load decision only |
| Recency / pick among recent (no topic) | `list_decisions` → ask unless one clear hit → Load if they need the brief |
| Why / what decisions about a topic | `search_decisions` → show ranked hits → **ask** unless one clear hit → `get_decision` before admit/commit |
| Commit existing `DEC-n` | Load decision (Pins) → stop if locked / not approved / in-flight / no recommendation → **Commit** → stop |

If a pick exists and they want it recorded, include `exploration`.

If the original ask includes more than one stage (e.g. explore and admit), do them in order. After Explore, show the draft; continue to **Follow-up** only when admit was in that ask; continue to **Commit** only when commit was in that ask.

## Auth

MCP server `decrec` must be configured in your MCP client. The server handles authentication via bearer tokens automatically.

## Shape

Map the thread (and the open workspace, when the decision is about this project) into admit-ready fields. One Shape → one decision (one admit).

If several distinct decision candidates appear in the thread: use the one they named; otherwise list them briefly (title-scale) and ask which to shape — don’t merge forks into one brief, and don’t Shape/admit more than one unless they ask.

1. Map the brief:
  - **Title:** prefer ≤8 words when you mint one; if they gave a title, use it verbatim (max 200 chars).
  - **Fields** in this order: `problem`, `evidenceSummary`, `assumptions`, `constraints`, `desiredOutcome`, `risks`. Pain/outcome is not a pick — pick lives in synthesis. Thread “use X” → reframe problem/outcome; leave X for the pick.
  - **Empty fields:** Tools require every field non-empty. Where the thread has nothing, use `"unspecified"` (do not invent). Prefer a single honest unknown in `assumptions` over stamping every thin field.
  - **Gap round** if too thin to defend — especially missing `problem` or `desiredOutcome`: ask ≤3 bullets, **stop and wait**; on reply continue. Skip gap round when this Shape is the brief source for **Explore** (Densify covers thin intake).
2. If admitting a pick: synthesis (see **Tool Parameters**); include all three `roles` only if role essays already exist in thread — otherwise omit `roles`. Never partial `roles`. When skipping Explore: map the named pick + alternatives into `preferredOption` / `optionsConsidered` (prefer 2–3, max 4; option `summary`/`tradeoffs` from the thread). Remaining required strings: `confidence` = `medium` if they stated the pick firmly, else `low`; `tradeoffs` / `unresolvedQuestions` / `disagreementHighlights` from the thread, else `"unspecified"` — do not invent disagreements.
3. Do not call tools unless this Shape is for admit. If admitting: **Create** / **Follow-up**. If **Explore** is next or already in progress, continue **Explore** with the mapped brief — do not end the skill.

## Explore

Run the council in this chat with [explore-prompts.md](explore-prompts.md).

1. Brief source: from scratch → **Shape** from the thread (no gap round); existing `DEC-n` → Load decision (call `get_decision` with `decRef`). **Stop** if `isLocked` / `!isApproved` / `activeExploration`. Else tip as-is (no reshape, no Densify — skip step 2).
2. **Densify** (from-scratch only): before role passes, assess intake gaps that would materially change options or confidence (including a missing `problem` or `desiredOutcome`; e.g. who it's for, hard limits, success criteria). This fills evidence / assumptions / constraints / risks so roles start fuller — it is **not** later `openQuestions` / `unresolvedQuestions`. If gaps remain: ask **once** (prefer ≤5; skip trivia), **stop and wait**; do not confirm the brief or run roles in the same turn. On reply, fold answers into the brief; skipped / “unknown” / “proceed anyway” → `"unspecified"`. If already dense, skip this ask. Then show title + brief, wait for **yes**, then roles. If still blocked on naming any option, proceed and leave it in synthesis open questions. Otherwise do not interview beyond that.
3. Three role passes — same title and brief each; clean context per pass (parallel isolated contexts when available, else sequential). See `explore-prompts.md`. Isolated contexts do not have this skill: send the explore-prompts.md role prompt (including Shared grounding) + title/brief + cited files. They may read the workspace; they must not edit it.
4. Synthesis pass on verbatim role outputs (`explore-prompts.md`).
5. Show draft; **stop** unless also asked to admit. When admitting after Explore, send `synthesis` + all three `roles` (never partial).

**Quality:** Prefer 2–3 options (max 4). `preferredOption` must match one `optionsConsidered[].title`; options must be mutually distinct paths (not restatements of the preferred option). After role passes: at least one material cross-role disagreement in `disagreementHighlights`. When `roles` omitted, do not invent disagreement.

## Create

title + brief (+ `exploration`?) → call `create_decision` tool → report → stop.

Writes an **approved** tip. Stage: brief-only → `brief`; +`exploration` → `exploration`. Keep `displayId` for a later **Follow-up** or **Commit**; always load decision for pins. The response `url` is the DecRec UI for this decision.

On tool error, branch on `code` (**Errors**).

## Follow-up

Fail-fast: Load decision (call `get_decision` with `decRef`) → if this chat already has a `briefVersion` for this DEC and it differs, **stop** (Pins) → stop if `isLocked` / `!isApproved` / `activeExploration` → pins from response → call `admit_exploration` tool with `decRef` + pins + exploration → report → stop.

Pins required: `briefVersion` + `expectedDecisionVersion`. Loaded `recommendation` is display-only (extra keys; not an `ExplorationObject`) — do not send it. A Follow-up always writes a new run and becomes the tip recommendation.

On tool error, branch on `code` (**Errors**).

## Commit

Fail-fast: Load decision (call `get_decision` with `decRef`) → if this chat already has a `briefVersion` for this DEC and it differs, **stop** (Pins) → stop if `isLocked` / `!isApproved` / `activeExploration` / no `recommendation` → pins + `explorationRunId` from response → call `commit_decision` tool with `decRef` + pins + `explorationRunId` → report → stop.

On tool error, branch on `code` (**Errors**).

## Errors

Tool failures return `{ "code": "pin_mismatch", "error": "…" }`. Branch on **`code`**, not `error` text. MCP auth that never reaches a tool (server not configured) is `unauthorized`. Schema/protocol errors with no `code` (empty strings, extra keys, partial `roles`) are `invalid_body` — fix; do not Update.

| `code` | Do |
|---|---|
| `unauthorized` | **Stop.** Configure MCP server `decrec`. Never echo tokens. |
| `not_owner` | **Stop.** Only the owner can Follow-up or Commit. |
| `not_found` | **Stop.** Ask for a real `DEC-n` or UUID. |
| `invalid_body` | Fix parameters (empty strings, extra keys, limits, partial `roles`). Do not Update. |
| `locked` | **Stop.** Committed, declined, or archived — no in-place reopen. |
| `not_approved` | **Stop.** Approve or revise at the decision `url`. |
| `in_flight` | Wait, or cancel at the `url`. Do not retry until clear. |
| `pin_mismatch` | Load decision; re-apply fail-fast. `briefVersion` moved vs this chat’s held pin → **stop**, show the six brief-field changes, ask (do not send the old exploration). Only `expectedDecisionVersion` changed → take the new pin, confirm, retry **once**. |
| `display_id_race` | Retry Create with the same parameters **once**. If it fails again, **stop**. |
| `exploration_run_mismatch` | Load decision. If `recommendation.explorationRunId` changed or is missing → **stop**; show the new pick; don’t seal a rec they didn’t confirm. |
| `internal` | Unexpected — **Update**. |

Unknown `code` → **Update**.

## Tool Parameters

Parameters for MCP tool calls. The JSON structure below shows the data sent to each tool.

**`create_decision` — brief only**:

```json
{
  "title": "Short concrete title",
  "brief": {
    "problem": "…",
    "evidenceSummary": "…",
    "assumptions": "…",
    "constraints": "…",
    "desiredOutcome": "…",
    "risks": "…"
  }
}
```

**`create_decision` — brief + synthesis** (`roles` omitted):

```json
{
  "title": "…",
  "brief": { … },
  "exploration": {
    "synthesis": {
      "preferredOption": "…",
      "confidence": "…",
      "tradeoffs": "…",
      "optionsConsidered": [
        { "title": "…", "summary": "…", "tradeoffs": "…" },
        { "title": "…", "summary": "…", "tradeoffs": "…" }
      ],
      "unresolvedQuestions": "…",
      "disagreementHighlights": "…"
    }
  }
}
```

**`create_decision` — brief + synthesis + roles** (all three roles — never partial):

```json
{
  "title": "…",
  "brief": { … },
  "exploration": {
    "synthesis": { … },
    "roles": {
      "customer_advocate": {
        "evidence": "…",
        "inference": "…",
        "assumptions": "…",
        "recommendations": "…",
        "openQuestions": "…"
      },
      "product_strategist": { … },
      "technical_reviewer": { … }
    }
  }
}
```

**`admit_exploration`** (no `title`/`brief`; add `decRef`; pins required; `exploration` required):

```json
{
  "decRef": "DEC-12",
  "briefVersion": 2,
  "expectedDecisionVersion": 3,
  "exploration": { … }
}
```

**`commit_decision`** (no `title`/`brief`/`exploration`; add `decRef`; pins required):

```json
{
  "decRef": "DEC-12",
  "briefVersion": 2,
  "expectedDecisionVersion": 3,
  "explorationRunId": "<uuid>"
}
```

For `get_decision`, call the tool with just `decRef`:

```json
{
  "decRef": "DEC-12"
}
```

**`list_decisions`** (all args optional; active only, `DEC-n` DESC — recency browse, not topic search):

```json
{
  "limit": 20,
  "cursor": "DEC-12"
}
```

Omit `cursor` on the first page. If `nextCursor` is set, pass it as `cursor` for the next page. Then `get_decision` for brief / pins.

**`search_decisions`** (topic search; read-only; no confirm):

```json
{
  "query": "What decisions have been made about pricing?"
}
```

Show ranked hits (`displayId` + title + stage + snippet). If `matchQuality` is `"weak"`, say nothing ranked well. Then `get_decision` for pins before admit/commit.

## Report after tool call success

Emit as normal chat lines — **not** inside a fenced code block:

"""
**{Create|Follow-up|Commit}:** [{displayId}]({url}) · stage={stage}  
**Pins:** briefVersion={n} expectedDecisionVersion={n}  
**explorationRunId:** {id or —}  
**Next (UI only):** revise / decline / evidence / archive  
**Next (this skill):** commit
"""

When `stage=committed`, omit revise / decline / evidence (locked) and omit commit. Archive can stay. Do not chain commit unless they asked.

## Update

This copy's version is `metadata.version` in the frontmatter. Canonical package:

```text
https://raw.githubusercontent.com/decrec-io/decrec-skill/main/
```

Do not fetch that unless they asked to update this skill, or they said **yes** after **Tool errors**.

**Tool errors:** Error whose `code` is `internal`, missing, or not in **Errors**. Not: any listed `code`, empty strings, partial `roles`. On unexpected errors: show the error, say this skill may be stale, ask to update; wait for **yes**. Then continue below.

1. Fetch `{canonical}SKILL.md`; read published `metadata.version`.
2. Same as this copy → already current; stop.
3. Different → show this vs published, then replace every file in this skill directory (`SKILL.md`, `explore-prompts.md`) from the canonical folder. Do not merge. Do not touch `.env.decrec`, MCP configuration, or `*.bk` archives (HTTP skill snapshots, not live).

If fetch fails: stop; they replace the folder from wherever they installed this skill.

## References

- [explore-prompts.md](explore-prompts.md) — role + synthesis prompts
- MCP server `decrec` — provides `get_decision`, `list_decisions`, `search_decisions`, `create_decision`, `admit_exploration`, `commit_decision` tools