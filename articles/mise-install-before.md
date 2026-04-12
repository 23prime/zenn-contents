---
title: "mise で Minimum Release Age 的なものを設定する"
emoji: "🔒"
type: "tech"
topics: ["mise", "security", "supplychain", "cli", "devtools"]
published: false
---

## 🚨 はじめに

近年、オープンソースのエコシステムを狙ったサプライチェーン攻撃が増加しています。IPA（情報処理推進機構）の「[情報セキュリティ10大脅威 2026](https://www.ipa.go.jp/security/10threats/10threats2026.html)」では、「サプライチェーンや委託先を狙った攻撃」が組織向け脅威の2位にランクインしており、継続的な脅威として認識されています。

有名な事例をいくつか挙げると：

- **axios**（2026年）：メンテナーアカウントが乗っ取られ、バックドア付きバージョンが約 3 時間公開された。（[参考](https://blog.flatt.tech/entry/axios_compromise)）
- **Trivy**（2026年）：セキュリティスキャナー自体が標的になり、悪意ある Trivy バイナリが CI/CD パイプライン経由で拡散した。（[参考](https://www.microsoft.com/en-us/security/blog/2026/03/24/detecting-investigating-defending-against-trivy-supply-chain-compromise/)）
- **XZ Utils**（2024年）：約2年かけてコントリビューターとして信頼を獲得した攻撃者が、SSH デーモンへのバックドアを仕込んだ。ローリングリリース系 Linux ディストリビューションに一時的に混入した。

axios や Trivy のような短時間の乗っ取りに対しては、**「リリース直後のバージョンをインストールしない」** という戦略が有効な防御になりえます。一方、XZ Utils のような長期潜伏型の攻撃にはこの戦略は通用しないので、別途注意が必要です。

## ⏳ Minimum Release Age とは

この「リリースから一定期間が経過しないとインストールしない」という考え方は、パッケージ管理ツールの世界で **Minimum Release Age**（最小リリース経過時間）として広まっています。

### 🔧 各ツールの対応状況

| ツール | 設定項目 | 単位 |
| --- | --- | --- |
| Renovate | [`minimumReleaseAge`](https://docs.renovatebot.com/configuration-options/#minimumreleaseage) | 期間文字列（例: `"3 days"`） |
| Dependabot | [`cooldown`](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference#cooldown) | 日数（例: `days: 3`） |
| pnpm（v10.6+） | [`minimumReleaseAge`](https://pnpm.io/settings#minimumreleaseage) | 分（例: `4320`） |
| npm（v11.10+） | [`min-release-age`](https://docs.npmjs.com/cli/v11/using-npm/config#min-release-age) | 日数（例: `3`） |
| Bun | [`minimumReleaseAge`](https://bun.sh/docs/pm/cli/install#minimum-release-age) | 秒（例: `259200`） |
| Yarn 4 | [`npmMinimalAgeGate`](https://yarnpkg.com/configuration/yarnrc#npmMinimalAgeGate) | 期間文字列（例: `"3d"`） |
| uv（Python） | [`exclude-newer`](https://docs.astral.sh/uv/reference/settings/#exclude-newer) | 絶対日時（例: `"2024-01-01T00:00:00Z"`） |

## 🛠️ mise の install_before

### 📌 概要

[mise](https://mise.jdx.dev/) は、Node.js・Python・Go・Rust などの開発ツールのバージョンを統合管理するツールです。

`install_before` は mise のバージョン管理レイヤーに Minimum Release Age を適用する設定です。指定した期間より新しいバージョンはインストール対象から除外されます。

### 📋 バージョン要件

| 機能 | 最低 mise バージョン |
| --- | --- |
| グローバル設定・環境変数・CLI フラグ | v2025.12.8 |
| ツール単位設定 | v2026.4.1 |

### ⚙️ 設定方法

#### グローバル設定

全プロジェクト共通のデフォルトとして設定するには、`~/.config/mise/config.toml` に追記します。

```toml:~/.config/mise/config.toml
[settings]
install_before = "3d"
```

#### プロジェクト単位の設定

プロジェクトの `mise.toml` に設定することで、そのプロジェクト内のみに適用できます。

```toml:mise.toml
[settings]
install_before = "3d"
```

#### ツール単位の設定

特定のツールだけに適用したい場合は、`[tools]` セクションで個別に設定します。

```toml:mise.toml
[tools.node]
version = "22"
install_before = "3d"
```

#### 環境変数・CLI フラグ

環境変数や CLI フラグでも指定できます。

```bash
# 環境変数
export MISE_INSTALL_BEFORE=3d

# CLI フラグ（一時的に上書き）
mise install --before 3d
```

### 📋 設定の優先順位

同じ設定が複数の場所で定義されている場合、以下の順で優先されます（上ほど優先）。

1. CLI フラグ（`--before`）
2. ツール単位の設定（`[tools.node] install_before`）
3. 環境変数（`MISE_INSTALL_BEFORE`）
4. プロジェクトの `[settings]`（`mise.toml`）
5. グローバル設定（`~/.config/mise/config.toml`）

また、複数の `mise.toml` が存在する場合（例：ネストしたディレクトリ）は、カレントディレクトリに近いファイルが優先されます。

:::message
環境変数とファイルの優先順位についてはドキュメントに記載がなく、[ソースコード](https://github.com/jdx/mise/blob/main/src/config/settings.rs)から確認しています。
:::

### ⚠️ 注意点・制限

#### バージョン固定時はスキップされる

`install_before` は、`node@22` や `latest` のようなファジー指定・範囲指定のバージョン解決に対してのみ有効です。`node@22.5.0` のように完全バージョンを固定している場合は、フィルタリングがスキップされます。

```toml:mise.toml
[tools]
# ✅ install_before が効く（ファジー指定）
node = "22"

# ❌ install_before がスキップされる（バージョン固定）
node = "22.5.0"
```

意図的に新しいバージョンを使いたい場合はバージョン固定という使い分けができます。

#### 対応バックエンドのみ有効

リリース日のタイムスタンプを提供しているバックエンドでのみ機能します。現時点で対応しているのは **aqua、cargo、npm、pipx** などです。タイムスタンプ情報を持たないバックエンドでは、フィルタリングは行われません。

npm 系ツールの場合、各パッケージマネージャーのネイティブな仕組み（`npm --before`、`bun --minimum-release-age`、`pnpm --config.minimumReleaseAge` など）に転送される形で動作します。

## 🤔 どのくらいの期間を設定すればいいの？

前述の axios や Trivy のように、悪意あるバージョンの露出時間が数時間以内のケースは多く、数日待つだけでも多くの攻撃をブロックできます。一方で、この手法には「セキュリティパッチの適用が遅れる」というトレードオフがあります。

脆弱性の修正リリースを早急に取り込みたい場面では、待機期間が長いほど対応が遅れるリスクがあります。「侵害バージョンをすぐに入れない」と「セキュリティパッチは早急に当てたい」のバランスをどこに置くかはプロジェクトの性質や運用方針によって異なるため、正解はありません。自分たちのリスク許容度に合わせて判断するのがよいでしょう。

## 🗒️ 実際の運用例

参考までに、本記事のリポジトリでは以下のようにツールをファジー指定した上で、`install_before = "3d"` をグローバルに設定しています。

```toml:mise.toml
[settings]
install_before = "3d"

[tools]
markdownlint-cli2 = "0.22"
actionlint = "1"
lefthook = "2"
cspell = "9"
pnpm = "10"
```

メジャーバージョンのみ固定し、マイナー・パッチはファジーに解決させることで、最新バージョンを自動的に追従しつつ、リリースから 3 日以内のバージョンはインストールされないようにしています。

## 📝 おわりに

パッケージ管理の世界では Minimum Release Age の考え方が標準的な防御策として広まっています。

mise の `install_before` は、これをツールバージョン管理レイヤーにも適用できる設定です。`3d` を設定するだけで、多くのサプライチェーン攻撃リスクを軽減できます。

設定自体はシンプルなので、グローバル設定に追加しておくだけでも効果があります。まだ設定していない方は、ぜひ導入をご検討ください！

## 🔗 参考

- [Settings | mise-en-place](https://mise.jdx.dev/configuration/settings.html#install_before)
- [Configuration | mise-en-place](https://mise.jdx.dev/configuration.html)
- [minimumReleaseAge - Renovate Docs](https://docs.renovatebot.com/configuration-options/#minimumreleaseage)
- [Dependabot cooldown - GitHub Docs](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference#cooldown)
- [pnpm minimumReleaseAge](https://pnpm.io/settings#minimumreleaseage)
- [npm min-release-age](https://docs.npmjs.com/cli/v11/using-npm/config#min-release-age)
- [Bun minimumReleaseAge](https://bun.sh/docs/pm/cli/install#minimum-release-age)
- [Yarn 4 npmMinimalAgeGate](https://yarnpkg.com/configuration/yarnrc#npmMinimalAgeGate)
- [uv exclude-newer](https://docs.astral.sh/uv/reference/settings/#exclude-newer)
- [Minimum Release Age is an Underrated Supply Chain Defense](https://daniakash.com/posts/simplest-supply-chain-defense/)
- [mise src/config/settings.rs | GitHub](https://github.com/jdx/mise/blob/main/src/config/settings.rs)
- [情報セキュリティ10大脅威 2026 | IPA](https://www.ipa.go.jp/security/10threats/10threats2026.html)
- [axios npm パッケージのサプライチェーン攻撃についてまとめてみた | Flatt Security Blog](https://blog.flatt.tech/entry/axios_compromise)
- [Guidance for detecting, investigating, and defending against the Trivy supply chain compromise | Microsoft Security Blog](https://www.microsoft.com/en-us/security/blog/2026/03/24/detecting-investigating-defending-against-trivy-supply-chain-compromise/)
