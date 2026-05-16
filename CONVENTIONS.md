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

### full rename policy

公開 crate は **package 名だけでなく lib 名も** `club_` prefix に full rename する。

| | crates.io package | lib name (`use`) |
|--|--|--|
| ✅ 現行 (full rename) | `club-unison` | `club_unison` |
| ✗ 旧 (lib 名据置) | `club-unison` | `unison` |

経緯: 当初 (unison v0.5.0) は 「package 名のみ rename、 lib 名は据置」 だったが、 v0.6.0 で 「lib 名も rename」 へ方針変更。 理由は `club-kdl` 側 (lib `club_kdl`) との整合性。 ecosystem 全体で **package 名と lib 名を一致させる** ことを優先した。

### namespace 別命名 map

1 つの project の identifier を namespace 別に並べると以下になる (例: `nostos` project):

| namespace | prefix | 例 |
|---|---|---|
| GitHub org | (固定) | `chronista-club` |
| GitHub repo | あり | `club-nostos` |
| local checkout (`~/repos/...`) | あり | `club-nostos` |
| `[package].name` (Cargo.toml) | あり | `club-nostos` |
| `[lib].name` (import path) | あり | `club_nostos` |
| workspace subdir (`crates/...`) | **なし** | `nostos` |
| creo-memories atlas | **(例外)** | `Nostos Club` |
| project 通称 (informal) | なし | `nostos` |

現行ルールは **「原則すべての layer で `club-` prefix、 例外は 2 つ」**:

1. **workspace subdir** (`crates/nostos-core/` 等) ── repo 内部の code path に埋もれるため prefix なし
2. **creo-memories atlas** ── machine identifier ではなく human-facing な **display label**。 Title Case + space を許容する (例: `Nostos Club`)

> 補足: prefix の有無を 「その identifier が browsable な一覧 UI (crates.io / GitHub org page / Finder 等) に出るか」 で判別する設計案もあったが、 full rename policy 採用で 「一貫性のため原則全 prefix」 に整理された。 上記 2 例外はその残余。

### User-facing 使用例 (確定形)

```toml
[dependencies]
club-nostos = "0.1"
```

```rust
use club_nostos::Bracket;
```

```bash
cargo add club-nostos
git clone https://github.com/chronista-club/club-nostos.git
cd ~/repos/club-nostos
```

### 新規 crate を足すとき

`[package].name = "club-<name>"` / `[lib].name = "club_<name>"` で統一する。
