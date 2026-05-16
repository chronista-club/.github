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
