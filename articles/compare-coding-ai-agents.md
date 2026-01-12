---
title: "AI コーディングエージェント比較一覧表（2025年5月）"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ai駆動開発]
published: false
---

## AI コーディングエージェント比較一覧表（2025年5月）

### 基本情報比較

| 項目 | Devin (Cognition AI) | Codex (OpenAI) | Jules (Google) | Claude Code (Anthropic) | GitHub Copilot Coding Agent |
|------|---------------------|----------------|----------------|------------------------|---------------------------|
| **料金** | $20/月〜従量課金<br>• $20で9 ACU<br>• 1 ACU = 15分<br>• 実質$11/時間 | ChatGPT Pro/Team/Enterprise<br>• 現在無料<br>• 今後従量課金予定 | 無料（ベータ期間）<br>• 1日5タスクまで<br>• 将来有料化予定 | Max サブスクリプション<br>または API 従量課金<br>• トークン消費量多い | Pro+/Enterprise<br>• Free版あり（機能制限）<br>• Individual/Business |
| **対象ユーザー** | • 大規模リファクタリング<br>• エンタープライズ開発者 | • Cisco, Temporal, Superhuman<br>• ChatGPT ユーザー | • バグ修正に追われるチーム<br>• GitHub ユーザー | • ターミナル愛好者<br>• 上級開発者<br>• エンタープライズ | • テスト済みコードベース<br>• GitHub エコシステム利用者 |
| **エディタ統合** | • ブラウザベース<br>• GitHub 統合 | • ChatGPT インターフェース<br>• IDE 統合なし | • GitHub 直接統合<br>• ブラウザベース | • 任意の IDE ターミナル<br>• macOS/Linux/WSL | • github.com/Mobile/CLI<br>• VS Code/JetBrains/Eclipse/Xcode |

### 主要機能比較

| 機能 | Devin | Codex | Jules | Claude Code | GitHub Copilot Agent |
|------|-------|-------|-------|-------------|-------------------|
| **自律的動作** | ✅ 完全自律型 | ✅ ChatGPT 内で自律 | ✅ 非同期自律動作 | ✅ ターミナルで自律 | ✅ イシュー割当で自律 |
| **並列実行** | ✅ 複数 Devin 並列可 | ❌ | ✅ 複数タスク同時処理 | ✅ Unix ユーティリティ的 | ✅ GitHub Actions 利用 |
| **コードベース理解** | ✅ プロジェクト理解 | ⚠️ 限定的 | ✅ 全体コンテキスト理解 | ✅ 数秒でマッピング | ✅ リポジトリ全体理解 |
| **テスト実行** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **PR/コミット作成** | ✅ | ❌ | ✅ | ✅ GitHub/GitLab 対応 | ✅ ドラフト PR 作成 |
| **リアルタイム報告** | ✅ | ⚠️ 1-30分後 | ✅ | ✅ | ✅ セッションログ |
| **音声要約** | ❌ | ❌ | ✅ Codecast機能 | ❌ | ❌ |
| **マルチモーダル** | ❌ | ❌ プレビュー版 | ❌ | ❌ | ✅ スクリーンショット対応 |

### 特徴的な機能

| ツール | 独自の強み | 制限事項 |
|--------|-----------|----------|
| **Devin** | • インタラクティブプランニング<br>• VSCode ライクな UI<br>• Nubank で12倍効率改善実績 | • 複雑タスクで成功率低い（3/20）<br>• 以前の$500/月から値下げも依然高額 |
| **Codex** | • ChatGPT エコシステム統合<br>• 手軽に開始可能<br>• 大手企業が早期採用 | • 画像ベースタスク未対応<br>• データプライバシー懸念<br>• 独立開発環境なし |
| **Jules** | • Gemini 2.5 Pro 使用<br>• プライバシー重視設計<br>• 無料枠で試しやすい | • 1日5タスク制限<br>• ベータ版の制限 |
| **Claude Code** | • Unix 哲学的設計<br>• 高カスタマイズ性<br>• エンタープライズ対応 | • トークン消費量が非常に多い<br>• ターミナル必須 |
| **GitHub Copilot Agent** | • GitHub 完全統合<br>• エンタープライズ機能充実<br>• VS Code チーム推奨 | • 有料プラン必須<br>• GitHub 依存 |

### ユーザー評価サマリー

| ツール | ポジティブな評価 | ネガティブな評価 |
|--------|-----------------|----------------|
| **Devin** | Nubank「12倍の効率改善、20倍のコスト削減」 | 「20タスク中3つのみ成功」「不必要な抽象化」 |
| **Codex** | 「仕様を数分でプロダクションコードに」 | 「データプライバシーリスク」「プレビュー版の制限」 |
| **Jules** | 「高度なコーディング推論能力」 | （ベータ版のため評価少ない） |
| **Claude Code** | 「レガシー React リファクタで威力発揮」「Intercom で新規アプリ開発可能に」 | 「トークン使用量が息をのむほど多い」 |
| **GitHub Copilot Agent** | Carvana「仕様を数分でコードに変換」 | （新機能のため限定的） |

### 推奨用途マトリックス

| 用途 | 最適 | 次点 |
|------|------|------|
| **個人開発・初心者** | Jules（無料枠） | GitHub Copilot Free |
| **ターミナル派上級者** | Claude Code | - |
| **エンタープライズチーム** | GitHub Copilot Agent | Devin |
| **ChatGPT ユーザー** | Codex | - |
| **大規模リファクタリング** | Devin | Claude Code |
| **GitHub 中心の開発** | GitHub Copilot Agent | Jules |
| **コスト重視** | Jules（無料枠） | GitHub Copilot Free |
