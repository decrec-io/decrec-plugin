---
name: decrec
description: "Records and retrieves DecRec decisions via MCP tools (list, search, create, update, follow-up, commit, decline, archive, or list/get councils). Use when recording, admitting, updating, or loading a DecRec decision or DEC-n; listing or searching org decisions; cataloging decision-worthy forks from a pasted conversation, Slack thread, or meeting transcript; exploring a DecRec brief; committing or declining a DecRec recommendation; archiving a decision; before a consequential choice that constrains later work or plausibly revisits an existing direction (search first—not before ordinary coding); when this chat settles such a choice—offer to record, do not auto-save trivia."
metadata:
  version: "2026.9.1"
---

## Pick a Route

Don’t add a step they didn’t name.

| Intent | Path |
|---|---|
| Extract / what to record from a pasted thread, transcript, or notes — or a dump with no search / load / commit / decline / archive | **Catalog** → stop. **Shape** only picked rows. Admit / Explore only if that was in the ask. Pasted + search / load / commit / decline / archive → that row, not Catalog |
| Explore existing `DEC-n` | **Load** → stop if locked / not approved / in-flight (approve or cancel at `url`; locked cannot reopen in place) → **Explore** on tip (as-is), even if `recommendation` is set → stop unless admit / Commit / Decline was in the ask. Named Commit / Decline on this draft → admit (Follow-up) first, then that action — don’t Commit the loaded rec |
| Explore from scratch (no `DEC-n`) | **Shape** brief → **Explore** (Densify, then confirm brief) → stop unless admit / Commit / Decline was in the ask. Named Commit / Decline on this draft → admit (Create) first, then that action |
| Draft / frame a brief (they asked to draft / frame / outline; no `DEC-n`; no record / save / create / admit / update verb) | **Shape** → stop. Do not Create |
| Admit framing only (no `DEC-n`) | **Shape** brief → **Create** brief-only → stop |
| Admit new + pick (no `DEC-n`) | **Shape** brief + exploration → **Create** with `exploration` → stop |
| Update, follow-up, or admit a pick onto existing `DEC-n` | **Load** → stop if locked / in-flight → if pick, also stop if not approved (approve at `url`) → asked to change framing: **Shape** from tip + requested changes → **Update** (`exploration` if admitting a pick) → stop; same brief + pick: **Follow-up** (Shape exploration if not in hand) → stop. After **Update**, show the new tip; continue to **Explore** only when that was in the ask |
| Already explored in this chat (admit / record the draft; they did not ask to re-explore) | **Shape** from that draft (include `roles`) → **Create** / **Update** / **Follow-up** (skip **Explore**) → stop unless Commit / Decline was in the ask. Asked to explore / re-explore → that row |
| Rename, attach evidence, approve the brief, or cancel an in-flight run | **Load** → send them to `{url}` → stop. No writes |
| Load / discuss `DEC-n` (named-id fallback: no explore / write / UI / terminal verb) | **Load** only. Asked to draft / frame / outline: **Shape** from the tip in chat → stop. Do not Update |
| Recency / pick among recent (no topic) | `list_decisions` → ask unless one clear hit → **Load** if they need the brief. If `nextCursor` is set and they need more, pass it as `cursor` |
| Why / what decisions about a topic | **Search** |
| Commit existing `DEC-n` | **Load** → stop if locked / not approved / in-flight / no recommendation → **Commit** → stop |
| Decline existing `DEC-n` | **Load** → stop if locked / not approved / in-flight / no recommendation → **Decline** → stop |
| Archive existing `DEC-n` | **Load** → stop if `status=archived` → **Archive** → stop |
| About to choose or check prior decisions (no `DEC-n`, no write or explore ask) | **Recall** → then reason |
| Which account / org this MCP session is | `whoami` → report `email`, `orgName`, and `auth` → stop. Skip if initialize instructions already named it and they did not ask |
| No DecRec ask; this chat just settled a **Decision-worthy** fork | **Capture** → offer once → stop unless they accept |

## Decision-worthy

A fork is decision-worthy when **all three** are true:

- **Alternatives:** ≥2 credible paths were on the table (including do-nothing / defer)
- **Consequence:** the choice meaningfully constrains later work (architecture, product, process, a constraint)
- **Future relevance:** another human or agent could reasonably ask *why did we do this?*

Judge the *direction*, not the action. A sent email cannot be unsent — that alone is not Consequence.

**Yes:** Postgres vs Dynamo as system of record; public API contract vs internal-only; usage-based vs seat pricing.

**No:** tabs vs spaces; local names; Tailwind spacing; temporary debug logs; which of two independent tickets to implement today; easily reversible implementation details; copy/outreach/sequence trials; temporary workflows.

**Recall**, **Capture**, and **Catalog** share this bar. If uncertain: Recall does not Search; Capture does not offer; Catalog still lists the row.

## Search

Topic in the ask (“why / what decisions about…”) and no `DEC-n`. Recency / what’s open with no topic → `list_decisions` (not this).

`search_decisions` (not list). Do not page `list_decisions` hoping titles match a theme.

One clear hit → use it. Several → show `displayId` + title + stage/snippet and **ask**. Don’t invent.

Hits are **active** only (archived omitted). If they asked about archived records, say so — do not treat empty or `weak` as “the org has none.” `matchQuality: "weak"` → say nothing ranked well; still show hits; do not claim the org has no decisions.

Hits are not a load and not an `exploration` payload (`brief` / `recommendation` on a hit are display-only). **Load** before update / admit / commit / decline / archive.

## Recall

**Search** before reasoning — not a second lookup. Only when the matched row is **Recall**: this chat is **Decision-worthy**, they didn’t name a `DEC-n`, and they didn’t ask to explore or write. Before materially reconsidering a consequential product, technical, operational, or company direction, Search when doing so could prevent reopening settled reasoning or contradicting an existing commitment.

Do **not** Search before ordinary implementation, bugfixes, or refactors inside an already-chosen approach.

Follow-through differs from **Search**: don’t **ask**. One clear hit → **Load**. Several → **Load** the one that would constrain this ask. `matchQuality: "weak"` → proceed with reasoning. Cite prior decisions before proposing a conflicting path. A `committed` hit is a constraint unless they explicitly want to reopen. Do not auto-admit, update, explore, commit, or decline from Recall.

**Skip (don’t Search):** any other matched row; this chat already searched or loaded this topic; they named a `DEC-n` (**Load** that). MCP missing / `unauthorized` → continue their work; don’t Stop the whole turn.

## Capture

When this chat produces a **Decision-worthy** choice, briefly offer to record it in DecRec. Do not record automatically. Thread “we decided / let’s go with / settled / ship it” is a preferred path to this offer, not a DecRec **Commit**. Do not skip as already done unless they say it is recorded elsewhere and should be skipped.

Finish the work they asked for first. Then offer only if it is **Decision-worthy**. If uncertain, do not offer — they can still ask to record.

Pasted dump / transcript → **Catalog**, not this. Already in Catalog / Shape / Explore / admit / update / commit / decline / archive for that fork → don’t re-offer. They declined or ignored this fork in this chat → don’t nag.

Offer in **one sentence** plus a working title (≤8 words). If they accept: **Admit new + pick** when a preferred path is in hand, else **Admit framing only**. Do not Explore unless they ask. If they decline or keep working, drop it.

## Catalog

For a pasted Slack thread, meeting notes, email, or transcript: find **Decision-worthy** forks. If they pasted but asked to search, load, commit, decline, or archive, that row wins — not this. Explore / draft / admit stay on Catalog; continue only as far as they asked. Do not call tools. Do not Shape, Explore, or admit until they pick — and only as far as they asked.

Require the conversation text. If missing, ask once.

Include forks that are **Decision-worthy** and have a decision moment: open question, disagreement, blocked work, **or** apparent agreement / “we decided X” — even if the group leaned or “decided.”

**Always include (high priority):** moments that sound decided — capture them so the preferred path is not lost. Do not drop them as “already done” unless they say it is recorded elsewhere and should be skipped. Thread settled language is not a DecRec **Commit**.

**Exclude:** status/FYI with no fork, brainstorms with no pressure to choose, tasks with no strategic fork, vague aspirations, meta-process unless the decision *is* the process. Do **not** exclude because speakers used decided/settled language. If unsure a row is **Decision-worthy**, still show it — they pick.

If nothing qualifies, say so. Do not invent decisions.

Output a ranked table (max 7; merge duplicates; split only if independently decidable). Rank by: apparent agreement that still needs recording, then urgency × impact × how blocked the team is:

| # | Working title (≤8 words) | Why decide / record now | Fork (options seen) | Thread stance (open / leaned / said-decided) | Material enough? |
|---|---|---|---|---|---|

Then **stop** unless they already named `#N` / `all top K`, or **skip catalog** (they named one decision, or the thread has a single obvious fork) — then **Shape** that item.

On reply, **Shape** only the chosen numbers (one decision each; don’t merge). If they say proceed without picking, Shape the top item only. Admit or Explore a row only when that was in the ask.

## Shape

Map the thread (and the open workspace, when the decision is about this project) into admit-ready fields. Read and cite the workspace (filename / path); do not edit. Do not invent stack, vendors, tenancy, or factual business metrics unless the thread, attachments, or workspace state them; proposing success metrics (what to measure) is allowed. One Shape → one decision (one admit). After **Catalog**, map only the selected row(s).

If several distinct decision candidates appear and this is a dumped conversation, **Catalog** instead of Shape. If they already named one fork, or Catalog was skipped, Shape that one — don’t merge forks into one brief, and don’t Shape/admit more than one unless they ask.

1. Map the brief:
  - **Title:** prefer ≤8 words when you mint one; if they gave a title, use it verbatim (max 200 chars).
  - **Fields** in this order: `problem`, `evidenceSummary`, `assumptions`, `constraints`, `desiredOutcome`, `risks`. Pain/outcome is not a pick — pick lives in synthesis. Thread “use X” → reframe problem/outcome; leave X for the pick.
  - **Empty fields:** Tools require every field non-empty. Where the thread has nothing, use `"unspecified"` (do not invent). Prefer a single honest unknown in `assumptions` over stamping every thin field.
  - **Gap round** if too thin to defend — especially missing `problem` or `desiredOutcome`: ask ≤3 bullets, **stop and wait**; on reply continue. Skip gap round when this Shape is the brief source for **Explore** (Densify covers thin intake).
2. If admitting a pick: send `exploration: { synthesis, roles? }`. Put `optionsConsidered`, `preferredOption`, `tradeoffs`, `confidence`, `unresolvedQuestions`, and `disagreementHighlights` **inside `synthesis`** — not on `exploration`. Prefer 2–3 `optionsConsidered` (max 4), each `{ title, summary, tradeoffs }` from the thread; mutually distinct paths; `preferredOption` equals one `title`. `confidence` = `medium` if they stated the pick firmly, else `low`. Remaining synthesis strings from the thread, else `"unspecified"` — do not invent disagreements. Include `roles` as an array of `{ id, label, result }` only if those essays already exist in thread — otherwise omit `roles`. Never partial. Skip **Explore** only when admitting without an explore ask and a preferred option plus ≥2 distinct alternatives already exist (including this chat’s Explore draft). Never invent a pick just to unlock admit. Never run **Explore** only to fill `roles`.
3. Do not call tools unless this Shape is for admit or update. Draft / frame only (no write verb) → stop after mapping; do not Create or Update. If admitting a new DEC: **Create**. If updating framing on an existing DEC: **Update** (include `exploration` if admitting a pick). If same brief, new pick: **Follow-up**. If **Explore** is next or already in progress, continue **Explore** with the mapped brief — do not end the skill.

## Explore

Run the council in this chat with [explore-prompts.md](explore-prompts.md). If they asked to explore (from scratch, `DEC-n`, or re-explore), run even if a pick or loaded `recommendation` already exists. Never run Explore only to fill `roles`. When the decision is about this project, read and cite the open workspace. Do not invent stack, vendors, tenancy, or factual business metrics unless the brief, attachments, or workspace state them; proposing success metrics is allowed.

**Council pick (before role passes):**
- One-off roster named in this chat → use that roster; do **not** call `list_councils`; do **not** auto-save to Settings.
- Named saved council, or this chat already used one for this DEC → `get_council` with that name or id; skip list unless they asked to change.
- Else → `list_councils`, pick from the brief using name + role labels; then `get_council` with that `councilId`. None fit → `get_council()` with no args (org default, including focus). Do not infer default from `isDefault` on the list then get — use no-args get.

1. Brief source: from scratch → **Shape** from the thread (no gap round); existing `DEC-n` → **Load**. **Stop** if `isLocked` / `!isApproved` / `activeExploration` — locked cannot reopen in place; unapproved → approve at the `url`; in-flight → wait or cancel at the `url`. Else tip as-is (no reshape, no Densify — skip step 2).
2. **Densify** (from-scratch only): before role passes, assess intake gaps that would materially change options or confidence (including a missing `problem` or `desiredOutcome`; e.g. who it's for, hard limits, success criteria). This fills evidence / assumptions / constraints / risks so roles start fuller — it is **not** later `openQuestions` / `unresolvedQuestions`. If gaps remain: ask **once** (prefer ≤5; skip trivia), **stop and wait**; do not confirm the brief or run roles in the same turn. On reply, fold answers into the brief; skipped / “unknown” / “proceed anyway” → `"unspecified"`. If already dense, skip this ask. Then show title + brief, wait for **yes**, then roles. If still blocked on naming any option, proceed and leave it in synthesis open questions. Otherwise do not interview beyond that.
3. Role passes (2–4 from the chosen council) — same title and brief each; clean context per pass (parallel isolated contexts when available, else sequential). See `explore-prompts.md`. Isolated contexts do not have this skill: send the explore-prompts.md role prompt (including Shared grounding) + title/brief + cited files. Substitute `{Role label}` + `{focus}` from `get_council` / the one-off. They may read the workspace; they must not edit it.
4. Synthesis pass on verbatim role outputs (`explore-prompts.md`).
5. Show draft; **stop** unless also asked to admit, Commit, or Decline. Named Commit / Decline on this draft implies admit first — don’t Commit / Decline the loaded `recommendation`. When admitting after Explore: new DEC → **Create** with `exploration`; existing DEC, brief changed → **Update** with `exploration`; existing DEC, same brief → **Follow-up**. Then Commit / Decline only if that was in the ask. Send `exploration` with `synthesis` plus `roles` as an array of every council role `{ id, label, result }` (never partial). Do not implement the pick unless they asked — exploring or recording is not a request to implement it.

**Quality:** Prefer 2–3 options (max 4). `preferredOption` must match one `optionsConsidered[].title`; options must be mutually distinct paths (not restatements of the preferred option). After role passes: at least one material cross-role disagreement in `disagreementHighlights`. When `roles` omitted, do not invent disagreement.

## Load

Need `DEC-n` or UUID. Search/list hits are not a load — do not send a hit’s `brief` or `recommendation` as `exploration` or pins.

**Tip** = the current brief version (`briefVersion` + `brief` from this response). Update, Explore-existing, Follow-up, Commit, and Decline target that tip.

Call `get_decision` with `decRef`. Use **this response** for fail-fast, pins, `url`, and `explorationRunId`. `recommendation` is display-only (extra keys; not an `exploration` payload) — do not send it. Archive sends `expectedDecisionVersion` only (no `briefVersion`, no run id). Commit / Decline send pins plus `explorationRunId` (not the whole `recommendation`).

**Reload** (before **Update** / **Follow-up** / **Commit** / **Decline** only): if this chat already saw a `briefVersion` for that DEC, compare **only** that number to this response. Unchanged → brief text did not move; do not re-diff it. Changed → **stop**, show the six brief-field changes, ask. **Load / discuss** always reads the brief. **Archive** does not compare `briefVersion`.

If Load is the whole ask: discuss the tip in normal sentences (brief; pick if `recommendation` is set). Asked to draft / frame / outline: **Shape** from the tip in chat — do not Update. Do not **Report** a write.

On tool error, branch on `code` (**Errors**).

## Create

title + brief (+ `exploration`?) → call `create_decision` tool → **Report** → stop.

Writes an **approved** tip. Stage: brief-only → `brief`; +`exploration` → `exploration`. Keep `displayId` for a later **Update**, **Follow-up**, **Commit**, **Decline**, or **Archive**; always **Load** for pins. The response `url` is the DecRec UI for this decision.

On tool error, branch on `code` (**Errors**).

## Update

Fail-fast: **Load** → stop if `isLocked` / `activeExploration` → if admitting a pick, also stop if `!isApproved` (approve at `url`) → **Shape** the new brief from the tip plus the requested changes (send all six fields) → pins from **Load** → call `update_decision` tool with `decRef` + pins + `brief` (+ `exploration` if admitting a pick) → **Report** → stop.

Writes a new tip. +`exploration` → stage `exploration` (owner only; current tip must be approved). Prior tip recommendation no longer applies. Same brief, new pick → **Follow-up**, not this path. Brief-only: any org member (draft tip allowed).

On tool error, branch on `code` (**Errors**).

## Follow-up

Fail-fast: **Load** → stop if `isLocked` / `!isApproved` / `activeExploration` → pins from **Load** → call `admit_exploration` tool with `decRef` + pins + exploration → **Report** → stop.

Do not send `recommendation` as `exploration`. A Follow-up always writes a new run on the **current** approved tip and becomes the tip recommendation. Brief changed → **Update**, not Follow-up.

On tool error, branch on `code` (**Errors**).

## Commit

**Commit** in this skill is DecRec only — not `git commit`. Take `explorationRunId` from `recommendation` (don’t invent; don’t send the whole object).

Fail-fast: **Load** → stop if `isLocked` / `!isApproved` / `activeExploration` / no `recommendation` → pins + `explorationRunId` from **Load** → call `commit_decision` tool with `decRef` + pins + `explorationRunId` → **Report** → stop.

On tool error, branch on `code` (**Errors**).

## Decline

Decline is “we decided not to” (no Commitment row). Take `explorationRunId` from `recommendation` (don’t invent; don’t send the whole object).

Fail-fast: **Load** → stop if `isLocked` / `!isApproved` / `activeExploration` / no `recommendation` → pins + `explorationRunId` from **Load** → call `decline_decision` tool with `decRef` + pins + `explorationRunId` → **Report** → stop.

On tool error, branch on `code` (**Errors**).

## Archive

Fail-fast: **Load** → stop if `status=archived` → `expectedDecisionVersion` from **Load** → call `archive_decision` tool with `decRef` + `expectedDecisionVersion` → **Report** → stop.

Committed or declined may still be archived (`isLocked` is not a stop). In-flight exploration is cancelled by Archive. Do not send `briefVersion` or `explorationRunId`.

On tool error, branch on `code` (**Errors**).

## Errors

Tool failures return `{ "code": "pin_mismatch", "error": "…" }`. Branch on **`code`**, not `error` text. MCP auth that never reaches a tool (not signed in / connection not allowed) is `unauthorized`. Schema/protocol errors with no `code` (empty strings, extra keys, partial `roles`) are `invalid_body` — fix; do not Update this skill.

| `code` | Do |
|---|---|
| `unauthorized` | **Stop.** Sign in or Allow the DecRec connection. |
| `not_owner` | **Stop.** Only the owner can Follow-up, Commit, Decline, Archive, or Update decision with `exploration`. Brief-only Update decision is any org member. |
| `not_found` | **Stop.** Ask for a real `DEC-n` or UUID. |
| `invalid_body` | Fix parameters (empty strings, extra keys, limits, partial `roles`). Do not Update this skill. |
| `locked` | **Stop.** Committed, declined, or archived — no in-place reopen. Archive of an already-archived decision is `locked`. |
| `not_approved` | **Stop.** Approve at the decision `url`. |
| `in_flight` | Wait, or cancel at the `url`. Do not retry until clear. Archive cancels in-flight — only if they asked to archive. |
| `pin_mismatch` | **Load**; re-apply fail-fast. On Update / Follow-up / Commit / Decline: `briefVersion` moved vs this chat’s held pin → **stop**, show the six brief-field changes, ask (do not send the old brief or exploration). Only `expectedDecisionVersion` changed, or Archive → take the new pin, confirm, retry **once**. |
| `display_id_race` | Retry Create with the same parameters **once**. If it fails again, **stop**. |
| `exploration_run_mismatch` | **Load**. If `recommendation.explorationRunId` changed or is missing → **stop**; show the new pick; don’t commit or decline a rec they didn’t confirm. |
| `internal` | Unexpected — **Update this skill**. |

Unknown `code` → **Update this skill**.

## Tool calls

Nested fields follow each tool’s MCP schema. `exploration` is `{ synthesis, roles? }` — do not hoist synthesis fields onto `exploration`. Empty strings fail — use `"unspecified"` (Shape). Pins and `explorationRunId` from **Load** — never invent. Omit unused keys (never `{}` or partial `roles`).

Top-level shapes:

| Call | Send |
|---|---|
| `create_decision` brief-only | `title`, `brief` |
| `create_decision` + pick | `title`, `brief`, `exploration` (`synthesis` required; optional `roles`) |
| `update_decision` brief-only | `decRef`, `briefVersion`, `expectedDecisionVersion`, `brief` |
| `update_decision` + pick | `decRef`, `briefVersion`, `expectedDecisionVersion`, `brief`, `exploration` (`synthesis` required; optional `roles`) |
| `admit_exploration` | `decRef`, `briefVersion`, `expectedDecisionVersion`, `exploration` (`synthesis` required; optional `roles`) |
| `commit_decision` | `decRef`, `briefVersion`, `expectedDecisionVersion`, `explorationRunId` |
| `decline_decision` | `decRef`, `briefVersion`, `expectedDecisionVersion`, `explorationRunId` |
| `archive_decision` | `decRef`, `expectedDecisionVersion` |
| `get_decision` | `decRef` |
| `list_decisions` | optional `limit`, `cursor` (omit `cursor` on the first page) |
| `search_decisions` | `query` |
| `list_councils` | (none) |
| `get_council` | optional `councilRef` (uuid or name; omit → org default) |
| `whoami` | (none) |

## Report

Do **not** paste pins, run ids, or brief-field JSON into the chat. Last `briefVersion` + six fields stay in session only so **Reload** can detect a brief move. Do not dump `stage`/`status` enums or run UUIDs.

Speak to the human in a few normal sentences (not a fence, not a labeled receipt):
- Link `{displayId}` to `{url}`
- What just happened, in product language (created, updated the brief, recorded a pick, committed, declined, archived)
- If a pick was written: the preferred option in plain words
- One optional next step only if it helps (e.g. open the URL to rename or attach evidence). Title, evidence, approve, and cancel in-flight are at the `url` — no skill tools for those. Do not list skill verbs.

When committed, declined, or archived: the record is locked — do not suggest editing title/evidence or further mutate unless they asked to archive.

Do not implement the pick unless they asked — recording is not a request to implement it. Do not chain commit, decline, or archive unless they asked.

On tool **error**, follow **Errors**. It is fine to show the error `code`; keep pins internal.

Examples (shape, not copy-paste): “Created [DEC-12](url) with an approved brief.” / “[DEC-11](url) is committed: keep offer-only capture for now.”

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
- MCP server `decrec` — provides `get_decision`, `list_decisions`, `search_decisions`, `create_decision`, `update_decision`, `admit_exploration`, `commit_decision`, `decline_decision`, `archive_decision`, `list_councils`, `get_council`, `whoami` tools