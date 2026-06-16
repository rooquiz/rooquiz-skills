# RooQuiz Skills for Claude

Build shareable [RooQuiz](https://rooquiz.com) assessments straight from Claude — just ask.

This repo is a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces.md). It ships the **`rooquiz-tools`** plugin, whose **`create-preview-quiz`** skill lets Claude spin up a quiz, scorecard, or personality test and hand you a live preview link — **no account, login, or API key required**.

## Install

```bash
claude plugin marketplace add rooquiz/rooquiz-skills
claude plugin install rooquiz-tools@rooquiz
```

> Replace `rooquiz` (the owner) with your GitHub org/username if you forked this repo.

## Use it

Just ask Claude in natural language:

> “Make a 5-question personality quiz about coffee preferences and give me a link.”

Claude builds the assessment and replies with a link like `https://quizster.app/b/7k3m9q2p` you can open or share. Previews are temporary (~1 hour) — recreate it in RooQuiz to keep it permanently.

Or invoke the skill directly:

```
/rooquiz-tools:create-preview-quiz
```

## What's inside

| Plugin | Skill | What it does |
| --- | --- | --- |
| `rooquiz-tools` | `create-preview-quiz` | Create a quiz / scorecard / personality (outcome) / random-quiz preview and get a shareable link |

## How it works

The skill `POST`s your assessment JSON to RooQuiz's open preview endpoint (`https://preview.rooquiz.com/api/preview-forms`) and builds a link to open it on `quizster.app`. It's a thin, transparent client — no credentials, just `curl`. The full field schema, scenes, scoring rules, and copy-paste examples live in the skill itself:

- [`create-preview-quiz` skill](plugins/rooquiz-tools/skills/create-preview-quiz/SKILL.md)

## Links

- [RooQuiz](https://rooquiz.com) — the lightweight assessment platform for lead capture and shareable results
