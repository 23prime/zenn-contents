---
name: reviewing-zenn-article
description: Use when reviewing a Zenn article for quality, correctness, and style. Checks content quality, Zenn rule compliance, and runs lint/spell checks. Invoked by the writing-zenn-article skill or independently after editing an article.
---

# Reviewing a Zenn Article

## Overview

Review a Zenn article for quality, correctness, and style. Run all checks and report findings. Fix auto-fixable issues; report the rest for manual resolution.

## Workflow

### 1. Identify the article

Read the article file at `articles/<slug>.md`.

### 2. Run automated checks

Run the following sequentially (fix first, then check):

```bash
mise run fix
mise run check
```

Report any errors or warnings that remain after `mise run fix`.

### 3. Content quality review

Check the article against each of the following criteria. Report findings in a structured list.

#### Structure

- [ ] Has an introduction that explains the background and what the article covers
- [ ] Main sections follow a logical order
- [ ] Has a conclusion or summary that recaps key points
- [ ] Uses h2–h4 only (no h1 in the body)
- [ ] Each section heading (h2–h4) starts with a fitting emoji

#### Target audience

- [ ] The target reader is clear (explicitly stated or implied consistently)
- [ ] Assumed prior knowledge matches the stated audience level
- [ ] No unexplained jargon that would confuse the target reader

#### Code samples

- [ ] Code samples are correct and runnable
- [ ] Each code sample is explained in the surrounding text
- [ ] Filenames are specified on code blocks where helpful (e.g., ` ```toml:mise.toml `)

#### Writing style

- [ ] Sentences are clear and concise
- [ ] No awkward or unnatural Japanese expressions
- [ ] Consistent tone and register throughout
- [ ] Paragraphs are separated by blank lines (not single line breaks)

#### Zenn compliance

- [ ] `published` field is set correctly (`false` for draft, `true` for publish)
- [ ] `emoji` is a single emoji character
- [ ] `topics` has 5 or fewer tags
- [ ] No HTML tags (except `<br>`)
- [ ] Image paths are absolute (`/images/<slug>/...`), not relative

### 4. Typo check

Read the full article text and identify:

- Misspelled words (Japanese or English)
- Wrong kanji (e.g., 違和感 written incorrectly)
- Obvious autocorrect errors

### 5. Report findings

Present a summary of all findings:

```md
## Review Results

### Automated checks
<pass / list of errors>

### Content quality
<checklist results — only list items that failed>

### Typos
<list of typos with location and suggested correction, or "None found">

### Summary
<overall assessment and recommended next steps>
```

If there are no issues, say so clearly.
