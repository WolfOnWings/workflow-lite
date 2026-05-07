---
name: research
description: "Dispatch the `researcher` subagent for prior-art investigation, multi-source triangulation, or documented research that grounds a decision. Wrapper workshops the question framing with the user via iterate-until-yes loop. Brief lands at `.scratch/research/<topic-slug>-MM-DD.md`; orchestrator surfaces summary + speculation inline. Use when the user invokes `/research`."
---

INPUT: research topic from user (free-form string via `/research <topic>`)

### Step 1: Workshop the question
Draft a neutral question + scope from the user's topic and recent context.
**Verify-pass**: re-read the drafted question against the watchlist's hypothesis-leak detection. IF a pattern matches → redraft and re-verify. ELSE → surface the proposed question + scope (no commentary).
IF user approves → break and proceed to Step 2. Ambiguous replies count as feedback, not approval.
ELSE → take the user's feedback, redraft, return to the verify-pass.

### Step 2: Dispatch verbatim
Pass the approved string to `Agent({subagent_type: "researcher", prompt: "<approved question + scope verbatim>", run_in_background: true})` character-for-character. Any deviation — paraphrase, cleanup, added context, even whitespace normalization — reopens the bias surface and requires a fresh approval pass.

### Step 3: Surface the brief
Take the `summary` field from the agent's return value — relay it verbatim to the user as the findings digest. Then add a brief hypothesis-form paragraph on updated direction. Hypothesis is OK now — you are informed by the evidence.

OUTPUT: `.scratch/research/<topic-slug>-MM-DD.md` (path on disk) and the inline brief

## Anti-pattern watchlist

### Hypothesis leak in framing
- **Detection:** Drafted question contains a prediction, an answer-laden adjective, or a binary frame that excludes a real third option.
- **Resolution:** Rewrite to a neutral interrogative; re-run the verify-pass before re-surfacing.

### Premature dispatch
- **Detection:** Orch dispatches without at least one workshop iteration (e.g., raw user topic forwarded as the question).
- **Resolution:** Always run Step 1 first, even when the topic looks self-evident.

### Paraphrase on dispatch
- **Detection:** The string passed to `Agent()` differs from the user-approved question.
- **Resolution:** Copy the approved string character-for-character. If it needs work, reopen the workshop loop.

### Commentary on workshop surface
- **Detection:** Step 1 surface includes orch's framing notes, hedges, or explanations beyond the question + scope.
- **Resolution:** Surface only the question + scope; commentary belongs in Step 3's speculation, after the agent returns.

### Pre-dispatch speculation
- **Detection:** Orch surfaces a guess at the answer before the agent returns.
- **Resolution:** Hold all hypothesis until Step 3. Pre-research framing must be neutral.
