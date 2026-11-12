---
name: preview-scorecard
description: Create a shareable RooQuiz preview scorecard — a scored questionnaire where each option adds points toward a total that buckets into levels — and get a link to open in the browser. No account, login, or API key required. Use this when someone wants to build, try out, or share a self-assessment, health/habit score, readiness or maturity check, satisfaction survey, or any rated questionnaire that turns answers into a total score and a level (no "correct" answer). For a right/wrong graded quiz, use the preview-quiz skill; for a personality/type test with no scores, use the preview-outcome skill. For a questionnaire whose result is a personalized AI-written report rather than a score or a type, use the preview-ai-quiz skill.
---

# Create a RooQuiz Preview Scorecard

POST a scorecard as JSON to RooQuiz's open preview endpoint and instantly get a **short-lived (~1 hour)**, **browser-openable** preview link. The creation endpoint is public (`access.create => true`), so this needs **no account, login, API key, or credentials** — anything that can make an HTTP request can use it.

A **scorecard** (`scene: "scored_quiz"`) is a scored questionnaire: there's no "correct" answer — instead each option carries a point value, the `report.formula` totals them, and `report.levels` buckets the total into a result band (e.g. "Needs Work" → "Healthy"). It produces a **temporary preview**, not a permanently published form — the link expires automatically (hit **Publish** on the preview page to carry it into RooQuiz and keep it).

Three sibling skills cover the other assessment types — pick the one that matches:
- **preview-quiz** — a right/wrong assessment where correct answers earn points and the taker gets a graded score.
- **preview-outcome** — a personality / type test where options vote for result types and the most-voted type wins (no scores).
- **preview-ai-quiz** — an unscored questionnaire (uploads allowed) where an AI writes each taker a personalized report.

## Three steps

1. **Build the scorecard JSON** (structure below).
2. **Create it:** `POST {PREVIEW_BASE}/api/preview-forms` with the JSON as the body and header `Content-Type: application/json`. Returns `{ doc: { publicToken, expiresAt }, message }`.
3. **Hand back the link:** `{QUIZ_BASE}/b/{publicToken}` — that is the whole link, nothing to append.

### Endpoints

| | Default (RooQuiz cloud) | Override env var |
| --- | --- | --- |
| `PREVIEW_BASE` (create) | `https://preview.rooquiz.com` | `ROOQUIZ_PREVIEW_BASE` |
| `QUIZ_BASE` (open preview) | `https://quizster.app` | `ROOQUIZ_QUIZ_BASE` |

Override the env vars only when targeting a self-hosted RooQuiz deployment; otherwise the defaults work as-is.

### Create it

Send one HTTP request. There is no auth — `Content-Type: application/json` is the only required header. Use whatever HTTP client your environment has (an agent's built-in fetch/HTTP tool, `curl`, `requests`, `fetch`, Postman, …):

```http
POST https://preview.rooquiz.com/api/preview-forms
Content-Type: application/json

<the scorecard JSON as the raw request body>
```

For example, with `curl` (write the JSON to a file first, or inline it with `--data`):

```bash
curl -sS -X POST https://preview.rooquiz.com/api/preview-forms \
  -H 'Content-Type: application/json' \
  --data-binary @/tmp/preview-form.json
```

The response is JSON shaped like:

```json
{ "doc": { "publicToken": "7k3m9q2p", "expiresAt": "2026-06-16T09:12:00.000Z" }, "message": "..." }
```

Read `doc.publicToken` and build the preview link to hand the user:

```
https://quizster.app/b/<publicToken>
```

That is the complete link — nothing to append. For a self-hosted RooQuiz, swap the two hosts for your deployment's preview-API and quiz hosts (see the endpoints table above).

## Scorecard JSON — top level

```jsonc
{
  "scene": "scored_quiz",            // required & fixed for this skill. Sets allowed question types and scoring. Cannot change after creation.
  "title": "My Scorecard",         // required
  "description": "Optional intro",
  "language": "en_US",             // form language; default zh_CN. See "language values" in Notes.
  "personalized": {                // appearance; omit to use defaults (light theme)
    "key": "default",
    "theme": { "name": "light" }
  },
  "indexDisplayMode": "number",    // question numbering: none (default) | number | uppercase | roman
  "fields": [ /* questions — see "Question types" and "Scoring" */ ],
  "report": { /* results page — see "Report configuration" */ },
  "rules": []                      // conditional-display rules (scorecard and outcome scenes only); usually []
}
```

Key points:
- Every question needs a **unique `code`** (string); option `code`s must be unique within their question. A `code` must be a valid identifier — start with a letter or `_`, then only letters/digits/`_` (no `-`, spaces, or leading digit), max 64 chars, and not a reserved math word (`e`, `E`, `pi`, `PI`, `tau`, `phi`, `i`, `Infinity`, `NaN`, `true`, `false`, `null`, `undefined`). The server rejects violations with HTTP 400. Codes are also what the `formula` references.
- `name` is the question text; `description` is optional helper text.
- **Rich text is HTML, never Markdown.** The form `description`, a question's `description`, `Statement` `content`, a level's `description` and `summaryTemplate` render as HTML: write `<p>`, `<strong>`, `<em>`, `<ul>`/`<ol>`/`<li>`, `<a href>`. Markdown such as `**bold**` or `- item` is not rendered as formatting.
- Unknown top-level fields are silently ignored by the server (no error).

## Themes

`personalized.theme.name` sets the visual theme. Omit `theme` to default to `light`. Set it as `"personalized": { "key": "default", "theme": { "name": "synthwave" } }`. There is no `personalized.layout` — card is the only answering layout. (The per-field `layout` (`list`/`grid`) is a different thing and still valid.)

Pick a theme that fits the questionnaire's topic/mood. **To use a random theme** — when the user asks for one, wants variety, or has no preference — just choose a random `name` from this list when building the JSON (there's no server-side "random" option). Recommended palette:

| name | vibe / good for |
| --- | --- |
| `light` | clean neutral bright; default, formal/general questionnaires |
| `corporate` | professional blue+gray; B2B, career, business, maturity checks |
| `dark` | modern sleek dark; tech, night, cool |
| `cupcake` | soft pink, cute, rounded; fun, food, kids, lighthearted |
| `pastel` | gentle pastel artsy; lifestyle, aesthetics, soft mood |
| `valentine` | pink romantic hearts; love, relationships, holidays |
| `synthwave` | neon purple/pink retro; gaming, trends, bold |
| `luxury` | dark + gold premium; finance, luxury, high-end |
| `forest` | deep green nature; environment, health, outdoors, wellness |
| `coffee` | warm brown cozy; food & drink, cafe, lifestyle |
| `autumn` | warm orange/brown seasonal; autumn, cozy, harvest |
| `halloween` | purple + orange spooky; Halloween, horror, festive |
| `night` | deep calm blue; astronomy, mindfulness, calm tech |
| `cyberpunk` | high-contrast neon yellow; tech, esports, gaming |

The full daisyUI theme set also renders (e.g. `emerald`, `dracula`, `retro`, `nord`, `sunset`, `winter`, `lofi`, `garden`, `aqua`, `business`, `lemonade`, `dim`, `bumblebee`, `acid`, `fantasy`, `wireframe`, `black`, `cmyk`) — the table above is just the recommended palette. The server doesn't validate the name, so a typo silently falls back to default styling rather than erroring.

## Question types

Each field is `{ type, code, name, ... }`. In a scorecard you can use:

- **Choice** (carry `choices: [{ code, value }]`, where `value` is the option label):
  `SingleCheck` (single), `MultiCheck` (multiple; optional `min`/`max`), `DropDown`, `TrueFalse` (no `choices`; optional `trueLabel`/`falseLabel` relabel the two buttons, ≤100 chars each)
- **Input**: `NumberField`, `DateField` (needs `precision`), `TimeField`, `Rate` (needs `steps`, optional `words`)
- **Collect only** (never scored, and unusable in the `formula`): `FillBlank` (optional `multiline`) for free text — "anything else?", a lead's name or email — and `Attachment` for material the taker uploads (optional `maxFiles` 1–3, `acceptGroups` drawn from `image` / `pdf` / `document` / `spreadsheet`)
- **Display only** (never scored): `Statement` (`content`), `Breaker` (page break / divider), `Swiper` (image carousel, `items`)

`Ordering` is the only type a scorecard rejects — order-sensitive grading exists solely in the quiz scene. Optional shared props on any field: `required`, `description`, `explain`, `hidden`, `activeColor` (`primary`/`secondary`/`accent`/`neutral`), `layout` (`list`/`grid`).

## Scoring

Give each option a point value with **`partialScoring`** (a field-level array, sitting next to `choices`). Do **not** set `correctAnswer` or `exactScoring` — there are no "correct" answers in a scorecard.

```jsonc
"partialScoring": [
  { "value": "never", "score": 0 },
  { "value": "some",  "score": 5 },
  { "value": "daily", "score": 10 }
]
```

Each `value` must be one of that question's `choices[].code`s (for `TrueFalse`, which has no `choices`, they are the strings `"true"` and `"false"`). A `Rate` question scores its selected star value by default — add a `partialScoring` entry whose `value` is that star (`"1"`, `"2"`, …) to override it, which is how reverse scoring and non-linear weights are done. A `NumberField` contributes the number itself; a `DateField`/`TimeField` contributes its timestamp.

Only these types produce a score the `formula` can reference: `SingleCheck`, `MultiCheck`, `DropDown`, `TrueFalse`, `Rate`, `NumberField`, `DateField`, `TimeField`. `FillBlank`, `Attachment` and the display-only fields (`Statement`/`Breaker`/`Swiper`) are never scored — keep their codes out of the formula. The per-question scores are combined by the `report.formula` (below).

## Report configuration

`report.overallAnalysis` is **required**. A scorecard needs a **`formula`** to total the questions, plus `levels` to bucket the total.

```jsonc
"report": {
  "overallAnalysis": {
    "title": "Your Score",
    "formula": "q1 + q2",          // total from question codes via + - * / ( )
    "summaryTemplate": "<p><art-field data-type=\"fieldVariable\" data-cid=\"score\"></art-field> pts · <art-field data-type=\"fieldVariable\" data-cid=\"level\"></art-field></p>",
    "levels": [                    // map total score → level (see strict rules below)
      { "minScore": null, "maxScore": 10, "label": "Needs Work" },
      { "minScore": 10, "maxScore": 20, "label": "On Track" },
      { "minScore": 20, "maxScore": null, "label": "Excellent", "description": "Top tier!" }
    ]
  }
}
```

**Strict `levels` rules** (violations return HTTP 400): the first level's `minScore` must be `null` (−∞); the last level's `maxScore` must be `null` (+∞); middle levels have finite numbers on both ends; for adjacent levels `current.minScore === previous.maxScore`; `maxScore` strictly increases down the array. A single level has `null` on both ends.

Levels may carry a `color` (hex) and a `cta` (results-page button): `{ enabled, type: "link", label, text, url }` — `label` is the line above the button (≤100 chars), `text` the button itself (≤30), `url` (≤2048) an `http`/`https` link. When the cta is enabled, `text` and `url` are required. CTAs always open in a new tab; there is no `newWindow` key any more (send it and it is ignored).

`summaryTemplate` defaults to a built-in template if omitted. To hide every score the taker sees — the numeric score / score ring and percentile, every dimension score, the points badge on each question — set the top-level `"hideScores": { "enabled": true }`; levels and their write-ups still show. (It used to be `overallAnalysis.hideOverallScore`; that key is gone and is now silently ignored.) Useful when the raw total is arbitrary and the band is the message.

The `formula` is more than `+ - * / ( )`: `abs` `ceil` `floor` `round` `min` `max` `sum` `mean` `pow`, the comparison operators and a `cond ? a : b` ternary all evaluate, so `round((q1 + q2) / 2)` or `min(q1, 5) + q2` are fine.

## Multi-dimension scoring (optional)

When one total isn't enough — a wellness score split into sleep / diet / movement, a maturity check split by capability — add `report.dimensionAnalysis` for per-area scores and a radar chart:

```jsonc
"dimensionAnalysis": {
  "title": "By area",
  "dimensions": [
    {
      "code": "sleep",           // required; what an option's dimensionScores point at
      "name": "Sleep",           // required
      "standardScore": 10,       // required — the dimension's full mark, i.e. the radar scale
      "formula": "q1 + q2",      // required in practice: a dimension with no formula scores 0
      "weight": 1,
      "levels": [ /* same strict rules as overallAnalysis.levels; dimension levels take no cta */ ],
      "description": "Optional blurb",
      "suggestionTemplate": "Supports {{score}}, {{level}}, {{standardScore}}"
    }
  ],
  "radarSettings": { "showRadarChart": true, "showStandardLine": true, "shape": "polygon" }
}
```

A dimension's score is its own `formula`, evaluated over the same question codes. By default a question is worth the same points in every dimension; to make one option count differently in one dimension, hang `dimensionScores` on that option's `partialScoring` entry:

```jsonc
"partialScoring": [
  { "value": "never", "score": 0,  "dimensionScores": [{ "code": "sleep", "score": 0 }] },
  { "value": "daily", "score": 10, "dimensionScores": [{ "code": "sleep", "score": 4 }] }
]
```

`score` remains the option's contribution to the overall total; `dimensionScores[].score` replaces it while the named dimension's formula runs (a dimension with no entry just uses `score`). Every `code` must match a dimension in `dimensionAnalysis.dimensions`, and one option can't list the same dimension twice.

## Conditional rules (optional)

`rules` shows or hides later questions based on earlier answers — ask about team habits only to people who said they manage a team, for example:

```jsonc
"rules": [
  {
    "conditions": [{ "fieldCode": "manages", "operator": "AnyIn", "value": ["yes"] }],
    "action": { "type": "show", "fieldCodes": ["team_size", "one_on_ones"] }
  }
]
```

- Operators: `AnyIn` / `NoneIn` for `SingleCheck`, `MultiCheck`, `DropDown`, `Rate` and `TrueFalse`, with `value` an array of option codes (`Rate` uses its star values `"1"`, `"2"`, …; `TrueFalse` uses `"true"` / `"false"`). `Eq` also works on `TrueFalse`. For `NumberField`, `DateField` and `TimeField`, use `Gt` / `Gte` / `Lt` / `Lte` with a number, or `Between` with `[min, max]`.
- All `conditions` in one rule must hold (AND). For OR, write separate rules with the same `action`.
- Every question in `fieldCodes` must come **after** all of that rule's condition questions in `fields`.
- A question targeted by a `show` rule stays hidden until one of its `show` rules fires; a question targeted only by `hide` rules stays visible until one fires. The same conditions can't both show and hide one question.
- A hidden question is skipped entirely: it isn't required, it scores nothing and drops out of the `formula`, and any condition that points at it never holds, so hiding cascades down a chain of rules.

## Common mistakes

These wrong patterns get reached for out of habit; the server rejects them with HTTP 400. The formats above are correct — match them exactly.

- **Option lists are `choices`, never `options`.** Every choice question uses `"choices": [{ "code": "a", "value": "Label" }]`. There is no `options` key.
- **Codes must be valid identifiers, not arbitrary text** — no hyphens (`type-a`), spaces, or leading digits (`1q`), and not reserved math words. Watch out for **5+ options coded `a,b,c,d,e`**: `e` (Euler's number) is reserved and rejected — use `o1, o2, …` or another non-reserved identifier.
- **Scoring is a field-level `partialScoring` array, not a value nested inside each choice.** Correct: `"partialScoring": [{ "value": "a", "score": 5 }]` next to `choices`. Wrong: `{ "code": "a", "value": "…", "score": 5 }` on the option itself.
- **Don't set `correctAnswer`/`exactScoring`** — a scorecard has no right answers, so they contribute nothing to the score; worse, `exactScoring` on a choice question makes `partialScoring` mandatory and earns a 400 when it's missing. Use `partialScoring` alone.
- **Provide a `formula`** that references your question codes — without it the total won't add up. And `report.overallAnalysis` is required.

## Complete example

The `personalized.theme.name` is matched to the topic (see **Themes** above) — swap in any other name, or pick one at random.

```json
{
  "scene": "scored_quiz",
  "title": "Healthy Habits Score",
  "language": "en_US",
  "personalized": { "key": "default", "theme": { "name": "forest" } },
  "fields": [
    {
      "type": "SingleCheck", "code": "q1", "name": "How often do you exercise?",
      "choices": [
        { "code": "never", "value": "Never" },
        { "code": "some", "value": "Sometimes" },
        { "code": "daily", "value": "Daily" }
      ],
      "partialScoring": [
        { "value": "never", "score": 0 },
        { "value": "some", "score": 5 },
        { "value": "daily", "score": 10 }
      ]
    },
    {
      "type": "SingleCheck", "code": "q2", "name": "Hours of sleep per night?",
      "choices": [
        { "code": "low", "value": "Under 6" },
        { "code": "ok", "value": "6-8" },
        { "code": "great", "value": "8+" }
      ],
      "partialScoring": [
        { "value": "low", "score": 0 },
        { "value": "ok", "score": 5 },
        { "value": "great", "score": 10 }
      ]
    }
  ],
  "report": {
    "overallAnalysis": {
      "title": "Your Wellness Score",
      "formula": "q1 + q2",
      "levels": [
        { "minScore": null, "maxScore": 10, "label": "Needs Work", "color": "#e57373" },
        { "minScore": 10, "maxScore": null, "label": "Healthy", "color": "#4caf50" }
      ]
    }
  }
}
```

## Notes & limits

- **Expiry:** previews self-destruct after about **1 hour** (`expiresAt`); the link 404s afterward. To keep one, open it and hit **Publish** in the preview's top bar — that carries this exact scorecard (questions, scoring, report, theme) into the RooQuiz editor, prefilled from the preview token.
- **Rate limit:** anonymous creation is capped at **5 previews per minute** per IP; past that the endpoint answers **429** — wait a minute and retry.
- **Validation errors:** a 400 nests the details — `errors[0].data.errors[]`, each entry with a `path` and a `message` (e.g. `fields.1.type`). Read those, fix the JSON, retry. Most common: a question type that doesn't match `scene: "scored_quiz"` (`Ordering`), a choice question carrying `exactScoring` without `partialScoring`, or non-contiguous `levels`. The `formula` itself is not checked on create — a typo'd code only shows up once the scorecard is published.
- **What the preview page shows:** a top bar with **Quiz** / **Report** tabs and a **Publish** button. The questionnaire is fully answerable, but **answers are never saved and nothing is totalled** — submitting just explains preview mode. The **Report** tab renders a *sample* report with placeholder numbers (score ring, levels, radar) to show the layout; it does not run your `formula` or `levels` against the answers. Real scoring starts once the scorecard is published, so `report` is still validated on create and travels with the form — just don't expect the preview to compute it.
- **`language` values:** `en_US` `de_DE` `es` `pt_BR` `fr` `zh_CN` (default) `zh_TW` `ja_JP` `ko_KR`.
