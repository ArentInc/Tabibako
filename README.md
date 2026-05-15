# Tabibako（旅箱）

旅行を構造化データとして記録し、地図と写真で振り返るパーソナル旅行データベース。Anthropic 中途入社研修の自由課題として、Webアプリ開発未経験者がアジャイル的に「小さく動かしながら積み上げる」進め方で実装しています。

## 動作環境

- **ブラウザ**：Chrome デスクトップ前提（Edge / Firefox でも動く想定だが未検証）
- **プロトコル**：`file://` で直接開ける（サーバー不要・ビルド不要）
- **依存**：すべて CDN（Leaflet 1.9.4 / OpenStreetMap / Tabler Icons / Leaflet.heat）
- **ストレージ**：localStorage（旅行データ・設定）+ IndexedDB（写真）

## 起動方法

```
C:\Dev\Tabibako\index.html を Chrome で開く
```

エクスプローラからダブルクリック、または Chrome のアドレスバーに `file:///C:/Dev/Tabibako/index.html` と入力。

オフライン時は地図タイルとアイコンの CDN 読み込みが失敗するため、骨格と配色のみ表示されます。

## サンプルデータの投入

サンプルデータの**生成ロジックは `index.html` に埋め込まれています**（git 管理対象）が、localStorage / IndexedDB はブラウザサンドボックスのため自動投入はできません。各ユーザーが DevTools コンソールから明示的に実行する必要があります。

### 手順

1. Chrome で `index.html` を開く
2. **F12** で DevTools を開く → **Console** タブ
3. 以下を順に実行（各コマンドで Enter）：

```js
devLoadSamples()
```

→ 日本各地・海外を含む **12 件のサンプル旅行**（計 30 か所程度）を投入。値プール（移動手段・同行者・タグ）にも初期値が入る。

```js
devLoadSamplePhotos()
```

→ サンプル旅行の各カードに **1〜2 枚の写真**を添付（Picsum 経由）。オフライン時は SVG プレースホルダーにフォールバック。`devLoadSamples` で写真なしの旅行のみが対象。

### サンプルデータの内容

| ヘルパ | 投入内容 | 永続化先 |
|---|---|---|
| `devLoadSamples()` | 12 旅行 / 計 30 か所程度 / 移動手段・同行者・タグの初期値 | localStorage |
| `devLoadSamplePhotos()` | 各旅行に 1〜2 枚（Picsum ランダム実写、`trip.name + index` を seed） | IndexedDB |

データ定義は `index.html` の `window.devLoadSamples` 関数（3744 行付近）にハードコードされています。新しい旅行を追加したい場合はこの関数を編集してください。

### サンプルデータのリセット

```js
devClearAll()
```

→ すべてのデータ（旅行・値プール・写真・絞り込み/ソート状態・パネル幅・期間絞り込み）を削除して F5 リロード。

### サンプルデータ再現の確実性

同じユーザーが同じ環境で `devLoadSamples()` → `devLoadSamplePhotos()` を実行した場合、以下は **再現します**：

- 12 旅行の内容（旅行名・日付・座標・訪問場所・タグ等）：完全同一
- 各旅行に添付される写真：seed が `${trip.name}-${index}` で deterministic なため、Picsum から同じ写真が返ってくる

以下は **再現しません**：

- `trip.id`（`crypto.randomUUID()` 由来）：実行ごとにランダム
- `photo.id`（同上）：実行ごとにランダム
- 写真添付枚数（1〜2 枚）：`Math.random()` 由来

## ドキュメント構成

| ファイル | 内容 |
|---|---|
| [`README.md`](./README.md) | 本ファイル。起動方法とサンプルデータ手順 |
| [`CLAUDE.md`](./CLAUDE.md) | Claude Code 向けプロジェクト規約・アーキテクチャ・YAGNI 方針 |
| [`SPRINT_LOG.md`](./SPRINT_LOG.md) | Sprint 1〜6 の実装内容・設計判断・残課題・レビュー指摘の履歴 |
| [`指示書/tabibako_spec.md`](./指示書/tabibako_spec.md) | 仕様書（データ定義・機能一覧・ユーザーストーリー） |
| [`指示書/tabibako_claude_code_prompt.md`](./指示書/tabibako_claude_code_prompt.md) | Claude Code 向け指示プロンプト原文 |
| [`指示書/tabibako_mockup_v6.html`](./指示書/tabibako_mockup_v6.html) | UI モック（Sprint 1〜3 では「正」扱い、Sprint 4 以降は乖離が大きい参考資料） |

## アーキテクチャ概要

- **単一ファイル構成**：HTML / CSS / JavaScript はすべて `index.html` 1 ファイル内（YAGNI による意図的選択）
- **vanilla HTML/CSS/JS**：React / TypeScript / ビルドツール不使用
- **3 カラム CSS Grid**：左 384px（初期値）/ 中央 1fr / 右 390px（初期値）。境界ドラッグで幅可変
- **データ**：localStorage に旅行データ・設定・絞り込み状態、IndexedDB に写真
- **デザイントークン**：`:root` の CSS 変数（色・余白・角丸）

詳細は `CLAUDE.md` を参照。

## 主要機能（実装済み）

- 旅行登録（モーダル + 地図クリック / 住所検索による訪問場所追加）
- 訪問場所・宿泊所のピン表示（選択中・非選択で配色変化）
- 写真添付（IndexedDB、EXIF 方向反映、1600px リサイズ、JPEG 0.80）
- マルチセレクト UI（移動手段・同行者・タグ、値プール独立管理）
- 編集 / 削除
- 絞り込み（移動手段・同行者・タグ・期間）
- ソート（日程新しい順 / 古い順 / 旅行名順）
- ヒートマップ（Leaflet.heat、選択時は個別ピン表示）
- 訪問地域自動判定（日本国内＝都道府県・海外＝国名）
- 左右パネル幅のドラッグリサイズ（キーボード操作対応）
- 旅行カードのサムネ表示
