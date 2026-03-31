# GitHub Actions ワークフロー ドキュメント

このドキュメントでは、`.github/workflows/` に定義されている GitHub Actions ワークフローについて説明します。

---

## 1. Claude Code (`claude.yml`)

### 概要

イシューやプルリクエストのコメントで `@claude` とメンションすることで、Claude AI に作業を依頼できるワークフローです。

### トリガー条件

| イベント | 条件 |
|---|---|
| `issue_comment` | コメントが作成されたとき |
| `pull_request_review_comment` | PRレビューコメントが作成されたとき |
| `issues` | イシューが作成またはアサインされたとき |
| `pull_request_review` | PRレビューが提出されたとき |

いずれのイベントでも、コメントまたはイシュー本文に `@claude` が含まれている場合のみ実行されます。

### 必要な権限

| 権限 | レベル | 用途 |
|---|---|---|
| `contents` | `read` | リポジトリの内容を読み取る |
| `pull-requests` | `read` | PRの情報を読み取る |
| `issues` | `read` | イシューの情報を読み取る |
| `id-token` | `write` | 認証トークンの取得 |
| `actions` | `read` | CIの実行結果を読み取る |

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の認証トークン |

### 使い方

イシューやPRのコメントに `@claude` を含めてメッセージを送ることで、Claude に作業を依頼できます。

```
@claude このコードをリファクタリングしてください
@claude バグを修正してください
@claude テストを追加してください
```

### カスタマイズ

`claude_args` オプションを使用することで、Claude の動作をカスタマイズできます。

```yaml
claude_args: '--allowed-tools Bash(gh pr:*)'
```

---

## 2. Claude Code Review (`claude-code-review.yml`)

### 概要

プルリクエストが作成・更新された際に、Claude AI が自動的にコードレビューを行うワークフローです。

### トリガー条件

以下のプルリクエストイベントで実行されます：

| イベントタイプ | 説明 |
|---|---|
| `opened` | PRが新規作成されたとき |
| `synchronize` | PRに新しいコミットが追加されたとき |
| `ready_for_review` | ドラフトPRがレビュー可能になったとき |
| `reopened` | クローズされたPRが再オープンされたとき |

### 必要な権限

| 権限 | レベル | 用途 |
|---|---|---|
| `contents` | `read` | リポジトリの内容を読み取る |
| `pull-requests` | `read` | PRの情報を読み取る |
| `issues` | `read` | イシューの情報を読み取る |
| `id-token` | `write` | 認証トークンの取得 |

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の認証トークン |

### 動作内容

1. リポジトリをチェックアウト
2. `claude-code-plugins` から `code-review` プラグインをロード
3. 対象PRに対して `/code-review:code-review` コマンドを実行
4. レビュー結果がPRのコメントに投稿される

### カスタマイズ

#### 特定のファイルのみレビュー対象にする

```yaml
on:
  pull_request:
    paths:
      - "src/**/*.ts"
      - "src/**/*.tsx"
```

#### 特定のユーザーのPRのみレビュー対象にする

```yaml
jobs:
  claude-review:
    if: |
      github.event.pull_request.user.login == 'external-contributor' ||
      github.event.pull_request.author_association == 'FIRST_TIME_CONTRIBUTOR'
```

---

## セットアップ

両ワークフローを使用するには、リポジトリに以下のシークレットを設定する必要があります。

1. GitHub リポジトリの **Settings** > **Secrets and variables** > **Actions** を開く
2. `CLAUDE_CODE_OAUTH_TOKEN` を追加する

Claude Code の OAuth トークンの取得方法については、[Claude Code ドキュメント](https://claude.ai/code) を参照してください。
