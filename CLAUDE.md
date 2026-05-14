# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

**Tabibako（旅箱）**：旅行を構造化データとして記録し、地図と写真で振り返るパーソナル旅行データベース。Anthropic 中途入社研修の自由課題として、Webアプリ開発未経験者がアジャイル的に「小さく動かしながら積み上げる」進め方で実装する。

研修課題上の制約：「数ショットで終わるような課題は避け、仕様をしっかり言語化して AI にコードを書かせるアプローチを意識する」。仕様書は `指示書/tabibako_spec.md`、Claude Code 向け指示プロンプト原文は `指示書/tabibako_claude_code_prompt.md` にある。

## アーキテクチャ

**単一ファイル構成。** HTML・CSS・JavaScript はすべて `index.html` 1 ファイル内に収める。サーバー不要・ビルド不要で `file://` プロトコルから Chrome で直接開いて動作する。

この構成は YAGNI 方針による意図的な選択（[SPRINT_LOG.md](./SPRINT_LOG.md) 参照）。Sprint が進んで肥大化してから分割を検討するため、現段階で勝手にモジュール分割・npm 化・ビルドツール導入をしないこと。

- 外部依存はすべて CDN（Leaflet 1.9.4 + OpenStreetMap、Tabler Icons webfont 3.5.0）。SRI ハッシュ付きで `unpkg` / `jsDelivr` から読み込み。バージョン変更時は `integrity` 値も更新する。
- データ保存は localStorage のみ。バックエンド／サーバーは存在しない。
- 技術スタックは vanilla HTML/CSS/JS。**React・TypeScript は使わない**（指示書で明示）。
- 住所検索は Nominatim API（Sprint 2 以降）。
- ヒートマップは Leaflet.heat プラグイン（Sprint 6 で導入予定）。

## デザインシステム

色・余白・角丸は `index.html` 冒頭の `:root` で CSS 変数として定義（モック v6 ベース）。新規スタイルを書くときは既存トークンを再利用し、生の色コードを直書きしない。

3カラム CSS Grid レイアウト：左 384px / 中央 1fr / 右 390px、高さ 100vh。**指示書原本は 256px / 260px だが、Sprint 1 レビューで「狭い」と判断し 1.5 倍に拡張**（SPRINT_LOG.md 参照）。トップバーは持たず、ヒートマップ切り替え・ピン数表示は地図上のオーバーレイで実装する。

アイコンは Tabler Icons の webfont 版を `<i class="ti ti-xxx">` で使用。

マルチセレクト型プロパティ（移動手段・同行者・タグ）は同じ入力 UI を共有するが **値プールは独立** して localStorage に保存する：
- `tabibako_transport_pool`（初期値：徒歩・自転車・車・電車・バス・飛行機・船）
- `tabibako_companions_pool`
- `tabibako_tags_pool`

カラー規約：移動手段は青系、同行者は緑系、タグは紫系でハイライト表示する。

## 開発ワークフロー

このプロジェクトは Sprint 1〜6 のアジャイル方式で進める。各 Sprint の目標・タスク・完了条件は `指示書/tabibako_claude_code_prompt.md` に定義済み。

### Sprint 進行時のルール

1. 新規 Sprint 着手前に **必ず `SPRINT_LOG.md` を読んで現状を把握する**。前 Sprint の決定事項・残課題・既知の留意点が記録されている。
2. Sprint 完了時に `SPRINT_LOG.md` の該当セクションへ以下を追記する：
   - 実装内容
   - 設計上の決定事項（理由付き）
   - 残課題（次 Sprint / 持ち越し）
   - 既知の留意点
3. Sprint 完了時、試行錯誤中のコメントアウトや未使用コードは削除してからログを書く（コンテキスト汚染回避）。
4. 仕様とモック画面（`指示書/tabibako_mockup_v6.html`）で齟齬がある場合は **モックを正** とする。

### 動作確認

ビルドコマンド・テストフレームワークは存在しない。動作確認は以下：

```
C:\Dev\Tabibako\index.html を Chrome で直接開く（file:// プロトコル）
```

UI 変更時は実際にブラウザで開いてゴールデンパスと主要なエッジケースを確認すること。型チェック・テストでは UI の正しさは検証できない。

## 環境上の留意点

- **作業ディレクトリは `C:\Dev\Tabibako\` 固定。** `C:\Users\...\Documents\` 配下は Windows Defender の Controlled Folder Access（CFA）によってファイル書き込みがブロックされる。`C:\Dev` は保護対象外。
- **オフライン環境では CDN タイル・アイコンが表示されない**（骨格・配色は表示される）。仕様上の前提として記録。
- シェルは PowerShell。コマンドは PowerShell 構文で書く。

## 仕様参照の優先順位

判断に迷ったときの参照優先順位：

1. `指示書/tabibako_mockup_v6.html`（UI モック）— 画面デザインの最終判断はここ
2. `指示書/tabibako_spec.md`（仕様書）— データ定義・機能一覧・ユーザーストーリー
3. `指示書/tabibako_claude_code_prompt.md`（指示プロンプト）— Sprint 計画・実装方針
4. `SPRINT_LOG.md`（開発ログ）— 過去の Sprint で確定した設計判断

## 進め方の前提

ユーザーは Webアプリ開発未経験（ウォーターフォール経験のみ）。大きな変更前は計画を提示し、エラー発生時は原因と修正方針を説明してから直すこと。「とりあえず動く」コードと「整理されたコード」を混在させない。
