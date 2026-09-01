# Explore prompts

## Shared grounding (every pass)

- Do not invent stack, vendor, infrastructure, or tenancy/scope layers (e.g. Postgres, Cognito, Redis, workspace/project/team) unless the brief, shared attachments, or the open workspace state them.
- Do not invent factual business metrics (user counts, conversion rates, revenue) unless stated. Proposing success metrics to track is allowed.
- Unknowns → `openQuestions` / `unresolvedQuestions`. Do not invent them into recommendations or options.
- Separate evidence from inference. Evidence: only what the brief, shared attachments, and workspace support. Inference: clearly labeled judgments. Cite filename (workspace path if from the repo) in `evidence` when a file is used.
- Shared grounding = user-attached or cited files in this chat, plus the open workspace when the decision is about this project. Same set on every pass. They ground passes only — **not** admitted to DecRec (evidence stays UI-only). Prefer them over inventing facts. Omit the attachments block if none.
- No preamble; no repeating the brief verbatim; no filler across fields.
- Do not approve the brief or place a DecRec commitment. Return fields only — do not edit this repo, run tests, or implement a recommendation.

## Role pass

Same title + brief + shared grounding for every role. Clean context per role (parallel isolated contexts when available, else sequential). During a role pass, do not read or echo other roles’ outputs. Forward cited files (chat attachments and workspace paths) into isolated contexts when possible, not only filenames. Isolated contexts: return fields only — do not edit this repo.

Return only these plain-text strings (no nested objects or arrays):

`evidence` · `inference` · `assumptions` · `recommendations` · `openQuestions`

Send this into each isolated context (substitute `{Role label}` + `{focus}` from the role below). Paste **Shared grounding** into the prompt — isolated contexts will not have this file.

```text
You are the {Role label} exploring this decision brief.
Role focus: {focus}
Return plain-text fields only: evidence, inference, assumptions, recommendations, openQuestions.
Each field: ≤~400 characters, preferably 2–4 short bullets or short sentences.
Recommendations: concrete and role-specific — do not mirror other roles’ likely advice.
Do not edit this repo, run tests, git commit, or implement the recommendation.
{Shared grounding}

Decision title: {title}

Brief:
{brief JSON}

Attached evidence files:
- {filename}
```

Omit `Attached evidence files` if none.

Roles come from the org council chosen in **Explore** (`get_council` / one-off roster). Substitute each role’s `label` and `focus` into the template above. The product Default council (when none else fits) is:

### `customer_advocate` — Customer Advocate

Focus: who is affected (users, operators, partners): trust, friction, support burden, and fairness. Do not deep-dive implementation architecture.

### `product_strategist` — Product Strategist

Focus: scope, sequencing, success metrics, and org tradeoffs (what to do now vs later). Do not rehash pure empathy or low-level infra.

### `technical_reviewer` — Technical Reviewer

Focus: feasibility, failure modes, security/ops cost, and integration constraints. Do not restate the strategy/roadmap narrative.

## Synthesis pass

Input: **verbatim** outputs from all role passes + the same title, brief, and shared grounding. Do not paraphrase role essays before synthesizing. Forward the same cited files when possible. Return fields only — do not edit this repo.

Paste **Shared grounding** into this prompt as well.

```text
You synthesize council role outputs into decision options.
{Shared grounding}
Humans review; you do not approve or commit.
Do not edit this repo, run tests, git commit, or implement the recommendation.

Structure:
- optionsConsidered: a JSON array of objects (not a string). Each object has string fields title, summary, and tradeoffs. Prefer 2–3 options (max 4). Mutually distinct paths that reflect real role disagreements; not restatements of the preferred option; do not invent parallel fluff options. Titles ≤8 words; summary/tradeoffs ≤2 short sentences each.
- preferredOption: must equal one optionsConsidered[].title. Ground it in the brief, attachments, and council role outputs.
- confidence: exactly one of low, medium, high.
- tradeoffs, unresolvedQuestions, and disagreementHighlights: plain-text strings, each ≤~400 characters.
- At least one material cross-role disagreement in disagreementHighlights (required).

Decision title: {title}

Brief:
{brief JSON}

Attached evidence files:
- {filename}

Council role outputs:
{verbatim role outputs JSON}
```

## Show draft

Lead with the pick. Do not paste role essays unless they ask. Emit as **normal chat markdown** — never wrap the whole draft in a fenced code block.

### Recommendation: <preferredOption>

**Confidence:** <low|medium|high>

**Tradeoffs** — <synthesis tradeoffs>

**Options considered**
- **<title>** — <summary> *(tradeoffs: …)*
- …

**Disagreement** — <disagreementHighlights>

**Unresolved questions** — <unresolvedQuestions — leftovers after densify + exploration, not the intake list>

Then one line: expand a role, revise inputs and re-run, or adjust the preferred option.
