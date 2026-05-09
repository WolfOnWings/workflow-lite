---
name: research
description: Dispatch the `research-analyst` subagent for prior-art investigation, multi-source triangulation, or documented research that grounds a decision. Wrapper workshops the question framing with the user via iterate-until-yes loop. Surfaces summary + speculation inline. Use when the user invokes `/research`.
---

INPUT: research topic from immediate prior context, and/or from user (free-form string via `/research <topic>`)

### Step 1: Workshop the question
Draft a neutral question + scope from the user's topic and recent context.

### Step 2: Hypothesis neutrality check
Read the drafted question cold — no conversation history. Run four probes:

- **Paraphrase**: restate the question. If the restatement splits at "and" joining distinct asks → split: each ask becomes its own dispatch.
- **Loaded terms**: list every adjective and verb.
  - **Direction-loaded** ("best", "fail", "broken", "leak") — telegraphs the hoped-for answer → swap to neutral ("what techniques affect…", "how does X relate to…").
  - **Criterion-loaded** ("better", "worse", "improved", "good") — implies a success dimension you haven't named → name it from context if you have it, else ask the user before redrafting.
- **Response space**: if the question implies a binary, both alternatives must be named. "Should X be allowed?" → "allowed or forbidden?"
- **Cold-read predictability**: can you predict what answer the asker wants? If yes → leaking, redraft.

IF split → recurse with each split question independently.
ELSE IF any probe fires → redraft, re-run probes.
ELSE → surface question(s) + scope (no commentary).
IF user approves → Step 3. Ambiguous = feedback, not approval.
ELSE → redraft on feedback, re-run probes.

### Step 3: Dispatch verbatim
Pass the approved string to `Agent({subagent_type: "research-analyst", prompt: "<approved question + scope verbatim>", run_in_background: true})` character-for-character. Any deviation — paraphrase, cleanup, added context, even whitespace normalization — reopens the bias surface and requires a fresh approval pass.

### Step 4: Surface the brief
Surface in this order:
1. The agent's `summary` field, verbatim — the findings digest.
2. The brief's `path` field, so the user can read the full brief.
3. Your own hypothesis paragraph on updated direction (informed by the evidence; not a guess).

OUTPUT: `.scratch/research/<topic-slug>-MM-DD.md` (path on disk) and the inline brief

## Anti-patterns

### Premature dispatch
- **Detection:** Orch skips Step 1 or Step 2 (e.g., raw user topic forwarded to the agent unmodified).
- **Resolution:** Always run Steps 1-2, even when the topic looks self-evident.

### Probe theater
- **Detection:** Step 2 ran but a probe's prescribed resolution was skipped — loaded term left in, alternatives unstated, criterion guessed instead of asked.
- **Resolution:** A probe's resolution path is binding. Apply it or re-enter the workshop loop.

### Paraphrase on dispatch
- **Detection:** The string passed to `Agent()` differs from the user-approved question.
- **Resolution:** Copy character-for-character. If it needs work, reopen the workshop loop.

### Commentary on workshop surface
- **Detection:** Surfacing the question with framing notes, hedges, or explanations beyond question + scope.
- **Resolution:** Surface only question + scope. Commentary belongs in Step 4, after the agent returns.

### Pre-dispatch speculation
- **Detection:** Orch surfaces a guess at the answer before the agent returns.
- **Resolution:** Hold all hypotheses until Step 4. Pre-research framing must be neutral.
