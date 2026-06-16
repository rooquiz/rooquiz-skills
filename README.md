# RooQuiz Skills

Build shareable [RooQuiz](https://rooquiz.com) assessments — a quiz, scorecard, personality test, or scored questionnaire — straight from your AI agent. Just ask in natural language and get back a live preview link you can open or share — **no account, login, or API key required**.

The skill is a single self-contained [`SKILL.md`](plugins/rooquiz-tools/skills/create-preview-quiz/SKILL.md): plain instructions for one public, no-auth HTTP endpoint. That makes it usable by **any AI agent or assistant that can read a skill file and make an HTTP request** — not just Claude. This repo also packages it as a [Claude Code plugin](https://code.claude.com/docs/en/plugin-marketplaces.md) for one-command install.

## Use with Claude Code

```bash
claude plugin marketplace add rooquiz/rooquiz-skills
claude plugin install rooquiz-tools@rooquiz
```

> Replace `rooquiz` (the owner) with your GitHub org/username if you forked this repo.

Then just ask in natural language:

> “Make a 5-question personality quiz about coffee preferences and give me a link.”

The agent builds the assessment and replies with a link like `https://quizster.app/b/7k3m9q2p`. You can also invoke the skill directly with `/rooquiz-tools:create-preview-quiz`.

## Use with any other agent

Install it with the [`skills`](https://github.com/vercel-labs/skills) CLI (works with Cursor, Windsurf, and any agent the CLI supports):

```bash
npx skills add rooquiz/rooquiz-skills
```

> Replace `rooquiz` with your GitHub org/username if you forked this repo.

Or skip the tooling entirely: the skill needs no special runtime. Point your agent at [`SKILL.md`](skills/create-preview-quiz/SKILL.md) — paste it into context, drop it into your agent's skills/instructions directory, or fetch it from this repo. It documents the full request format, scenes, scoring rules, and copy-paste examples. Anything that can issue an HTTP `POST` — a coding agent's fetch tool, a script, `curl`, Postman — can create a preview.

## What's inside

| Plugin | Skill | What it does |
| --- | --- | --- |
| `rooquiz-tools` | `create-preview-quiz` | Create a quiz / scorecard / personality (outcome) / random-quiz preview and get a shareable link |

## How it works

The skill `POST`s your assessment JSON to RooQuiz's open preview endpoint (`https://preview.rooquiz.com/api/preview-forms`) and builds a link to open it on `quizster.app`. It's a thin, transparent client — no credentials, just an HTTP request. Previews are temporary (~1 hour); recreate the assessment in RooQuiz to keep it permanently. The full field schema, scenes, scoring rules, and examples live in the skill itself:

- [`create-preview-quiz` skill](skills/create-preview-quiz/SKILL.md)

The same skill is published in two layouts that must stay in sync: [`skills/create-preview-quiz/SKILL.md`](skills/create-preview-quiz/SKILL.md) (for the `skills` CLI and generic agents) and [`plugins/rooquiz-tools/skills/create-preview-quiz/SKILL.md`](plugins/rooquiz-tools/skills/create-preview-quiz/SKILL.md) (for the Claude Code plugin).

## Links

- [RooQuiz](https://rooquiz.com) — the lightweight assessment platform for lead capture and shareable results
