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

---

> 規約が増えたら section を追加する。 各 section は背景 (なぜ) + 規約 (何を) + 例 / 適用ルールの順で書くと参照しやすい。
