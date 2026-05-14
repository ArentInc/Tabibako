# Tabibako 開発ログ

このファイルは Sprint をまたいで Claude Code が文脈を引き継ぐためのログです。
次の Sprint を開始する前にこのファイルを更新し、新しいセッションでは最初にこのファイルを読ませてください。

---

## Sprint 1：骨格と地図表示（完了：2026-05-14）

### 実装内容

- `index.html` を **1 ファイル構成** で作成（HTML / CSS / JavaScript を内包）
- CSS Grid で **3 カラムレイアウト**（左 256px / 中央 1fr / 右 260px、高さ 100vh）
- **Leaflet 1.9.4 + OpenStreetMap** で中央パネルに日本地図を表示
  - 初期ビュー：中心 `[36.5, 138.0]` / ズーム 5
  - ズーム・ドラッグ・スクロールホイール操作可
- **Tabler Icons (webfont 3.5.0)** を CDN 導入し、`<i class="ti ti-xxx">` でアイコン表示
- モック v6 の配色・余白・角丸を `:root` の CSS 変数として自前定義
- 仮置き DOM（CSS のみ完成・JS 未実装）
  - 左パネル：「Tabibako」テキスト + ソートボタン + ＋ボタン + 旅行一覧の空メッセージ
  - 中央パネル：左上「0か所」ピン数バッジ、右上「ヒートマップ」ボタン
  - 右パネル：写真ビューア領域、詳細空メッセージ、編集・削除ボタン（disabled）

### 設計上の決定事項

| 決定 | 理由 |
|------|------|
| 単一 `index.html` 構成 | 指示書プロンプトの方針通り。Sprint が進んで肥大化してから分割を検討（YAGNI） |
| 外部依存はすべて CDN | `file://` プロトコルで直接ブラウザから開ける。Node.js サーバー不要 |
| `:root` の CSS 変数を自前定義 | モック v6 が外部デザイントークン前提で書かれており、そのままだと色が出ないため |
| アイコンは Tabler Icons | モック v6 が `ti-arrows-sort` / `ti-plus` などを使用。Sprint 4 以降のアイコンもこれで賄える |
| ＋ボタン / ソートボタンは `<button>` 要素 | モックは `<div>` だが、後の Sprint で `disabled` 制御やキーボード操作を考えるなら `<button>` が適切 |
| プロジェクトフォルダを `C:\Dev\Tabibako\` に配置 | `C:\Users\...\Documents\` 配下は Windows Defender Controlled Folder Access (CFA) によって書き込みがブロックされていたため。`C:\Dev` は保護対象外で書き込み可能 |

### 残課題（Sprint 2 以降）

#### Sprint 2 でやること
- ＋ボタンの旅行登録モーダル / スライドパネル
- 旅行名・開始日・終了日の入力フォーム
- 訪問場所の住所検索（Nominatim API）
- 地図クリックでの訪問場所登録
- localStorage への保存ロジック（キー設計含む）
- 左パネルの旅行カード一覧描画

#### Sprint 3 以降に持ち越し
- 旅行カード選択 → 地図ピン表示 → 右パネル詳細表示の連携
- マルチセレクト UI（移動手段・同行者・タグ、値プール独立管理）
- 写真添付（DataURL 変換 + localStorage 保存）
- 写真ビューア（矢印切替・カウンター・object-fit:contain）
- 編集 / 削除フロー
- 絞り込み 3 セクション（移動手段・同行者・タグ）の独立 ON/OFF
- ソートドロップダウン（モックの sort-dropdown UI 相当）
- 訪問都道府県の自動判定（緯度経度 → 都道府県名）
- ヒートマップ（Leaflet.heat プラグイン）
- 旅行選択時の `fitBounds` オートズーム

### 既知の留意点

- **CFA（Controlled Folder Access）**：`Documents` 配下では Claude Code が書き込めない。プロジェクトは `C:\Dev\Tabibako\` で運用すること。`Documents` 側の元フォルダはコピー元として残してあるので、不要になったら手動で削除可。
- **オフライン時の挙動**：Leaflet / Tabler Icons / OSM タイルすべて CDN なので、オフライン環境ではタイルとアイコンが表示されない（骨格・配色は表示される）。
- **SRI ハッシュ**：Leaflet 1.9.4 の `integrity` 値は公式 README から取得済み。バージョン変更時は更新が必要。

### レビュー指摘と修正（2026-05-14）

ユーザー目視レビューで以下 2 件を検出・修正。

| 指摘 | 原因 | 修正 |
|------|------|------|
| 左パネル「Tabibako」横のアイコンが空欄表示 | Tabler Icons webfont の CDN パスが誤り。`@tabler/icons-webfont@3.5.0` ではアセットが `dist/` 配下に置かれており、旧パス `tabler-icons.min.css` は 404 になっていた | `href` を `dist/tabler-icons.min.css` に修正 |
| ズーム +/− ボタンがピン数バッジ「○か所」と重なる | Leaflet 既定の `zoomControl` 位置は `topleft`。`zoomControl: true` のままでは仕様（右下配置）と一致しない。CSS 側に `.leaflet-bottom.leaflet-right` のマージン調整はあったが、肝心の position 指定が抜けていた | `zoomControl: false` で標準コントロールを無効化し、`L.control.zoom({ position: 'bottomright' })` で右下に配置 |
| 左右パネルが狭く、コンテンツが窮屈に見える | 指示書原本の `256px / 260px` は FHD ディスプレイ前提の値で、横長ディスプレイ（実機：DAIV S5 想定）では中央地図が広すぎ、左右が狭く感じる | 1.5 倍に拡張：左 `384px` / 右 `390px`。今後の Sprint で左パネルに絞り込み 3 セクション、右パネルに各プロパティ表示が乗ることを見越した先行調整 |
| 地図 `<div>` の `role="application"` + `aria-label="OpenStreetMap"` が ARIA アンチパターン | `role="application"` はスクリーンリーダーを「フォーカスモード」に強制切り替えし、ブラウズモードのナビゲーション（矢印キー・見出しジャンプ等）を全停止させる。Leaflet 自身が標準的な `<a role="button">` で a11y を提供しているため不要。さらに親 `<main aria-label="地図">` と `aria-label` が二重指定で読み上げが矛盾していた | `<div id="map">` から `role` と `aria-label` を両方削除。親 `<main>` のランドマークと Leaflet 内部の a11y に任せる |
| `.photo-viewer` の背景・文字色が CLAUDE.md 方針違反（生色コード `#c9d9e8` / `#8ab` を直書き） | 既存トークン（`--color-background-info` 等）と意味的にも色味的にも合致しないため、生コードを書かざるを得なかった状況 | プレースホルダー専用トークン `--color-background-photo-placeholder` / `--color-text-photo-placeholder` を `:root` に新設して参照。Sprint 4 で写真ビューア本実装する際、プレースホルダー処理と一緒にこのトークンの去就を再検討する |

---

## Sprint 2：旅行データの登録（未着手）

（着手時に追記）

---

## Sprint 3：地図ピン表示と詳細パネル（未着手）

（着手時に追記）

---

## Sprint 4：残りのプロパティ入力と写真（未着手）

（着手時に追記）

---

## Sprint 5：編集・削除・絞り込み・ソート（未着手）

（着手時に追記）

---

## Sprint 6（オプション）：ヒートマップ・オートズーム（未着手）

（着手時に追記）
