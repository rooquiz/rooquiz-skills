---
name: preview-ai-quiz
description: Create a shareable RooQuiz preview AI quiz — a questionnaire with no scoring where an AI reads each taker's answers (and any uploaded photo, audio, video or document) and writes them a personalized report — and get a link to open in the browser. No account, login, or API key required. Use this when someone wants to build, try out, or share an AI-written assessment, diagnosis, review, consultation intake, skin / style / resume / pitch analysis, or any questionnaire whose result is a custom write-up rather than a score or a type. For a right/wrong graded quiz, use the preview-quiz skill; for a scored questionnaire totaled into a level, use the preview-scorecard skill; for a personality/type test, use the preview-outcome skill.
---

# Create a RooQuiz Preview AI Quiz

POST an AI quiz as JSON to RooQuiz's open preview endpoint and instantly get a **short-lived (~1 hour)**, **browser-openable** preview link. The creation endpoint is public (`access.create => true`), so this needs **no account, login, API key, or credentials** — anything that can make an HTTP request can use it.

An **AI quiz** (`scene: "ai_quiz"`) scores nothing. Its questions only collect answers — choices, free text, numbers, ratings, even uploaded files — and once the quiz is published, an AI reads every answer and writes each taker a personalized report, steered by the **generation instructions** you put in `report.aiReport`. It produces a **temporary preview**, not a permanently published form — the link expires automatically (hit **Publish** on the preview page to carry it into RooQuiz and keep it).

Three sibling skills cover the other assessment types — pick the one that matches:
- **preview-quiz** — a right/wrong assessment where correct answers earn points and the taker gets a graded score.
- **preview-scorecard** — a scored questionnaire where each option adds points toward a total + level.
- **preview-outcome** — a personality / type test where options vote for result types and the most-voted type wins.

Pick this one when the result should be a write-up tailored to the person rather than a number or one of a fixed set of types.

## Three steps

1. **Build the AI quiz JSON** (structure below).
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

<the AI quiz JSON as the raw request body>
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

## AI quiz JSON — top level

```jsonc
{
  "scene": "ai_quiz",              // required & fixed for this skill. Sets allowed question types and scoring. Cannot change after creation.
  "title": "My AI Assessment",     // required
  "description": "Optional intro",
  "language": "en_US",             // form language, and the language the AI writes in; default zh_CN. See "language values" in Notes.
  "personalized": {                // appearance; omit to use defaults (light theme)
    "key": "default",
    "theme": { "name": "light" }
  },
  "indexDisplayMode": "number",    // question numbering: none (default) | number | uppercase | roman
  "fields": [ /* questions — see "Question types" */ ],
  "report": { /* the AI's brief — see "Report configuration" */ }
}
```

Key points:
- Every question needs a **unique `code`** (string); option `code`s must be unique within their question. A `code` must be a valid identifier — start with a letter or `_`, then only letters/digits/`_` (no `-`, spaces, or leading digit), max 64 chars, and not a reserved math word (`e`, `E`, `pi`, `PI`, `tau`, `phi`, `i`, `Infinity`, `NaN`, `true`, `false`, `null`, `undefined`). The server rejects violations with HTTP 400. Give questions **meaningful codes** (`goal`, `budget`, `photo`) — they are how the generation instructions point the AI at an answer.
- `name` is the question text; `description` is optional helper text.
- **Rich text is HTML, never Markdown.** The form `description`, a question's `description` and `Statement` `content` render as HTML: write `<p>`, `<strong>`, `<em>`, `<ul>`/`<ol>`/`<li>`, `<a href>`. Markdown such as `**bold**` or `- item` is not rendered as formatting.
- Unknown top-level fields are silently ignored by the server (no error).

## Themes

`personalized.theme.name` sets the visual theme. Omit `theme` to default to `light`. Set it as `"personalized": { "key": "default", "theme": { "name": "synthwave" } }`. There is no `personalized.layout` — card is the only answering layout. (The per-field `layout` (`list`/`grid`) is a different thing and still valid.)

Pick a theme that fits the topic/mood. **To use a random theme** — when the user asks for one, wants variety, or has no preference — just choose a random `name` from this list when building the JSON (there's no server-side "random" option). Recommended palette:

| name | vibe / good for |
| --- | --- |
| `light` | clean neutral bright; default, general |
| `corporate` | professional blue+gray; B2B, career, business, resume / pitch reviews |
| `dark` | modern sleek dark; tech, night, cool |
| `cupcake` | soft pink, cute, rounded; fun, food, kids, lighthearted |
| `pastel` | gentle pastel artsy; beauty, skincare, lifestyle, aesthetics |
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

Each field is `{ type, code, name, ... }`. In an AI quiz you can use:

- **Choice** (carry `choices: [{ code, value }]`, where `value` is the option label):
  `SingleCheck` (single), `MultiCheck` (multiple; optional `min`/`max`), `DropDown`, `TrueFalse` (no `choices`; optional `trueLabel`/`falseLabel` relabel the two buttons, ≤100 chars each)
- **Input**: `FillBlank` (free text; optional `multiline`), `NumberField`, `Rate` (needs `steps`, optional `words`)
- **Upload**: `Attachment` — **`kind` is required** and decides what the taker uploads and how the AI reads it:

  | `kind` | taker uploads | the AI | per-file limit |
  | --- | --- | --- | --- |
  | `image` | photos | looks at them | 10 MB |
  | `audio` | a recording | reads a transcript | 10 MB, 3 min |
  | `video` | a clip | reads a transcript | 20 MB, 60 s |
  | `document` | PDF, `.docx` / `.xlsx` / `.pptx`, or plain text / CSV / Markdown (not legacy `.doc` / `.xls`) | reads the extracted text | 10 MB |

  Optional `maxFiles` (1–3, default 3). **At most 2 `Attachment` questions per quiz** (any mix of kinds), and a taker can submit **at most 3 files in total** across them — two questions with `maxFiles: 3` each still allow only 3. `acceptGroups` is ignored once `kind` is set.
- **Display only**: `Statement` (`content`), `Breaker` (page break / divider), `Swiper` (image carousel, `items`)

`Ordering` is **not** allowed. Optional shared props on any field: `required`, `description`, `hidden`, `activeColor` (`primary`/`secondary`/`accent`/`neutral`), `layout` (`list`/`grid`). Keep it to **20 questions or fewer** (page breaks and display blocks don't count): the preview accepts more, but a longer AI quiz is rejected when you publish it.

## No scoring

Nothing is graded, so **do not set `correctAnswer`, `exactScoring`, `partialScoring`, or `outcomeScoring`** on any field — each one is rejected with HTTP 400 in this scene. There is also no `formula`, no `levels`, no `dimensionAnalysis`, and no conditional `rules`. Everything the taker learns comes from the AI's report.

## Report configuration

Two parts:

- **`report.overallAnalysis`** is **required** — give it a `title` (the report's heading). Nothing else in it applies here.
- **`report.aiReport`** is the AI's brief, and it is what makes the quiz worth publishing:
  - **`instructions`** — what the report should cover, how to read the answers, what to recommend. At most **20000 characters**, HTML markup included.
  - **`style`** (optional) — tone, length, and things to avoid, as plain text. At most **5000 characters**.

```jsonc
"report": {
  "overallAnalysis": { "title": "Your Personalized Plan" },
  "aiReport": {
    "instructions": "<p>Build a 4-week plan around <art-field data-type=\"fieldVariable\" data-cid=\"goal\"></art-field>.</p><p>Fit the sessions to <art-field data-type=\"fieldVariable\" data-cid=\"days\"></art-field> and keep the gear to what <art-field data-type=\"fieldVariable\" data-cid=\"equipment\"></art-field> allows.</p>",
    "style": "Encouraging and concrete. Under 400 words. No medical advice."
  }
}
```

**Pointing the AI at an answer.** Write `instructions` as HTML paragraphs and reference a question with a variable tag carrying its code:

```html
<art-field data-type="fieldVariable" data-cid="goal"></art-field>
```

That tells the AI which answer a sentence is about; the answer itself is not pasted in, since the AI always sees every answer anyway. Only reference **visible input questions** (not `Statement`/`Breaker`/`Swiper`, not `hidden` ones), and use the exact `code`. Write the tag exactly as shown, and do not write `{{code}}` placeholders — the preview stores `instructions` verbatim, so a `{{goal}}` would reach the editor as literal text, not as a variable. Plain text with no tags is accepted, but it only becomes rich text if it starts with a tag such as `<p>`. Stick to basic rich text — `<p>`, `<strong>`, `<em>`, `<ul>`/`<ol>`/`<li>`, `<a href>`.

Write instructions the way you would brief an analyst: what to look at, how to weigh it, what sections the report should have, and what to recommend next. The AI also picks its own layout and can chart estimates it makes, so asking for "a short summary, then three priorities, each with one action" works well.

### Result tags (optional)

To sort takers into segments for lead follow-up, add up to **10** tags under `report.outcomeAnalysis.outcomes`. Before writing each report, the AI picks the one tag that fits the taker. Tags are internal: takers never see the tag name, but the team can filter leads by it.

```jsonc
"outcomeAnalysis": {
  "outcomes": [
    { "code": "hot", "name": "Ready to buy", "description": "Has a budget and wants to start within a month." },
    { "code": "nurture", "name": "Nurture", "description": "Interested, but no budget or timeline yet." }
  ]
}
```

Each tag needs a unique `code` and a `name`. Put the criteria for choosing it in `description`, since that is what the AI reads. An optional `cta` (`{ enabled, type: "link", label, text, url }`, same rules as elsewhere) shows a button under the report of takers who get that tag. The preview does not enforce the 10-tag cap, but publishing does.

## Common mistakes

These wrong patterns get reached for out of habit; the server rejects most of them with HTTP 400. The formats above are correct — match them exactly.

- **Option lists are `choices`, never `options`.** Every choice question uses `"choices": [{ "code": "a", "value": "Label" }]`. There is no `options` key.
- **Codes must be valid identifiers, not arbitrary text** — no hyphens (`skin-type`), spaces, or leading digits (`1q`), and not reserved math words. Watch out for **5+ options coded `a,b,c,d,e`**: `e` (Euler's number) is reserved and rejected — use `o1, o2, …` or another non-reserved identifier.
- **No scoring keys.** `correctAnswer`, `exactScoring`, `partialScoring` and `outcomeScoring` all earn a 400 here — this is not a scored scene.
- **`Attachment` without `kind`** is rejected, and so is a third `Attachment` question.
- **`{{code}}` in `instructions`.** That shorthand is for RooQuiz's MCP server, which converts it for you. The preview endpoint does not convert it — use the `<art-field … data-cid="code">` tag.
- **Empty `aiReport`.** The preview accepts it, but a published AI quiz with no instructions has nothing to steer its reports. Always write `instructions`.

## Complete example

The `personalized.theme.name` is matched to the topic (see **Themes** above) — swap in any other name, or pick one at random.

```json
{
  "scene": "ai_quiz",
  "title": "Your Skincare Routine, Reviewed",
  "description": "Answer a few questions and share a photo — get a routine built for your skin.",
  "language": "en_US",
  "personalized": { "key": "default", "theme": { "name": "pastel" } },
  "fields": [
    {
      "type": "SingleCheck", "code": "skin_type", "name": "How would you describe your skin?", "required": true,
      "choices": [
        { "code": "oily", "value": "Oily" },
        { "code": "dry", "value": "Dry" },
        { "code": "combo", "value": "Combination" },
        { "code": "sensitive", "value": "Sensitive" }
      ]
    },
    {
      "type": "MultiCheck", "code": "concerns", "name": "What would you like to improve?",
      "choices": [
        { "code": "acne", "value": "Breakouts" },
        { "code": "spots", "value": "Dark spots" },
        { "code": "lines", "value": "Fine lines" },
        { "code": "dull", "value": "Dullness" }
      ]
    },
    {
      "type": "FillBlank", "code": "routine", "name": "What do you use now, morning and night?", "multiline": true
    },
    {
      "type": "Attachment", "code": "photo", "name": "Add a makeup-free photo in daylight (optional)",
      "kind": "image", "maxFiles": 1
    }
  ],
  "report": {
    "overallAnalysis": { "title": "Your Skin Report" },
    "outcomeAnalysis": {
      "outcomes": [
        { "code": "routine_ready", "name": "Ready for a routine", "description": "Already uses two or more products and wants to optimize." },
        { "code": "beginner", "name": "Beginner", "description": "Uses one product or none and needs the basics first." }
      ]
    },
    "aiReport": {
      "instructions": "<p>Start with a two-sentence read of the skin, combining <art-field data-type=\"fieldVariable\" data-cid=\"skin_type\"></art-field> with what the photo <art-field data-type=\"fieldVariable\" data-cid=\"photo\"></art-field> shows, if there is one.</p><p>Then review the current routine in <art-field data-type=\"fieldVariable\" data-cid=\"routine\"></art-field>: what to keep, what to drop, and why.</p><p>Finish with a simple morning and evening routine that targets <art-field data-type=\"fieldVariable\" data-cid=\"concerns\"></art-field>, naming ingredient types rather than brands.</p>",
      "style": "Warm and practical. Short sections with bullet points. No medical diagnoses — suggest a dermatologist for anything persistent."
    }
  }
}
```

## Notes & limits

- **Expiry:** previews self-destruct after about **1 hour** (`expiresAt`); the link 404s afterward. To keep one, open it and hit **Publish** in the preview's top bar — that carries this exact quiz (questions, generation instructions, result tags, theme) into the RooQuiz editor, prefilled from the preview token.
- **Rate limit:** anonymous creation is capped at **5 previews per minute** per IP; past that the endpoint answers **429** — wait a minute and retry.
- **Validation errors:** a 400 nests the details — `errors[0].data.errors[]`, each entry with a `path` and a `message` (e.g. `fields.1.type`). Read those, fix the JSON, retry. Most common: a question type the scene doesn't allow (`Ordering`), a scoring key left on a field, or an `Attachment` with no `kind`. The art-field references in `instructions` are not checked on create — a mistyped code only shows up in the editor after publishing.
- **What the preview page shows:** a top bar with **Quiz** / **Report** tabs and a **Publish** button. The questionnaire is fully answerable, but **answers are never saved and no AI runs** — submitting just explains preview mode. The **Report** tab renders a *sample* AI report in the form's language to show the layout; it is not generated from your instructions. Real reports start once the quiz is published.
- **`language` values:** `en_US` `de_DE` `es` `pt_BR` `fr` `zh_CN` (default) `zh_TW` `ja_JP` `ko_KR`.
