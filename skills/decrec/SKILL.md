---
name: decrec
description: "Records and retrieves DecRec decisions via MCP tools (list, search, create, update, follow-up, commit, decline, or archive). Use when recording, admitting, updating, or loading a DecRec decision or DEC-n; listing or searching org decisions; cataloging decision-worthy forks from a pasted conversation, Slack thread, or meeting transcript; exploring a DecRec brief; committing or declining a DecRec recommendation; archiving a decision; before a consequential choice that constrains later work or plausibly revisits an existing direction (search first—not before ordinary coding); when this chat settles such a choice—offer to record, do not auto-save trivia; or when asked to update this skill."
metadata:
  version: "2026.8.27"
---

# Record a Decision into DecRec

## Hard rules

- **Pause:** don’t advance past what they asked (no catalog → shape → update → explore → admit → commit/decline/archive chain unless asked).
- **Recall:** conservative — **Decision-worthy** only. Never before ordinary coding. Don’t block the user’s work if search isn’t available.
- **Capture:** one offer for a live **Decision-worthy** fork (see **Capture trigger**); never auto-Create; never interrupt trivia. Pasted dumps → **Catalog**.
- **Finality:** Thread “we decided / let’s go with / settled / ship it” is a preferred path, not a DecRec commit. Catalog it; do not skip as already done unless they say it is recorded elsewhere and should be skipped.
- **Auth:** MCP server `decrec` must be configured. Never echo tokens.
- **Commit / Decline:** Load decision first (call `get_decision`); take `explorationRunId` from `recommendation` (don’t invent; don’t send the whole object). **Commit** in this skill is DecRec only — not `git commit`. Decline is “we decided not to” (no Commitment row).
- **Explore vs existing pick:** If they asked to explore (from scratch, `DEC-n`, or re-explore), run **Explore** even if a pick or loaded `recommendation` already exists. Skip Explore only when **admitting** without an explore ask, and a preferred option plus ≥2 distinct alternatives already exist — map those into `exploration.synthesis`. Never invent a pick just to unlock admit. Never run Explore just to fill `roles` (omit `roles` unless essays already exist).
- **Workspace:** the open project is context for Shape / Explore — read and cite it. Do not implement the pick, open a PR, or edit the repo unless they asked for code.
- **No invention:** don’t invent stack, vendors, tenancy, or factual business metrics (counts, rates, revenue) unless the brief, chat attachments, or workspace state them. Proposing success metrics (what to measure) is allowed.
- **UI only:** title, evidence, cancel in-flight explore — at the decision `url` from response/Create (no tools for these).
- **Update this skill:** only if they ask. On tool errors, ask first and wait for **yes** — don’t fetch or replace until then. Never check on a normal run.

## Enter from anywhere

Match the **most specific** row. Don’t advance to a later stage (shape/update/explore/admit/commit/decline/archive) unless they asked for it. Clarifying a thin brief during **Shape** / **Explore** is part of that stage — not a one-shot requirement.

If this chat is **Decision-worthy** and they didn’t name a `DEC-n`, run **Decision recall** first, then the matched row. Ordinary coding, bugfixes, and implementing an already-chosen approach do **not** qualify.

**Tip** = the current brief version (`briefVersion` + `brief` from response). Update, Explore-existing, Follow-up, Commit, and Decline target that tip.

**Pins** = `briefVersion` + `expectedDecisionVersion` from response. Commit and Decline also need `explorationRunId` from response `recommendation`. Archive needs `expectedDecisionVersion` only. Always load decision for pins. If this chat already has a `briefVersion` for that DEC (create, update, or a prior GET), a later load is a **reload**: compare **only** that number. Unchanged → do not inspect brief text. Changed → **stop**, show the six brief-field changes, ask. Do not admit, commit, decline, or update-from-stale onto the new tip.

**Find without `DEC-n`:** Topic in the ask (“why / what decisions about…”) → **`search_decisions`** (not list). Recency / what’s open / pick among recent with no topic → **`list_decisions`**. One clear hit → use it. Several → show `displayId` + title (+ stage/snippet for search) and **ask**. Don’t invent. Search hits are **not** pins — still `get_decision` before update/admit/commit/decline/archive. `matchQuality: "weak"` → say nothing ranked well; still show hits; do not claim the org has no decisions. Do not page `list_decisions` hoping titles match a theme. If `nextCursor` is set on list and they need more, pass it as `cursor`.

| Intent | Path |
|---|---|
| Pasted thread / transcript / meeting notes (extract / what to record) | **Catalog** → stop. **Shape** only picked rows. Admit / Explore only if that was in the ask |
| Explore from scratch | **Shape** brief → **Explore** (Densify, then confirm brief) → stop |
| Explore existing `DEC-n` | Load decision → stop if locked / not approved / in-flight → **Explore** on tip (as-is), even if `recommendation` is set → stop |
| Admit framing only | **Shape** brief → **Create** brief-only → stop |
| Admit new + pick | **Shape** brief + exploration → **Create** with `exploration` → stop |
| Admit onto existing `DEC-n` (same brief) | Load decision (Pins) → stop if locked / not approved / in-flight → **Follow-up** (Shape exploration if not in hand) → stop |
| Update existing `DEC-n` (brief ± pick) | Load decision (Pins) → stop if locked / in-flight → **Shape** from tip + requested changes → **Update** (include `exploration` if admitting a pick) → stop |
| Already explored in chat | **Shape** brief + exploration → **Create** / **Update** / **Follow-up** (skip Explore) → stop |
| Load / discuss `DEC-n` | Load decision only |
| Recency / pick among recent (no topic) | `list_decisions` → ask unless one clear hit → Load if they need the brief |
| Why / what decisions about a topic | `search_decisions` → show ranked hits → **ask** unless one clear hit → `get_decision` before update/admit/commit/decline/archive |
| Commit existing `DEC-n` | Load decision (Pins) → stop if locked / not approved / in-flight / no recommendation → **Commit** → stop |
| Decline existing `DEC-n` | Load decision (Pins) → stop if locked / not approved / in-flight / no recommendation → **Decline** → stop |
| Archive existing `DEC-n` | Load decision (Pins) → stop if `status=archived` → **Archive** → stop |
| About to reopen a **Decision-worthy** direction (no `DEC-n`) | **Decision recall** → `search_decisions` → then reason |
| Which account / org this MCP session is | `whoami` → report email + org → stop. Skip if initialize instructions already named it and they did not ask |
| No DecRec ask; this chat just settled a **Decision-worthy** fork | **Capture trigger** → offer once → stop unless they accept |

If a pick exists and they want it recorded, include `exploration`.

If the original ask includes more than one stage (e.g. catalog and admit, or explore and admit), do them in order. After Catalog, **Shape** only picked rows; continue to **Create** / **Explore** only when that was in the ask. After **Update**, show the new tip; continue to **Explore** / **Follow-up** only when that was in the ask (Follow-up only if the brief did not change — otherwise the pick belonged on **Update**). After Explore, show the draft; continue to **Create** / **Update** / **Follow-up** only when admit was in that ask; continue to **Commit** / **Decline** / **Archive** only when that action was in that ask.

## Decision-worthy

A fork is decision-worthy when **all three** are true:

- **Alternatives:** ≥2 credible paths were on the table (including do-nothing / defer)
- **Consequence:** the choice meaningfully constrains later work (architecture, product, process, a constraint)
- **Future relevance:** another human or agent could reasonably ask *why did we do this?*

Judge the *direction*, not the action. A sent email cannot be unsent — that alone is not Consequence.

**Yes:** Postgres vs Dynamo as system of record; public API contract vs internal-only; usage-based vs seat pricing.

**No:** tabs vs spaces; local names; Tailwind spacing; temporary debug logs; which of two independent tickets to implement today; easily reversible implementation details; copy/outreach/sequence trials; temporary workflows.

**Recall**, **Capture trigger**, and **Catalog** share this bar. If uncertain: Recall does not search; Capture does not offer; Catalog still lists the row.

## Decision recall

Search before reasoning — only when this chat is **Decision-worthy** and they didn’t name a `DEC-n`. Before materially reconsidering a consequential product, technical, operational, or company direction, search DecRec for relevant prior decisions when doing so could prevent reopening settled reasoning or contradicting an existing commitment.

Do **not** search before ordinary implementation, bugfixes, or refactors inside an already-chosen approach.

`search_decisions` (not list). One clear hit → `get_decision`. Several → show `displayId` + title + stage; load the one that would constrain this ask. `matchQuality: "weak"` → say nothing ranked well; still show hits; proceed with reasoning. Do not claim the org has no decisions.

Cite prior decisions before proposing a conflicting path. A `committed` hit is a constraint unless they explicitly want to reopen. Do not auto-admit, update, explore, commit, or decline from recall. If they also asked to Shape / Explore / admit, search first, then continue that path.

**Skip (don’t search):** this chat already searched or loaded this topic; they named a `DEC-n` (Load that). MCP missing / `unauthorized` → continue their work; don’t Stop the whole turn.

## Capture trigger

When this chat produces a **Decision-worthy** choice, briefly offer to record it in DecRec. Do not record automatically.

Finish the work they asked for first. Then offer only if it is **Decision-worthy**. If uncertain, do not offer — they can still ask to record.

Pasted dump / transcript → **Catalog**, not this. Already in Catalog / Shape / Explore / admit / update / commit / decline / archive for that fork → don’t re-offer. They declined or ignored this fork in this chat → don’t nag.

Offer in **one sentence** plus a working title (≤8 words). If they accept: **Admit new + pick** when a preferred path is in hand, else **Admit framing only**. Do not Explore unless they ask. If they decline or keep working, drop it.

## Auth

MCP server `decrec` must be configured in your MCP client. The server handles authentication via bearer tokens automatically. Initialize instructions already name this session (`Connected as {email} in {orgName}`). Call `whoami` only if they ask which account, or two DecRec connections might disagree. Do not call it before list, search, create, or load.

## Catalog

For a pasted Slack thread, meeting notes, email, or transcript: find **Decision-worthy** forks. Do not call tools. Do not Shape, Explore, or admit until they pick — and only as far as they asked.

Require the conversation text. If missing, ask once.

Include forks that are **Decision-worthy** and have a decision moment: open question, disagreement, blocked work, **or** apparent agreement / “we decided X” — even if the group leaned or “decided.”

**Always include (high priority):** moments that sound decided — capture them so the preferred path is not lost. Do not drop them as “already done.”

**Exclude:** status/FYI with no fork, brainstorms with no pressure to choose, tasks with no strategic fork, vague aspirations, meta-process unless the decision *is* the process. Do **not** exclude because speakers used decided/settled language. If unsure a row is **Decision-worthy**, still show it — they pick.

If nothing qualifies, say so. Do not invent decisions.

Output a ranked table (max 7; merge duplicates; split only if independently decidable). Rank by: apparent agreement that still needs recording, then urgency × impact × how blocked the team is:

| # | Working title (≤8 words) | Why decide / record now | Fork (options seen) | Thread stance (open / leaned / said-decided) | Material enough? |

Then **stop** unless they already named `#N` / `all top K`, or **skip catalog** (they named one decision, or the thread has a single obvious fork) — then **Shape** that item.

On reply, **Shape** only the chosen numbers (one decision each; don’t merge). If they say proceed without picking, Shape the top item only. Admit or Explore a row only when that was in the ask.

## Shape

Map the thread (and the open workspace, when the decision is about this project) into admit-ready fields. One Shape → one decision (one admit). After **Catalog**, map only the selected row(s).

If several distinct decision candidates appear and this is a dumped conversation, **Catalog** instead of Shape. If they already named one fork, or Catalog was skipped, Shape that one — don’t merge forks into one brief, and don’t Shape/admit more than one unless they ask.

1. Map the brief:
  - **Title:** prefer ≤8 words when you mint one; if they gave a title, use it verbatim (max 200 chars).
  - **Fields** in this order: `problem`, `evidenceSummary`, `assumptions`, `constraints`, `desiredOutcome`, `risks`. Pain/outcome is not a pick — pick lives in synthesis. Thread “use X” → reframe problem/outcome; leave X for the pick.
  - **Empty fields:** Tools require every field non-empty. Where the thread has nothing, use `"unspecified"` (do not invent). Prefer a single honest unknown in `assumptions` over stamping every thin field.
  - **Gap round** if too thin to defend — especially missing `problem` or `desiredOutcome`: ask ≤3 bullets, **stop and wait**; on reply continue. Skip gap round when this Shape is the brief source for **Explore** (Densify covers thin intake).
2. If admitting a pick: synthesis (see **Tool Parameters**); include all three `roles` only if role essays already exist in thread — otherwise omit `roles`. Never partial `roles`. When skipping Explore: map the named pick + alternatives into `preferredOption` / `optionsConsidered` (prefer 2–3, max 4; option `summary`/`tradeoffs` from the thread). Remaining required strings: `confidence` = `medium` if they stated the pick firmly, else `low`; `tradeoffs` / `unresolvedQuestions` / `disagreementHighlights` from the thread, else `"unspecified"` — do not invent disagreements.
3. Do not call tools unless this Shape is for admit or update. If admitting a new DEC: **Create**. If updating framing on an existing DEC: **Update** (include `exploration` if admitting a pick). If same brief, new pick: **Follow-up**. If **Explore** is next or already in progress, continue **Explore** with the mapped brief — do not end the skill.

## Explore

Run the council in this chat with [explore-prompts.md](explore-prompts.md).

1. Brief source: from scratch → **Shape** from the thread (no gap round); existing `DEC-n` → Load decision (call `get_decision` with `decRef`). **Stop** if `isLocked` / `!isApproved` / `activeExploration`. Else tip as-is (no reshape, no Densify — skip step 2).
2. **Densify** (from-scratch only): before role passes, assess intake gaps that would materially change options or confidence (including a missing `problem` or `desiredOutcome`; e.g. who it's for, hard limits, success criteria). This fills evidence / assumptions / constraints / risks so roles start fuller — it is **not** later `openQuestions` / `unresolvedQuestions`. If gaps remain: ask **once** (prefer ≤5; skip trivia), **stop and wait**; do not confirm the brief or run roles in the same turn. On reply, fold answers into the brief; skipped / “unknown” / “proceed anyway” → `"unspecified"`. If already dense, skip this ask. Then show title + brief, wait for **yes**, then roles. If still blocked on naming any option, proceed and leave it in synthesis open questions. Otherwise do not interview beyond that.
3. Three role passes — same title and brief each; clean context per pass (parallel isolated contexts when available, else sequential). See `explore-prompts.md`. Isolated contexts do not have this skill: send the explore-prompts.md role prompt (including Shared grounding) + title/brief + cited files. They may read the workspace; they must not edit it.
4. Synthesis pass on verbatim role outputs (`explore-prompts.md`).
5. Show draft; **stop** unless also asked to admit. When admitting after Explore: new DEC → **Create** with `exploration`; existing DEC, brief changed → **Update** with `exploration`; existing DEC, same brief → **Follow-up**. Send `synthesis` + all three `roles` (never partial).

**Quality:** Prefer 2–3 options (max 4). `preferredOption` must match one `optionsConsidered[].title`; options must be mutually distinct paths (not restatements of the preferred option). After role passes: at least one material cross-role disagreement in `disagreementHighlights`. When `roles` omitted, do not invent disagreement.

## Create

title + brief (+ `exploration`?) → call `create_decision` tool → report → stop.

Writes an **approved** tip. Stage: brief-only → `brief`; +`exploration` → `exploration`. Keep `displayId` for a later **Update**, **Follow-up**, **Commit**, **Decline**, or **Archive**; always load decision for pins. The response `url` is the DecRec UI for this decision.

On tool error, branch on `code` (**Errors**).

## Update

Fail-fast: Load decision (call `get_decision` with `decRef`) → if this chat already has a `briefVersion` for this DEC and it differs, **stop** (Pins) → stop if `isLocked` / `activeExploration` → **Shape** the new brief from the tip plus the requested changes (send all six fields) → pins from response → call `update_decision` tool with `decRef` + pins + `brief` (+ `exploration` if admitting a pick) → report → stop.

Writes a new tip. +`exploration` → stage `exploration` (owner only). Prior tip recommendation no longer applies. Same brief, new pick → **Follow-up**, not this path. Brief-only: any org member.

On tool error, branch on `code` (**Errors**).

## Follow-up

Fail-fast: Load decision (call `get_decision` with `decRef`) → if this chat already has a `briefVersion` for this DEC and it differs, **stop** (Pins) → stop if `isLocked` / `!isApproved` / `activeExploration` → pins from response → call `admit_exploration` tool with `decRef` + pins + exploration → report → stop.

Pins required: `briefVersion` + `expectedDecisionVersion`. Loaded `recommendation` is display-only (extra keys; not an `ExplorationObject`) — do not send it. A Follow-up always writes a new run on the **current** approved tip and becomes the tip recommendation. Brief changed → **Update decision**, not Follow-up.

On tool error, branch on `code` (**Errors**).

## Commit

Fail-fast: Load decision (call `get_decision` with `decRef`) → if this chat already has a `briefVersion` for this DEC and it differs, **stop** (Pins) → stop if `isLocked` / `!isApproved` / `activeExploration` / no `recommendation` → pins + `explorationRunId` from response → call `commit_decision` tool with `decRef` + pins + `explorationRunId` → report → stop.

On tool error, branch on `code` (**Errors**).

## Decline

Fail-fast: Load decision (call `get_decision` with `decRef`) → if this chat already has a `briefVersion` for this DEC and it differs, **stop** (Pins) → stop if `isLocked` / `!isApproved` / `activeExploration` / no `recommendation` → pins + `explorationRunId` from response → call `decline_decision` tool with `decRef` + pins + `explorationRunId` → report → stop.

On tool error, branch on `code` (**Errors**).

## Archive

Fail-fast: Load decision (call `get_decision` with `decRef`) → if this chat already has a `briefVersion` for this DEC and it differs, **stop** (Pins) → stop if `status=archived` → `expectedDecisionVersion` from response → call `archive_decision` tool with `decRef` + `expectedDecisionVersion` → report → stop.

Committed or declined may still be archived (`isLocked` is not a stop). In-flight exploration is cancelled by Archive. Do not send `briefVersion` or `explorationRunId`.

On tool error, branch on `code` (**Errors**).

## Errors

Tool failures return `{ "code": "pin_mismatch", "error": "…" }`. Branch on **`code`**, not `error` text. MCP auth that never reaches a tool (server not configured) is `unauthorized`. Schema/protocol errors with no `code` (empty strings, extra keys, partial `roles`) are `invalid_body` — fix; do not Update this skill.

| `code` | Do |
|---|---|
| `unauthorized` | **Stop.** Configure MCP server `decrec`. Never echo tokens. |
| `not_owner` | **Stop.** Only the owner can Follow-up, Commit, Decline, Archive, or Update decision with `exploration`. Brief-only Update decision is any org member. |
| `not_found` | **Stop.** Ask for a real `DEC-n` or UUID. |
| `invalid_body` | Fix parameters (empty strings, extra keys, limits, partial `roles`). Do not Update this skill. |
| `locked` | **Stop.** Committed, declined, or archived — no in-place reopen. Archive of an already-archived decision is `locked`. |
| `not_approved` | **Stop.** Approve at the decision `url`. |
| `in_flight` | Wait, or cancel at the `url`. Do not retry until clear. Archive cancels in-flight — only if they asked to archive. |
| `pin_mismatch` | Load decision; re-apply fail-fast. `briefVersion` moved vs this chat’s held pin → **stop**, show the six brief-field changes, ask (do not send the old brief or exploration). Only `expectedDecisionVersion` changed → take the new pin, confirm, retry **once**. |
| `display_id_race` | Retry Create with the same parameters **once**. If it fails again, **stop**. |
| `exploration_run_mismatch` | Load decision. If `recommendation.explorationRunId` changed or is missing → **stop**; show the new pick; don’t commit or decline a rec they didn’t confirm. |
| `internal` | Unexpected — **Update this skill**. |

Unknown `code` → **Update this skill**.

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

**`update_decision` — brief only** (add `decRef`; pins required):

```json
{
  "decRef": "DEC-12",
  "briefVersion": 2,
  "expectedDecisionVersion": 3,
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

**`update_decision` — brief + exploration** (`exploration` same shape as Create; owner only):

```json
{
  "decRef": "DEC-12",
  "briefVersion": 2,
  "expectedDecisionVersion": 3,
  "brief": { … },
  "exploration": { … }
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

**`decline_decision`** (same pins as commit):

```json
{
  "decRef": "DEC-12",
  "briefVersion": 2,
  "expectedDecisionVersion": 3,
  "explorationRunId": "<uuid>"
}
```

**`archive_decision`** (`expectedDecisionVersion` only; no `briefVersion` / `explorationRunId`):

```json
{
  "decRef": "DEC-12",
  "expectedDecisionVersion": 3
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

Show ranked hits (`displayId` + title + stage + snippet). If `matchQuality` is `"weak"`, say nothing ranked well. Then `get_decision` for pins before update/admit/commit/decline/archive.

**`whoami`** (no args; only if they asked which account, or two DecRec connections might disagree):

```json
{}
```

Report `email`, `orgName`, and `auth` (`pat` | `oauth` | `session`). Do not echo tokens.

## Report after tool call success

Emit as normal chat lines — **not** inside a fenced code block:

"""
**{Create|Update|Follow-up|Commit|Decline|Archive}:** [{displayId}]({url}) · stage={stage} · status={status}  
**Pins:** briefVersion={n} expectedDecisionVersion={n}  
**explorationRunId:** {id or —}  
**Next (UI only):** title / evidence  
**Next (this skill):** update / commit / decline / archive
"""

When `stage=committed` or `stage=declined`, omit title / evidence (locked) and omit update / commit / decline. Archive can stay unless `status=archived`. When `status=archived`, omit all next actions. Do not chain commit, decline, or archive unless they asked.

## Update this skill

This copy's version is `metadata.version` in the frontmatter. Canonical package:

```text
https://raw.githubusercontent.com/decrec-io/decrec-plugin/main/skills/decrec/
```

Do not fetch that unless they asked to update this skill, or they said **yes** after **Tool errors**.

**Tool errors:** Error whose `code` is `internal`, missing, or not in **Errors**. Not: any listed `code`, empty strings, partial `roles`. On unexpected errors: show the error, say this skill may be stale, ask to update; wait for **yes**. Then continue below.

1. Fetch `{canonical}SKILL.md`; read published `metadata.version`.
2. Same as this copy → already current; stop.
3. Different → show this vs published, then replace every file in this skill directory (`SKILL.md`, `explore-prompts.md`) from the canonical folder. Do not merge. Do not touch MCP configuration.

If fetch fails: stop; they replace the folder from wherever they installed this skill.

## References

- [explore-prompts.md](explore-prompts.md) — role + synthesis prompts
- MCP server `decrec` — provides `get_decision`, `list_decisions`, `search_decisions`, `create_decision`, `update_decision`, `admit_exploration`, `commit_decision`, `decline_decision`, `archive_decision`, `whoami` tools