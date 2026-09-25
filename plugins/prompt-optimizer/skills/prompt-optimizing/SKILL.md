---
name: prompt-optimizing
description: Rewrite a raw, unstructured prompt into a clearer, better-structured one before it is acted on. Applies the 3Cs framework by default, OPAL for structured content-creation tasks, and a conditional Self-Ask instruction for analytical/multi-factor questions. Use when asked to optimize, rewrite, tighten, or restructure a prompt.
---

# Prompt Optimizing

Decision procedure for rewriting a raw prompt. Follow these steps in order.

## 1. Classify the task type

- **Structured content-creation task** — the request asks to produce a
  document/guide/report/article with recognizable phases (introduction,
  steps, conclusion; e.g. "write a guide to X", "create a report on Y") →
  use the **OPAL template** (§3).
- **Everything else** (default) → use the **3Cs template** (§2).

## 2. 3Cs template (default)

Fill these three slots from what the raw prompt implies — never invent
facts or specifics the user didn't state or clearly imply:

- **Context** — what background/complexity level should the model assume?
  Keep it concise (1-2 sentences).
- **Clarity** — state the concrete intent in one unambiguous sentence.
  Resolve vague verbs ("look at", "handle") into a specific action.
- **Constraints** — output format, length, style, or role, if inferable or
  if a sane default improves the answer (e.g. a "write X" request with no
  stated length benefits from a rough word/length constraint).

Assemble into a single prompt, in this shape:

```
[Role, if one is implied or clearly useful]
[Context: 1-2 sentences]
[Clarity: the concrete task, one sentence]
[Constraints: format/length/style, if any]
```

## 3. OPAL template (structured content-creation tasks)

Fill the four slots, again only from what's inferable — do not invent
domain facts:

- **Observations** — background/context the model should know before
  starting.
- **Process** — the phases/structure the output should follow.
- **Action** — the concrete output type and style/tone.
- **Limitations** — what to avoid or exclude.

Assemble as:

```
Observations: "..."
Process: "..."
Action: "..."
Limitations: "..."
```

## 4. Self-Ask add-on (conditional)

If the request is analytical or multi-factor — it asks "why", "how does X
affect Y", asks for a comparison, or asks to explain a causal chain or a
question that plausibly has several contributing factors — append this
line to the assembled prompt (after the 3Cs or OPAL block):

```
Zerlege die Frage zuerst in Teilfragen, beantworte jede einzeln,
synthetisiere dann die Endantwort.
```

Do not add this for simple, single-fact, or single-step requests.

## 5. Explicitly excluded techniques

- **Least-to-Most prompting** — do not apply. It requires seeing a
  response and escalating across multiple turns; a single-pass rewrite
  has no result to react to.
- **Temperature** — do not mention or suggest setting it. On current
  Claude models (Sonnet 5, Opus 5/5.5, Fable 5/5.1, Opus 4.7/4.8) the
  `temperature` API parameter is removed and returns HTTP 400; it is not
  a lever available to the rewriter or the model being asked.

## 6. Already well-structured prompts

If the raw prompt already states role/context and constraints clearly,
return it unchanged, or with only a minimal, targeted fix (e.g. resolving
one ambiguous word). Do not restructure or lengthen a prompt that doesn't
need it.

## 7. Output contract

Return ONLY the rewritten prompt text. No preamble, no explanation of what
changed, no meta-commentary, and do not attempt to answer or solve the
task the prompt describes.
