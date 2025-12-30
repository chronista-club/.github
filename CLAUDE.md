# Chronista Club - Organization .github Repository

> **組織全体に共通設定を提供する特殊リポジトリ**

## このリポジトリの役割

GitHubの `.github` リポジトリは、組織（chronista-club）内の全リポジトリに対して共通の設定やテンプレートを提供する特殊なリポジトリです。

### 提供機能

| ディレクトリ/ファイル | 説明 |
|---------------------|------|
| `.github/workflows/` | 組織内全リポジトリで共有されるワークフロー |
| `profile/README.md` | 組織のプロフィールページ（作成予定） |
| `ISSUE_TEMPLATE/` | 共通のIssueテンプレート（作成予定） |
| `PULL_REQUEST_TEMPLATE.md` | 共通のPRテンプレート（作成予定） |
| `FUNDING.yml` | スポンサーシップ設定（作成予定） |

## 現在のワークフロー

### add-to-project.yml

Issue/PRを自動的にChronista Projectに追加し、担当者を発行者に設定する。

**トリガー**:
- Issue作成時
- Pull Request作成時

**アクション**:
1. Chronista Project（projects/6）に自動追加
2. 発行者を担当者として自動アサイン

**必要なSecret**:
- `PROJECT_TOKEN`: プロジェクトへの書き込み権限を持つPAT

## 開発ガイドライン

### ワークフローの追加

1. `.github/workflows/` に新しいYAMLファイルを作成
2. `on:` でトリガー条件を定義
3. 組織内の全リポジトリに適用されることを意識して設計

### 注意事項

- このリポジトリの変更は組織内の全リポジトリに影響する
- ワークフローの変更は慎重にテストすること
- Secretは組織レベルで管理される

## 関連リンク

- [GitHub Docs: Creating a default community health file](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)
- [Chronista Project](https://github.com/orgs/chronista-club/projects/6)
