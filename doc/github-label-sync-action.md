# github-label-sync

GitHubリポジトリのラベルをYAMLファイルで定義し、同期するreusable workflow。

## 概要

このworkflowは[EndBug/label-sync](https://github.com/EndBug/label-sync)をラップし、複数のラベル定義ファイルを結合して適用できるようにしたものです。

## Inputs

| 名前 | デフォルト | 型 | 説明 |
|------|-----------|-----|------|
| `source_paths` | `.github/labels/labels.yaml` | string | `source_repository` の `.github/labels/` 配下にあるラベル定義ファイルパス（改行区切りで複数指定可能）。複数指定した場合は順番に結合される |
| `source_repository` | `yuzu441/workflows` | string | `source_paths` のパスを解決するリポジトリ |
| `allow_added_labels` | `false` | boolean | `true` にすると、定義ファイルに存在しないラベルをリポジトリから削除しない |

## Secrets

| 名前 | 必須 | 説明 |
|------|------|------|
| `gh_token` | 任意 | ラベルを同期するためのGitHubトークン。未指定の場合は `github.token` を使用。`source_repository` が別のプライベートリポジトリの場合は必須 |

## 利用例

### ベースラベルのみ

```yaml
jobs:
  sync:
    uses: yuzu441/workflows/.github/workflows/github-label-sync.yaml@main
    secrets: inherit
```

### ベース + claude用ラベル

```yaml
jobs:
  sync:
    uses: yuzu441/workflows/.github/workflows/github-label-sync.yaml@main
    with:
      source_paths: |
        .github/labels/labels.yaml
        .github/labels/claude.yaml
    secrets: inherit
```

### 呼び出し側の完全なワークフロー例

```yaml
name: github-label-sync

on:
  push:
    paths:
      - .github/workflows/github-label-sync.yaml
  workflow_dispatch:

permissions:
  contents: read
  issues: write

jobs:
  sync:
    uses: yuzu441/workflows/.github/workflows/github-label-sync.yaml@main
    with:
      source_paths: |
        .github/labels/labels.yaml
        .github/labels/claude.yaml
    secrets: inherit
```

## ラベルファイルの構成

`yuzu441/workflows` リポジトリで管理しているラベル定義ファイルは以下の通りです。

| ファイル | 内容 |
|---------|------|
| `labels.yaml` | ベースラベル（全リポジトリ共通） |
| `claude.yaml` | claude操作用ラベル（`claude:implement`, `claude:fix`） |

### ラベル定義フォーマット

```yaml
- name: ラベル名
  description: ラベルの説明
  color: "16進数カラーコード（#なし）"
```

### 新しいラベルセットを追加する

1. `.github/labels/` に新しいYAMLファイルを作成する
2. 呼び出し側の `source_paths` に追加する
