---
name: writing-zenn-article
description: Use when writing, creating, or editing a Zenn article in this repository. Covers CLI commands, frontmatter fields, Zenn-specific markdown syntax, and the publishing workflow.
---

# Writing Zenn Articles

## Overview

This repository manages [23prime](https://zenn.dev/23prime)'s Zenn articles via GitHub sync. Articles live in `articles/` as Markdown files. Push to `main` to publish.

## Creating an Article

```bash
mise run zenn-new-article -- <name>
```

Creates `articles/<name>-<random>.md` with a unique slug. Edit the frontmatter (title, emoji, type, topics) as needed. If you change the slug, rename the file to match:

```bash
mv articles/<old-slug>.md articles/<new-slug>.md
```

Then create a branch using the final slug.

### Slug Rules

- 12–50 characters
- Lowercase letters, numbers, hyphens (`-`), underscores (`_`) only
- Becomes the URL path and filename: `articles/<slug>.md`

## Frontmatter Reference

```yaml
---
title: "記事タイトル"          # required
emoji: "🛠"                   # required — single emoji
type: "tech"                  # required — "tech" (技術) or "idea" (アイデア)
topics: ["mise", "cli"]       # up to 5 tags
published: false              # false = draft, true = published
published_at: "2026-04-12 09:00" # optional — JST, scheduled publish
---
```

**There is no `--published` CLI flag.** Set `published: true` in the frontmatter to publish.

## Writing Guidelines

### Headings

Use h2–h4. h1 is reserved for the article title.

### Zenn-Specific Syntax

**Message boxes:**

```md
:::message
通常のメッセージ
:::

:::message alert
警告・注意情報
:::
```

**Accordion:**

```md
:::details タイトル
折りたたみ内容
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
![alt](/path/to/img.png "キャプション")
![alt](/path/to/img.png =500x)
```

### Notes

- Single line breaks are ignored — use a blank line between paragraphs.
- HTML tags are not supported (except `<br>`).

## Preview

```bash
mise run zenn-preview
# Opens http://localhost:8000
```

## Publishing Workflow

1. Decide on a name for the article (e.g. `mise-dev-environment`).

2. Create a branch:

    ```bash
    git switch -c article/<name>
    ```

3. Generate the article file:

    ```bash
    mise run zenn-new-article -- <name>
    ```

    A file named `articles/<name>-<random>.md` is created.

4. Write the article. Preview with `mise run zenn-preview`.

5. When ready to publish, set `published: true` in frontmatter.

6. Commit and push:

    ```bash
    git add articles/<slug>.md
    git commit -m "docs: Add article about <topic>"
    git push -u origin article/<slug>
    ```

7. Open a Pull Request targeting `main`. Zenn syncs automatically when the PR is merged.

## Commit Type

Use `docs` for article-related commits (new articles, edits, drafts).

## Common Mistakes

| Mistake | Fix |
| ------- | --- |
| `npx zenn new:article` | Use `mise run zenn-new-article` |
| `--published true` flag | Set `published: true` in frontmatter |
| Slug under 12 chars | Minimum 12 characters |
| h1 inside article body | Start from h2 |
| Single line break | Use blank line to separate paragraphs |
