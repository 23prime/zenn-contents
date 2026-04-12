---
name: writing-zenn-article
description: このリポジトリで Zenn の記事を執筆・作成・編集するときに使う。トピックの確認から PR 作成までのステップバイステップのワークフローに従う。
translated_from: SKILL.md
---

# Zenn 記事の執筆

## 概要

このリポジトリは GitHub 連携を通じて [23prime](https://zenn.dev/23prime) の Zenn 記事を管理する。記事は `articles/` に Markdown ファイルとして配置する。`main` ブランチへ push することで公開される。

## ワークフロー

### 1. 記事内容の確認

執筆前にユーザーと以下を確認する：

- **トピック**: 何についての記事か
- **ターゲット読者**: 誰が読む記事か（例: 初心者、経験のあるエンジニア）
- **要点**: 読者に何を持ち帰ってほしいか
- **タイプ**: `tech`（技術）または `idea`（意見・アイデア）
- **Topics（タグ）**: 最大5個（例: `["mise", "cli"]`）

### 2. アウトラインの提案

確認した内容をもとにアウトラインを作成し、ユーザーに提示する。承認を得てから執筆を開始する。修正を求められた場合は改訂して再確認する。

推奨するアウトライン構成：

```md
## はじめに
- 背景・動機
- この記事で扱う内容

## <本編セクション 1>
...

## <本編セクション N>
...

## まとめ
- 要点の振り返り
- 次のステップや関連リソース（あれば）
```

### 3. ブランチの作成

記事名（短く意味のある識別子）を `<name>` として使う：

```bash
git switch -c article/<name>
```

### 4. 記事ファイルの生成

```bash
mise run zenn-new-article -- <name>
```

`articles/<name>-<random>.md` が作成される。frontmatter を編集する：

```yaml
---
title: "記事タイトル"
emoji: "🛠"
type: "tech"
topics: ["tag1", "tag2"]
published: false
---
```

slug を変更した場合は、ファイル名も合わせてリネームする：

```bash
mv articles/<old-slug>.md articles/<new-slug>.md
```

#### Slug のルール

- 12〜50文字
- 小文字英字・数字・ハイフン（`-`）・アンダースコア（`_`）のみ使用可
- URL パスおよびファイル名になる: `articles/<slug>.md`

### 5. 記事の執筆

承認済みのアウトラインに沿って執筆する。必要に応じて Zenn 固有の記法を使用する（後述のリファレンスを参照）。

執筆ルールは `docs/WRITING_RULES.md` に定義されている。執筆時はそちらを読んで従う。

Zenn のコミュニティガイドライン（`references/zenn-guideline.md` を参照）に従う：

- 実体験や独自の考察を含める（公式ドキュメントの要約だけにしない）
- タイトルは内容を正確に反映する（クリックベイト禁止）
- 冒頭に概要を記載し、対象読者を明示する
- AI 生成コンテンツを使用する場合は内容の正確性を確認してから投稿する
- 引用時は出典を明記し、必要な範囲にとどめる

本文中にインラインで引用した URL（例：`（[参考](https://...)）`）は、末尾の `## 参考` セクションにも必ず掲載する。

### 6. レビュー

Agent ツールで `reviewing-zenn-article` スキルを呼び出す：

```js
Agent({
  subagent_type: "general-purpose",
  description: "Zenn 記事のレビュー",
  prompt: "reviewing-zenn-article スキルを使って ~/develop/zenn-contents の articles/<slug>.md をレビューして。"
})
```

レビューエージェントが報告した問題をすべて修正してから次に進む。

### 7. 公開設定

公開準備ができたら frontmatter で `published: true` を設定する。

予約公開する場合は `published_at` も設定する（JST）：

```yaml
published: true
published_at: "2026-04-12 09:00"
```

### 8. コミット・プッシュ

```bash
git add articles/<slug>.md
git commit -m "docs: Add article about <topic>"
git push -u origin article/<slug>
```

記事関連のコミット（新規記事・編集・下書き）には `docs` を使う。

### 9. Pull Request の作成

`main` を対象に Pull Request を作成する。マージされると Zenn が自動的に同期する。

---

## リファレンス

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

### 画像

リポジトリルート直下の `images/<slug>/` に画像を配置し、**絶対パス**で参照する：

```txt
images/
  <slug>/
    image1.png
```

```md
![代替テキスト](/images/<slug>/image1.png)
![代替テキスト](/images/<slug>/image1.png "キャプション")
![代替テキスト](/images/<slug>/image1.png =500x)
```

- 対応フォーマット: `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`
- ファイルサイズ上限: 3MB
- 相対パスは動作しない — 必ず `/images/...` で指定する
- GitHub から画像を削除すると Zenn 上からも削除される

### Frontmatter リファレンス

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

### プレビュー

```bash
mise run zenn-preview
# http://localhost:8000 が開く
```

### よくある間違い

| 間違い | 修正方法 |
| ------- | -------- |
| `npx zenn new:article` | `mise run zenn-new-article` を使う |
| `--published true` フラグ | frontmatter で `published: true` を設定する |
| 12文字未満の slug | 最低12文字必要 |
| 記事本文に h1 を使う | h2 から始める |
| 単一改行 | 段落間は空行で区切る |
