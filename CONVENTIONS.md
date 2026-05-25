# Chronista Club — Conventions

chronista-club ecosystem 全体に適用される規約。 各 repo はこの規約に従う。

> **status**: draft (2026-05-16) ── 最初の section は命名規則。 規約が増えたら section を追加する。

---

## 命名規則 (Naming Conventions)

### 原則 — `club-` prefix

chronista-club の **crates.io 公開 crate** は `club-` prefix で統一する。

発端: 2026-05-15、 `unison` を crates.io へ publish しようとして既存 crate (RobertWHurst, 2018-、 無関係な config loader) と衝突。 名前を諦めるのではなく ecosystem 共通の prefix を導入して解決した。

prefix 候補の比較:

| Prefix | 長さ | 由来 | 判定 |
|--------|-----|------|------|
| `cc-` | 2 | Chronista Club / Claude Code | 内部ツール用に温存 |
| `chronista-` | 9 | フル | 冗長 |
| **`club-`** | **4** | **chronista-club org 直結** | **✅ 採用** |

### prefix 早見表 — 内部 vs 公開

| Layer | Prefix | 例 | 公開先 |
|-------|--------|----|-------|
| 内部ツール / plugin | `cc-` | `ccwire`, `ccws` | GitHub のみ |
| 公開 crate (library) | `club-` | `club-unison`, `club-kdl`, `club-nostos` | **crates.io** |
| 内部 crate (`publish = false`) | (任意) | `unison-mcp-probe` | publish なし |
| product (CLI tool) | (product 固有) | `creo`, `vp`, `fleetflow` | (個別) |

### prefix を付ける範囲 — registry-listed か code path か

`club-` prefix の責務は **crates.io の global flat namespace での衝突回避**。 よって prefix は 「その identifier が browsable な registry / index に listed されるか」 で判別する:

| 種別 | prefix | 該当 identifier |
|------|--------|----------------|
| registry / index に listed | **あり** | `[package].name` (crates.io)、 GitHub repo、 local checkout dir (`~/repos/...`、 Finder) |
| code path 内に埋もれる | **なし** | `[lib].name` (import path)、 workspace subdir (`crates/...`) |
| human-facing display label | (例外・自由) | creo-memories atlas |

### lib 名は bare name

lib 名 (`[lib].name`、 consumer の `use` で書く識別子) は registry namespace の外 ── code path に属する ── ため **prefix を付けない**。

| crates.io package | lib name (`use`) |
|--|--|
| `club-unison` | `unison` |
| `club-nostos` | `nostos` |

consumer は Cargo の package rename で吸収する:

```toml
[dependencies]
club-unison = "0.10"     # crates.io package 名 (prefix あり)
```
```rust
use unison::Channel;     # lib 名は bare
```

経緯: unison v0.5.0 で 「package 名のみ rename」、 v0.6.0 で 「lib 名も `club_` 化 (full rename policy)」 を一旦採用。 2026-05-16 に見直し、 lib 名は bare へ戻した ── prefix は import を毎行冗長にするコストに見合わず、 衝突問題も registry 外には存在しないため。

> **例外 — `club-kdl`**: bare lib 名 `kdl` は upstream の `kdl` crate と衝突する (両方を依存に持つ project が存在しうる)。 よって `club-kdl` のみ lib 名を `club_kdl` とする。 これは衝突回避という原則に基づく carve-out。

### namespace 別命名 map

1 つの project の identifier を namespace 別に並べると以下になる (例: `nostos` project):

| namespace | prefix | 例 |
|---|---|---|
| GitHub org | (固定) | `chronista-club` |
| GitHub repo | あり | `club-nostos` |
| local checkout (`~/repos/...`) | あり | `club-nostos` |
| `[package].name` (Cargo.toml) | あり | `club-nostos` |
| `[lib].name` (import path) | **なし** | `nostos` |
| workspace subdir (`crates/...`) | **なし** | `nostos` |
| creo-memories atlas | **(例外)** | `Nostos Club` |
| project 通称 (informal) | なし | `nostos` |

### User-facing 使用例 (確定形)

```toml
[dependencies]
club-nostos = "0.1"
```

```rust
use nostos::Bracket;
```

```bash
cargo add club-nostos
git clone https://github.com/chronista-club/club-nostos.git
cd ~/repos/club-nostos
```

### 新規 crate を足すとき

`[package].name = "club-<name>"` / `[lib].name = "<name>"` (bare) で統一する。 bare lib 名が他 crate と衝突する場合のみ、 `club-kdl` 同様に prefix 付き lib 名を例外とする。

---

## Rust API 規約

### 公開 error enum は `#[non_exhaustive]` を最初から付ける

新しく公開する `pub enum *Error { ... }` には **最初から `#[non_exhaustive]` を付ける**。 後付けは 1 度きりの軽い breaking を発生させるので、 ecosystem 共通 default として「最初から」運用する。

#### なぜ

error enum は時間が経つほど variant が増える (Io / Network / 新 dialect / 新統合 …)。 `#[non_exhaustive]` 無しだと variant 追加が毎回 minor breaking になる。 付けておくと:

- maintainer は variant を自由に追加可能 (non-breaking)
- consumer は `match` に `_ => ...` 1 行を強制されるだけ
- `tokio::Error` / `serde_json::Error` 等の ecosystem 慣行と一致

#### 例

```rust
#[derive(Debug, thiserror::Error)]
#[non_exhaustive]
pub enum ParseError {
    #[error("kdl parse error: {0}")]
    Kdl(String),
    // ... 後で Compose / Io / Network 等を追加しても breaking じゃない
}
```

#### 適用範囲

- **公開 error enum**: 必須
- **公開 enum で variant 追加が見込まれるもの**: 推奨
- internal enum / 数学的に閉じた enum (例: `enum BinOp { Add, Sub, Mul, Div }`): 不要

---

## ドキュメント (Documentation)

### README の言語構成

chronista-club ecosystem の OSS project の README は **英語を default、日本語を翻訳として併設** する:

| ファイル | 内容 | 役割 |
|---|---|---|
| `README.md` | **英語** | canonical source。 crates.io / docs.rs / GitHub の README ペインに表示される default |
| `README.ja.md` | **日本語訳** | 国内読者向けの併設翻訳 |

両ファイルの**冒頭に言語切替リンク** を配置し相互リンクする (例: `English | **[日本語](README.ja.md)**` / `**[English](README.md)** | 日本語`)。

#### 理由

OSS publication context では crates.io / docs.rs / GitHub の来訪者は**英語 README を期待する**慣例。chronista-club 内部コミュニケーション (session・コミット・design memo・creo-memories) は **日本語をメイン** にするが、**外向き docs だけは英語 origin** にする stratification を取る。

これは「日本語がメイン」を**内向きコミュニケーション**向けと解釈し、**外向き docs は分けて運用**する明示的な原則。

#### content drift 回避

機能拡張や crate 追加で内容が古くなったら**両言語を同時に更新**する。 README sync は release PR にバンドルすると crates.io ページにも同タイミングで反映されて効率がいい (Living Documentation 原則)。

---

## Git / Commit

### Conventional Commits + 日本語

commit message は **Conventional Commits** 形式の prefix に **日本語の説明**を添える形で書く:

| Prefix | 用途 |
|--------|------|
| `feat:` / `feat(scope):` | 新機能 |
| `fix:` / `fix(scope):` | バグ修正 |
| `chore(release):` | リリース commit (version bump + CHANGELOG) |
| `chore(scope):` | その他の事務 commit (dep 更新等) |
| `docs:` / `docs(scope):` | ドキュメント変更 |
| `test:` / `test(scope):` | テスト変更 |
| `refactor:` / `refactor(scope):` | リファクタ (機能変更を伴わない) |
| `ci:` / `ci(scope):` | CI / workflow 変更 |
| `perf:` / `perf(scope):` | パフォーマンス改善 |

#### 例

```
feat(codegen): protocol 方言に envelope enum + identifier sanitize

VP sidebar IPC を KDL schema 化する Phase A の前提として ...

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

### Claude が編集に関与した commit は `Co-Authored-By` footer を付ける

Claude (Anthropic) が編集に関与した commit には footer を付ける:

```
Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

model 名は実際に動いた model version に合わせる (`Claude Sonnet 4.6` 等)。 trace と review の起点として機能する。

### branch 名は `<author>/<slug>` 形式

```
<author>/<short-slug>
<author>/<memory-short-id>-<slug>   (memory との traceability が欲しい時)
```

例:
- `mako/codegen-uses-compose`
- `mako/release-v0.11.1`
- `mako/1CbBXDUTbz-envelope-enum-sanitize`

memory id を埋め込むと creo-memories の handoff / spec / design と branch の対応が取れる (recall 時に branch 名 → memory 経由で full context を辿れる)。

---

## CI / Repository 構造

### `chronista-club` crate の標準 CI ジョブ構成

新規 club-* crate の `.github/workflows/ci.yml` は **最低限** 以下の 8 job を持つ:

| Job | 内容 | matrix |
|---|---|---|
| `fmt / clippy / doc` | `cargo fmt --check` + `cargo clippy --all-targets --workspace -- -D warnings` + `cargo doc --no-deps --workspace` (`RUSTDOCFLAGS=-D warnings`) | ubuntu-latest |
| `test` | `cargo test --workspace --all-targets` + `cargo test --doc` | 3 OS × stable + ubuntu × beta = 4 |
| `MSRV` | `Cargo.toml` の `rust-version` 自動読み取り + `cargo check` | ubuntu-latest |
| `cargo-deny` | license / advisories / bans | ubuntu-latest |
| `semver-checks` | `cargo-semver-checks` (PR 時のみ) | ubuntu-latest |

#### 環境変数

- `RUSTFLAGS: -D warnings` — warning を error に昇格 (CI のみ)
- `RUSTDOCFLAGS: -D warnings` — doc 内の broken intra-doc-link / 警告を error に

#### 既存 template

`club-kdl` の `.github/workflows/ci.yml` をそのまま流用できる。 bun / node を要する crate のみ `oven-sh/setup-bun` 等を追加。

---

## リリース運用 (Release operations)

### `release.yml` の publish 順は依存グラフ topological に保つ

workspace に新しい crate 間依存を追加したら、 release.yml の `cargo publish` ステップの順序を**必ず再評価する**。 path 依存は local では即解決するが、`cargo publish` は crates.io 上で依存を検証するため、 **依存先が先に publish 済 でないと失敗する**。

#### 教訓 — club-kdl v0.11.0 partial release (2026-05-25)

PR で `codegen → compose` 依存を追加した際に release.yml の publish 順 (`derive → main → codegen → compose`) を見直さなかった結果、 `codegen 0.11.0` の publish 時点で `club-kdl-compose = "^0.11.0"` を crates.io から解決できず失敗。 後続 step も走らず、 **`derive` / `main` のみ 0.11.0 published、 `codegen` / `compose` は 0.10.0 のまま** の partial release 状態に陥った。v0.11.1 で 4 crate を揃え直して復旧。

#### 適用ルール

- workspace member 間の依存を**追加・変更**するたびに `.github/workflows/release.yml` を確認
- publish 順は **依存グラフの topological order** に保つ (依存される側が先)
- 各 publish step の間に `sleep 30` を入れて crates.io の index 反映を待つ既存パターンを踏襲
- `workflow_dispatch` の dry-run step も同じ順序に揃える (順序不整合が起きるのは多くの場合「片方だけ追加して片方を忘れる」)

### CI watch の exit code は pipe で潰さない

`gh run watch --exit-status` の終了コードは workflow の成否を反映するが、 `| tail -N` のような **pipe 終端コマンドの exit (= 0) でマスクされる**。 結果、 watch が「成功」を返したのに実は workflow が失敗していた、 という誤検出が起きる。

#### 安全な取得パターン

```bash
# NG: tail の 0 で gh の非ゼロが潰れる
gh run watch <run-id> --exit-status 2>&1 | tail -10

# OK: ファイル経由で gh の exit を直接保存
gh run watch <run-id> --exit-status > /tmp/watch.log 2>&1
echo "EXIT=$?" > /tmp/watch-exit.txt
```

または `set -o pipefail` を有効化した shell で動かす。

#### 適用範囲

CI / release / 任意の long-running command を `gh run watch` 等で監視する場合、 **exit code を後で参照する用途では pipe 終端を避ける**。 ログを切り詰めたい場合は file 経由 + `tail` を別 step で行う。

### workspace version bump は全 path 依存 location も同時に

workspace で複数 crate を持つ project では、 workspace version の bump だけでなく **`path = "..."` で繋がれた依存の `version = ` も同時に更新**する。 path 依存は local では即解決するが、 `cargo publish` 時に **crates.io 上の version 要件として検証**されるので、 整合性が崩れると release 失敗の原因になる。

#### 更新対象の典型例 (workspace project)

| location | 例 |
|---|---|
| `Cargo.toml` の `[workspace.package] version` | `version = "0.11.1"` |
| 親 `Cargo.toml` の strict pin path 依存 (derive 等) | `... version = "=0.11.1"` |
| 子 crate の workspace 内 path 依存 (`*/Cargo.toml`) | `... version = "0.11.1"` |

club-kdl v0.11.1 では 5 箇所が対象だった。 release PR 作成時に grep で全件洗うと漏れ防止になる:

```bash
rg '"=?0\.\d+\.\d+"|^version = ' Cargo.toml */Cargo.toml
```

### CHANGELOG.md は Keep a Changelog 形式 + compare links を維持

[Keep a Changelog](https://keepachangelog.com/) 形式を採用。 各 version の section と **ファイル末尾の compare link を両方同時に**維持する。

#### 構造

```markdown
# Changelog

## [Unreleased]

## [X.Y.Z] - YYYY-MM-DD

### 追加
- ...

### 変更
- ...

### 修正
- ...

[Unreleased]: https://github.com/<org>/<repo>/compare/vX.Y.Z...HEAD
[X.Y.Z]: https://github.com/<org>/<repo>/compare/vX.Y.W...vX.Y.Z
[X.Y.W]: https://github.com/<org>/<repo>/compare/vX.Y.V...vX.Y.W
```

#### 教訓 — 過去の link 欠落

club-kdl 0.8.0 / 0.9.1 release で **compare link 欠落** が複数回発生。 release のたびに **(a) [Unreleased] の compare base 更新 + (b) 新 version の compare link 追加** をセットで行う。 release PR で diff を見れば人間でも catch しやすい。

### release tag は annotated (`git tag -a`) で打つ

リリース tag は **annotated tag** (`git tag -a vX.Y.Z -m "..."`) で作る。 lightweight tag (`git tag vX.Y.Z`) は message を持たず、 `git describe` の出力も安定しないので避ける。

```bash
git tag -a v0.11.1 -m "v0.11.1 — release.yml publish 順 fix"
git push origin v0.11.1
```

`release.yml` の tag push trigger 自体は lightweight でも反応するが、 `git show <tag>` で release context が即読める価値がある。

---

## メモリ運用 / SDG

### Spec → Design → 実装 の三段 (SDG)

**非自明な変更**は creo-memories に **SDG 三段**で trace する:

1. **spec memory** — `category: spec`、 Why & What を書く。 不変の要件
2. **design memory** — `category: design`、 spec から `derivedFrom`、 How を書く。 設計判断
3. **実装** — design memory を **living doc として annotate で改訂** しつつコードに落とす

#### なぜ

- session を跨いだ trace を作る (誰が・いつ・なぜ・どう決めたか)
- handoff の受け手が **memory 1 つから context を catch up** できる
- 設計改訂時の **why が code commit に残らず失われる**のを防ぐ (annotate で trace を残す)

#### 細則

- **spec**: 不変の要件 (What & Why)。 実装中に判断が変わったら本文を直さず annotate で記録
- **design**: How の判断列。 実装中に意思決定が変わったら annotate で改訂 (上書きではない、 trace を残す)
- **handoff memory**: 受け手 agent / session 用、 `status: active` で保持、 完了で `done` に遷移 + 完了 annotation を付ける
- **branch 名と link**: `mako/<memory-short-id>-<slug>` 形式で branch を切ると memory と branch が双方向参照可能 (Git/Commit section 参照)

---

> 規約が増えたら section を追加する。 各 section は背景 (なぜ) + 規約 (何を) + 例 / 適用ルールの順で書くと参照しやすい。
