---
name: writing-zenn-article
description: このリポジトリで Zenn の記事を執筆・作成・編集するときに使う。CLI コマンド、frontmatter フィールド、Zenn 固有の markdown 記法、公開フローを網羅する。
translated_from: SKILL.md
---

# Zenn 記事の執筆

## 概要

このリポジトリは GitHub 連携を通じて [23prime](https://zenn.dev/23prime) の Zenn 記事を管理する。記事は `articles/` に Markdown ファイルとして配置する。`main` ブランチへ push することで公開される。

## 記事の作成

```bash
mise run zenn-new-article -- <name>
```

`articles/<name>-<random>.md` が生成される。frontmatter の title・emoji・type・topics を編集する。slug を変更した場合は、ファイル名も合わせてリネームする:

```bash
mv articles/<old-slug>.md articles/<new-slug>.md
```

### Slug のルール

- 12〜50文字
- 小文字英字・数字・ハイフン（`-`）・アンダースコア（`_`）のみ使用可
- URL パスおよびファイル名になる: `articles/<slug>.md`

## Frontmatter リファレンス

```yaml
---
title: "記事タイトル"          # 必須
emoji: "🛠"                   # 必須 — 絵文字1文字
type: "tech"                  # 必須 — "tech"（技術）または "idea"（アイデア）
topics: ["mise", "cli"]       # タグ（最大5個）
published: false              # false = 下書き、true = 公開
published_at: "2026-04-12 09:00" # 任意 — JST、予約公開
---
```

**`--published` という CLI フラグは存在しない。** 公開するには frontmatter で `published: true` を設定する。

## 執筆ガイドライン

### 見出し

h2〜h4 を使う。h1 は記事タイトル用に予約されている。

### Zenn 固有の記法

**メッセージボックス:**

```md
:::message
通常のメッセージ
:::

:::message alert
警告・注意情報
:::
```

**アコーディオン:**

```md
:::details タイトル
折りたたみ内容
:::
```

**ファイル名付きコードブロック:**

````md
```toml:mise.toml
[tools]
node = "22"
```
````

**埋め込み（リンクカード）:**

```md
@[card](https://example.com)
```

**Mermaid ダイアグラム:**

````md
```mermaid
graph TD
  A --> B
```
````

**メディア埋め込み:**

```md
@[youtube](VIDEO_ID)
@[github](owner/repo)
@[tweet](TWEET_ID)
```

**数式（KaTeX）:**

```md
$$
E = mc^2
$$
```

**キャプション・幅指定付き画像:**

```md
![alt](/path/to/img.png "キャプション")
![alt](/path/to/img.png =500x)
```

### 注意事項

- 単一の改行は無視される — 段落間は空行で区切る。
- HTML タグは非サポート（`<br>` を除く）。

## プレビュー

```bash
mise run zenn-preview
# http://localhost:8000 が開く
```

## 公開フロー

1. 記事の名前を決める（例: `mise-dev-environment`）。

2. ブランチを作成する:

    ```bash
    git switch -c article/<name>
    ```

3. 記事ファイルを生成する:

    ```bash
    mise run zenn-new-article -- <name>
    ```

    `articles/<name>-<random>.md` が作成される。

4. 記事を執筆する。`mise run zenn-preview` でプレビュー確認する。

5. 公開する準備ができたら frontmatter で `published: true` を設定する。

6. コミット・プッシュする:

    ```bash
    git add articles/<slug>.md
    git commit -m "docs: Add article about <topic>"
    git push -u origin article/<slug>
    ```

7. `main` を対象に Pull Request を作成する。マージされると Zenn が自動的に同期する。

## コミットタイプ

記事関連のコミット（新規記事・編集・下書き）には `docs` を使う。

## よくある間違い

| 間違い | 修正方法 |
| ------- | -------- |
| `npx zenn new:article` | `mise run zenn-new-article` を使う |
| `--published true` フラグ | frontmatter で `published: true` を設定する |
| 12文字未満の slug | 最低12文字必要 |
| 記事本文に h1 を使う | h2 から始める |
| 単一改行 | 段落間は空行で区切る |
