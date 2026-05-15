# 制作会社でのGitLabの運用方法

## GitLabとは

- GitベースのDevOpsプラットフォーム（リポジトリ管理、CI/CD、イシュー管理を一元化）
- GitHubと同様のWebインターフェースでリポジトリを操作できる
- **セルフホスト（オンプレミス）** での運用が可能で、社内サーバーに設置できる
- 制作会社ではクライアントの機密コードを社内に閉じて管理できる点が採用理由になりやすい
- 無料のCommunity Edition（CE）でも基本的なGit運用に必要な機能が揃っている

> 参照: [GitLab公式ドキュメント](https://docs.gitlab.com/)

## 制作会社でGitLabを使う理由

| 観点 | GitLab | GitHub |
|---|---|---|
| ホスティング | セルフホスト可能（社内サーバー） | クラウド中心（GitHub Enterprise Serverでオンプレ可能だが高額） |
| コスト | CE（無料版）で基本機能を網羅 | Team/Enterpriseプランは有料 |
| クライアント案件の管理 | 社内ネットワーク内に閉じて運用可能 | クラウド上にコードを置く契約上の制約が生じる場合がある |
| CI/CD | GitLab CI/CDが組み込み済み | GitHub Actionsで別途設定 |
| プロジェクト管理 | イシュー、マイルストーン、ボードが統合 | Issues、Projectsで対応可能 |

- クライアントとのNDA（秘密保持契約）でコードの外部ホスティングが禁止されるケースがある
- 社内サーバーにGitLabを立てれば、コードが外部に出ないため契約上の問題を回避できる

> 参照: [GitLab vs GitHub](https://about.gitlab.com/competition/github/)

## プロジェクト（リポジトリ）の構成

### グループ（Namespace）の設計

```
GitLab
├── client-a/               ← クライアントA用グループ
│   ├── corporate-site      ← コーポレートサイト
│   ├── ec-site             ← ECサイト
│   └── lp-campaign-2025    ← 2025年キャンペーンLP
├── client-b/               ← クライアントB用グループ
│   ├── brand-site
│   └── recruitment-site
├── internal/               ← 社内ツール・テンプレート
│   ├── starter-template
│   ├── coding-guidelines
│   └── design-system
└── sandbox/                ← 検証・学習用
    └── tech-spike
```

- **クライアント単位でグループを作成**するのが制作会社での基本パターン
- グループごとにメンバーのアクセス権限を設定できる
- 社内共通のテンプレートやガイドラインは `internal` グループにまとめる
- 案件終了後もアーカイブとしてリポジトリを残しておくと保守対応時に役立つ

### プロジェクトの命名規則

```
良い例:
client-name/project-name     ← ケバブケース
client-a/corporate-site
client-b/ec-renewal-2025

悪い例:
ClientA/CorporateSite        ← URLが読みにくくなる
client_a/corporate_site      ← GitLabのURLでは - が標準
```

- ケバブケース（`kebab-case`）で統一するとURLが読みやすい
- 年度やリニューアルなどの文脈がある場合はサフィックスに付与する

> 参照: [GitLab — Groups](https://docs.gitlab.com/ee/user/group/)

## ブランチ戦略

### 制作会社の案件に適したブランチモデル

```
main（本番環境）
 ├── staging（ステージング / クライアント確認環境）
 │   ├── feature/top-page         ← トップページ実装
 │   ├── feature/about-section    ← Aboutセクション実装
 │   ├── feature/contact-form     ← お問い合わせフォーム実装
 │   └── fix/header-responsive    ← ヘッダーのレスポンシブ修正
 └── hotfix/security-patch        ← 本番の緊急修正
```

| ブランチ | 役割 | デプロイ先 |
|---|---|---|
| `main` | 本番環境のコード | 本番サーバー |
| `staging` | クライアント確認用 | ステージングサーバー |
| `feature/*` | 機能・ページ単位の開発 | ローカル / プレビュー |
| `fix/*` | バグ修正 | ローカル / プレビュー |
| `hotfix/*` | 本番の緊急修正 | 本番サーバー |

### ブランチの保護設定

```
Settings > Repository > Protected Branches

main:
  - Allowed to merge: Maintainers
  - Allowed to push: No one（直接pushを禁止）

staging:
  - Allowed to merge: Developers + Maintainers
  - Allowed to push: No one
```

- `main` への直接pushを禁止し、マージリクエスト経由のみに制限する
- `staging` もマージリクエスト経由が望ましいが、小規模案件ではpush許可にする場合もある
- ブランチ保護によりレビューなしのコードが本番に入ることを防ぐ

> 参照: [GitLab — Protected Branches](https://docs.gitlab.com/ee/user/project/protected_branches.html)

## マージリクエスト（MR）の運用

### GitHubでいうプルリクエスト（PR）に相当

```
開発の流れ:
1. staging から feature/xxx ブランチを切る
2. ローカルで開発・コミット
3. feature/xxx を push する
4. GitLab上でマージリクエスト（MR）を作成する
5. レビュアーがコードを確認する
6. 承認後、staging にマージする
7. ステージング環境でクライアントが確認する
8. 確認OK後、staging を main にマージして本番デプロイ
```

### MRのテンプレート

```markdown
<!-- .gitlab/merge_request_templates/Default.md -->

## 概要
<!-- 何を実装・修正したか -->

## 対応ページ / セクション
<!-- 対象のページやセクション名 -->

## 確認方法
<!-- レビュアーがどう確認すればよいか -->
- [ ] ローカルで動作確認済み
- [ ] レスポンシブ確認済み（SP / Tablet / PC）
- [ ] ブラウザチェック済み（Chrome / Safari / Edge）

## スクリーンショット
<!-- 見た目の変更がある場合は貼り付ける -->

## 関連イシュー
<!-- closes #123 のように書くとマージ時にイシューが自動で閉じる -->
```

- `.gitlab/merge_request_templates/` にテンプレートを置くとMR作成時に自動適用される
- 制作会社ではレスポンシブ確認やブラウザチェックの項目を入れておくと漏れを防げる

### レビューの観点（制作案件向け）

- **表示崩れ**: 主要ブラウザ・デバイスで見た目が意図通りか
- **レスポンシブ**: ブレイクポイントごとにレイアウトが適切か
- **画像**: alt属性、ファイルサイズ、パスの正しさ
- **リンク**: リンク切れ、target属性の適切さ
- **コーディング規約**: プロジェクトのガイドラインに沿っているか

> 参照: [GitLab — Merge Requests](https://docs.gitlab.com/ee/user/project/merge_requests/)

## イシュー管理

### 制作案件でのイシューの使い方

```
イシューの粒度:
✓ ページ単位 — 「トップページの実装」
✓ セクション単位 — 「Aboutセクションのコーディング」
✓ 修正指示単位 — 「SPでヘッダーメニューが開かない」

✗ 大きすぎる — 「サイト全体のコーディング」
✗ 細かすぎる — 「h2のfont-sizeを24pxに変更」
```

### ラベルの設計

```
種類ラベル:
  📄 page        — ページ実装
  🔧 fix         — バグ修正
  🎨 design      — デザイン調整
  📱 responsive  — レスポンシブ対応
  ♿ a11y        — アクセシビリティ対応

状態ラベル:
  🔵 To Do       — 未着手
  🟡 In Progress — 作業中
  🟢 Review      — レビュー待ち
  🔴 FB          — クライアントフィードバック（修正指示）

優先度ラベル:
  🔺 high        — 優先度高
  ▶ medium      — 通常
  🔽 low         — 優先度低
```

### マイルストーンの活用

```
マイルストーン例:
  v1.0 初回公開         — 2025/04/01
  v1.1 フィードバック対応 — 2025/04/15
  v1.2 追加ページ       — 2025/05/01
```

- マイルストーンに期日を設定し、イシューを紐付ける
- 進捗率がマイルストーン画面で自動計算される
- クライアントへの進捗報告に使える

> 参照: [GitLab — Issues](https://docs.gitlab.com/ee/user/project/issues/)

## イシューボード

```
┌─────────┬─────────┬─────────┬─────────┐
│ To Do   │ Doing   │ Review  │  Done   │
├─────────┼─────────┼─────────┼─────────┤
│ #5 FAQ  │ #3 Top  │ #1 About│         │
│ #6 News │ #4 Form │         │ #2 MV   │
│         │         │         │         │
└─────────┴─────────┴─────────┴─────────┘
```

- GitLabのイシューボード機能でカンバン形式のタスク管理が可能
- ラベルに対応したカラムでイシューをドラッグ＆ドロップで移動できる
- チーム全体の進捗を視覚的に把握できる

> 参照: [GitLab — Issue Boards](https://docs.gitlab.com/ee/user/project/issue_board.html)

## アクセス権限の管理

### ロール（役割）

| ロール | 権限 | 制作会社での想定 |
|---|---|---|
| Owner | グループ・プロジェクトの全権限 | 管理者、テックリード |
| Maintainer | 設定変更、MRの承認・マージ | ディレクター、リードエンジニア |
| Developer | コードのpush、MR作成 | コーダー、フロントエンドエンジニア |
| Reporter | イシューの作成・コメント、コード閲覧 | デザイナー、PM |
| Guest | イシューの閲覧・コメントのみ | クライアント（限定的に招待する場合） |

### クライアントにアクセス権を付与する場合

```
クライアントに見せたいもの:
  ✓ ステージング環境のURL
  ✓ イシューでの修正指示・コメント

クライアントに見せたくないもの:
  ✗ ソースコード
  ✗ 社内のコミュニケーション（コードレビューのやり取り）
```

- クライアントを `Guest` ロールで招待すると、イシューの作成・コメントだけ可能
- コードを見せたくない場合は GitLab のプロジェクトに招待せず、**イシュー管理は別ツール**（Backlog、Notion等）を使う選択肢もある
- 機密性が高い案件ではプロジェクトの可視性を `Private` に設定する

> 参照: [GitLab — Permissions and roles](https://docs.gitlab.com/ee/user/permissions.html)

## GitLab CI/CDの基本活用

### 制作案件で使うCI/CDパイプライン

```yaml
# .gitlab-ci.yml
stages:
  - lint
  - build
  - deploy

# コードのチェック
lint:
  stage: lint
  image: node:20
  script:
    - npm ci
    - npm run lint
  only:
    - merge_requests

# ビルド
build:
  stage: build
  image: node:20
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
  only:
    - staging
    - main

# ステージングデプロイ
deploy_staging:
  stage: deploy
  script:
    - rsync -avz --delete dist/ $STAGING_USER@$STAGING_HOST:$STAGING_PATH
  only:
    - staging
  environment:
    name: staging
    url: https://staging.example.com

# 本番デプロイ
deploy_production:
  stage: deploy
  script:
    - rsync -avz --delete dist/ $PROD_USER@$PROD_HOST:$PROD_PATH
  only:
    - main
  environment:
    name: production
    url: https://www.example.com
  when: manual  # 手動承認で本番デプロイ
```

### よく使うCI/CDのステージ

| ステージ | 目的 | 制作案件での活用 |
|---|---|---|
| `lint` | コードの静的解析 | ESLint、Stylelintでコーディング規約チェック |
| `build` | ビルド・コンパイル | Astro、Next.jsなどのビルド |
| `deploy` | デプロイ | rsync、FTP、ホスティングサービスへのデプロイ |
| `test` | テスト | Lighthouseスコアチェック、リンク切れ検出 |

### 環境変数の管理

```
Settings > CI/CD > Variables

STAGING_HOST=staging.example.com
STAGING_USER=deploy
STAGING_PATH=/var/www/staging
PROD_HOST=www.example.com
PROD_USER=deploy
PROD_PATH=/var/www/production
SSH_PRIVATE_KEY=（秘密鍵）

※ Protected: mainとstagingブランチのジョブのみ使用可能にする
※ Masked: ジョブのログに値が表示されないようにする
```

- サーバーの接続情報や秘密鍵はGitLabの環境変数に保存する（リポジトリにコミットしない）
- `Protected` フラグを有効にすると保護ブランチのジョブでのみ使用可能になる

> 参照: [GitLab CI/CD](https://docs.gitlab.com/ee/ci/)

## コミットメッセージの規約

### 制作案件向けのシンプルな規約

```
良い例:
feat: トップページのメインビジュアルを実装
fix: SPでハンバーガーメニューが開かない問題を修正
style: Aboutセクションの余白を調整
chore: パッケージを更新

悪い例:
修正                      ← 何を修正したか分からない
WIP                       ← 何の作業中か分からない
asdf                      ← 意味不明
fix fix fix               ← 適当
```

### Conventional Commitsの最小セット

| プレフィックス | 用途 |
|---|---|
| `feat:` | 新機能・新ページの実装 |
| `fix:` | バグ修正 |
| `style:` | 見た目の調整（CSSの変更など） |
| `refactor:` | コードの整理（動作変更なし） |
| `chore:` | ビルド設定、パッケージ更新など |
| `docs:` | ドキュメントの追加・修正 |

- イシュー番号を含めると追跡が容易: `fix: SPのヘッダー崩れを修正 #42`
- `#42` と書くとGitLab上でイシューへの自動リンクが生成される

> 参照: [Conventional Commits](https://www.conventionalcommits.org/)

## 案件のライフサイクルと運用フロー

### プロジェクト開始から公開・保守まで

```
1. プロジェクト作成
   └── グループにリポジトリを作成、メンバーを追加、ブランチ保護を設定

2. 開発フェーズ
   └── feature/* ブランチで開発 → MRでレビュー → staging にマージ

3. クライアント確認
   └── ステージング環境をクライアントに共有 → フィードバックをイシューに起票

4. フィードバック対応
   └── fix/* ブランチで修正 → MRでレビュー → staging にマージ → 再確認

5. 本番公開
   └── staging を main にマージ → 本番デプロイ（CI/CDまたは手動）

6. 保守フェーズ
   └── 更新依頼をイシューに起票 → feature/* or fix/* で対応 → staging → main
```

### 案件終了後のリポジトリ管理

```
Settings > General > Advanced > Archive project

アーカイブすると:
  ✓ コードは閲覧可能
  ✓ イシューやMRの履歴も残る
  ✗ 新しいpushやMR作成はできなくなる
```

- 案件終了後はリポジトリをアーカイブして読み取り専用にする
- 保守契約がある場合はアーカイブせず `main` ブランチの保護だけ維持する
- 削除はせずに残しておくと、リニューアルや類似案件の参考にできる

## よくあるトラブルと対処

### マージコンフリクト

```bash
# stagingブランチの最新を取り込んでからMRを出す
git checkout feature/xxx
git fetch origin
git merge origin/staging
# コンフリクトが発生したらエディタで解消 → コミット → push
```

- featureブランチの作業が長引くとstagingとの差分が大きくなりコンフリクトが起きやすい
- **こまめにstagingの変更を取り込む**ことで予防できる
- GitLab上のMR画面でもコンフリクト解消のUIが提供されている

### 誤って `main` に直接pushした

```
防止策:
  Settings > Repository > Protected Branches で
  main への push を "No one" に設定する

もし起きてしまった場合:
  1. 落ち着いてチームに報告する
  2. git revert で該当コミットを打ち消すコミットを作る
  3. 直接 git reset --hard + force push はチームの作業を壊す恐れがあるため避ける
```

### 大きなファイルをコミットしてしまった

```
防止策:
  .gitignore に以下を含めておく:
    *.psd
    *.ai
    *.sketch
    *.fig（Figmaのローカルファイル）
    node_modules/
    dist/
    .DS_Store

対処:
  git rm --cached <file>  ← トラッキングを外す
  .gitignore に追記して再コミット
```

- デザインデータ（PSD、AI、Sketchなど）はGitに含めない
- 大きなファイルが必要な場合は Git LFS（Large File Storage）を検討する

> 参照: [GitLab — Git LFS](https://docs.gitlab.com/ee/topics/git/lfs/)

## .gitignore のテンプレート（制作案件向け）

```gitignore
# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/
*.swp

# Dependencies
node_modules/

# Build output
dist/
.output/
.next/
.astro/

# Environment
.env
.env.local
.env.production

# Design files
*.psd
*.ai
*.sketch

# Misc
*.log
```

## まとめ

- 制作会社でのGitLabは**クライアント単位のグループ**でリポジトリを整理するのが基本
- ブランチ戦略は `main`（本番）→ `staging`（確認用）→ `feature/*`（開発）の3層構成
- `main` への直接pushを禁止し、**マージリクエスト経由**でコードをマージする
- イシューとラベルでタスクを管理し、マイルストーンで進捗を追跡する
- GitLab CI/CDで `lint → build → deploy` を自動化できる
- 案件終了後はリポジトリをアーカイブして読み取り専用にする
- `.gitignore` でデザインデータやビルド成果物を管理対象外にする

## 参照元

- [GitLab公式ドキュメント](https://docs.gitlab.com/)
- [GitLab — Groups](https://docs.gitlab.com/ee/user/group/)
- [GitLab — Protected Branches](https://docs.gitlab.com/ee/user/project/protected_branches.html)
- [GitLab — Merge Requests](https://docs.gitlab.com/ee/user/project/merge_requests/)
- [GitLab — Issues](https://docs.gitlab.com/ee/user/project/issues/)
- [GitLab — Issue Boards](https://docs.gitlab.com/ee/user/project/issue_board.html)
- [GitLab — Permissions and roles](https://docs.gitlab.com/ee/user/permissions.html)
- [GitLab CI/CD](https://docs.gitlab.com/ee/ci/)
- [GitLab — Git LFS](https://docs.gitlab.com/ee/topics/git/lfs/)
- [Conventional Commits](https://www.conventionalcommits.org/)
