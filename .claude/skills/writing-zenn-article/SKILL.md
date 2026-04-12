---
name: writing-zenn-article
description: Use when writing, creating, or editing a Zenn article in this repository. Follows a step-by-step workflow from topic planning to PR creation.
---

# Writing Zenn Articles

## Overview

This repository manages [23prime](https://zenn.dev/23prime)'s Zenn articles via GitHub sync. Articles live in `articles/` as Markdown files. Push to `main` to publish.

## Workflow

### 1. Clarify the article

Confirm the following with the user before writing:

- **Topic**: What is the article about?
- **Target audience**: Who is the reader? (e.g., beginners, experienced engineers)
- **Key points**: What should readers take away?
- **Type**: `tech` (technical) or `idea` (opinion/idea)
- **Topics (tags)**: Up to 5 tags (e.g., `["mise", "cli"]`)

### 2. Propose an outline

Draft an outline based on the confirmed information and present it to the user. Wait for approval before writing. If the user requests changes, revise and re-confirm.

A good outline structure:

```md
## Introduction
- Background / motivation
- What this article covers

## <Main Section 1>
...

## <Main Section N>
...

## Conclusion / Summary
- Recap of key points
- Next steps or related resources (if any)
```

### 3. Create a branch

Use the article name (a short, meaningful identifier) as `<name>`:

```bash
git switch -c article/<name>
```

### 4. Generate the article file

```bash
mise run zenn-new-article -- <name>
```

Creates `articles/<name>-<random>.md`. Edit the frontmatter:

```yaml
---
title: "Article Title"
emoji: "🛠"
type: "tech"
topics: ["tag1", "tag2"]
published: false
---
```

If you change the slug, rename the file to match:

```bash
mv articles/<old-slug>.md articles/<new-slug>.md
```

#### Slug Rules

- 12–50 characters
- Lowercase letters, numbers, hyphens (`-`), underscores (`_`) only
- Becomes the URL path and filename: `articles/<slug>.md`

### 5. Write the article

Follow the approved outline. Apply Zenn-specific syntax as needed (see Reference section below).

Writing rules:

- Use h2–h4 only. h1 is reserved for the article title.
- Add a fitting emoji at the start of each section heading (h2–h4). Choose an emoji that matches the content of that section.
- Separate paragraphs with a blank line (single line breaks are ignored).
- HTML tags are not supported (except `<br>`).

### 6. Review

Invoke the `reviewing-zenn-article` skill via the Agent tool:

```js
Agent({
  subagent_type: "general-purpose",
  description: "Review Zenn article",
  prompt: "Use the reviewing-zenn-article skill to review the article at articles/<slug>.md in ~/develop/zenn-contents."
})
```

Fix all issues reported by the review agent before proceeding.

### 7. Publish

When the article is ready to publish, set `published: true` in the frontmatter.

To schedule publishing, set `published_at` (JST):

```yaml
published: true
published_at: "2026-04-12 09:00"
```

### 8. Commit and push

```bash
git add articles/<slug>.md
git commit -m "docs: Add article about <topic>"
git push -u origin article/<slug>
```

Use `docs` for article-related commits (new articles, edits, drafts).

### 9. Open a Pull Request

Open a PR targeting `main`. Zenn syncs automatically when the PR is merged.

---

## Reference

### Zenn-Specific Syntax

**Message boxes:**

```md
:::message
Normal message
:::

:::message alert
Warning / caution
:::
```

**Accordion:**

```md
:::details Title
Hidden content
:::
```

**Code block with filename:**

````md
```toml:mise.toml
[tools]
node = "22"
```
````

**Embed (link card):**

```md
@[card](https://example.com)
```

**Mermaid diagrams:**

````md
```mermaid
graph TD
  A --> B
```
````

**Embed media:**

```md
@[youtube](VIDEO_ID)
@[github](owner/repo)
@[tweet](TWEET_ID)
```

**Math (KaTeX):**

```md
$$
E = mc^2
$$
```

**Image with caption or width:**

```md
![alt](/path/to/img.png "Caption")
![alt](/path/to/img.png =500x)
```

### Images

Place images under `images/<slug>/` in the repository root and reference them with an **absolute path**:

```txt
images/
  <slug>/
    image1.png
```

```md
![alt text](/images/<slug>/image1.png)
![alt text](/images/<slug>/image1.png "Caption")
![alt text](/images/<slug>/image1.png =500x)
```

- Supported formats: `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`
- Max file size: 3 MB
- Relative paths do not work — always use `/images/...`
- Deleting an image from GitHub also removes it from Zenn

### Frontmatter Reference

```yaml
---
title: "Article Title"       # required
emoji: "🛠"                  # required — single emoji
type: "tech"                 # required — "tech" or "idea"
topics: ["mise", "cli"]      # up to 5 tags
published: false             # false = draft, true = published
published_at: "2026-04-12 09:00"  # optional — JST, scheduled publish
---
```

**There is no `--published` CLI flag.** Set `published: true` in the frontmatter to publish.

### Preview

```bash
mise run zenn-preview
# Opens http://localhost:8000
```

### Common Mistakes

| Mistake | Fix |
| ------- | --- |
| `npx zenn new:article` | Use `mise run zenn-new-article` |
| `--published true` flag | Set `published: true` in frontmatter |
| Slug under 12 chars | Minimum 12 characters |
| h1 inside article body | Start from h2 |
| Single line break | Use blank line to separate paragraphs |
