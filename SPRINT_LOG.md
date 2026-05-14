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

## Sprint 2：旅行データの登録（完了：2026-05-14）

### 実装内容

- **JS 構造化**：Sprint 1 で `const map` がグローバル汚染していたのを `App` 名前空間 + IIFE に集約。`state` / `map` / `el`（DOM キャッシュ）/ `storage` / `api` 系メソッドを 1 オブジェクトに整理
- **旅行登録モーダル**：中央オーバーレイ型。Esc / 背景クリック / × ボタン / キャンセルボタンで閉じる
- **入力フォーム**：旅行名（必須）、開始日・終了日（必須、HTML5 `<input type="date">`）、訪問場所（必須、1 件以上）
- **訪問場所の追加方法 2 系統**
  - **住所検索**：Nominatim `/search` API + 500ms debounce + 1 req/sec throttle。検索結果ドロップダウンから選択
  - **地図クリック**：「地図から選ぶ」ボタンで「ピックモード」に切替。モーダルが右下に縮小し背景透過、地図クロスヘアカーソル、地図クリックで Nominatim `/reverse` で名称取得 → 追加 → モーダル復帰
- **入力中の視覚フィードバック**：draft.places を地図上に L.marker で表示。保存・キャンセルで除去
- **localStorage 永続化**：キー `tabibako_trips` に JSON で保存。reload しても残る
- **左パネル旅行カード描画**：旅行名 + 開始日〜終了日 + 「訪問場所 N か所」。0 件時は空メッセージに戻る
- **バリデーション**：旅行名 / 日付 / 開始日 ≤ 終了日 / 訪問場所 1 件以上。最初に検出したエラーをモーダル下部に赤バナーで表示
- **XSS 対策**：DOM への文字列挿入はすべて `App.escape()` 経由

### 設計上の決定事項

| 決定 | 理由 |
|------|------|
| モーダル + 中央オーバーレイ採用 | ユーザー選択（指示書は「モーダルまたはスライドパネル」） |
| 地図クリック登録のための「ピックモード」設計 | モーダルが中央配置だと地図を隠してしまうため、モーダルを右下に縮小し背景透過にする独自の二段階モードを採用 |
| 訪問場所名は Nominatim 逆引きで自動取得 | ユーザー選択。`display_name` を full string で保存（Sprint 5 編集機能で短縮可能） |
| 旅行カードに「訪問場所 N か所」を表示 | 訪問都道府県自動判定（Sprint 5）までの暫定情報。情報量を維持 |
| Nominatim throttle を 1 req/sec で実装 | 公式利用規約（[osmfoundation policies](https://operations.osmfoundation.org/policies/nominatim/)）に従う |
| 旅行カードに `cursor: pointer` / `:hover` / `selected` スタイルを付けていない | Sprint 3 のクリック選択実装と同時に追加する。Sprint 2 では描画だけに責務を絞る |
| データスキーマは Sprint 4/5 用フィールド（accommodation/transport/companions/tags/memo/photos/prefectures）も含めて空で初期化 | Sprint 4 以降で旧データに後付けせず済むよう先行確保。配列は `[]`、文字列は `""` |

### 残課題（Sprint 3 以降）

#### Sprint 3 でやること
- 旅行カードクリックで選択状態 → 青枠ハイライト
- 選択中の旅行の訪問場所を地図ピンで表示
- ピンクリックで訪問場所名のポップアップ
- 右パネルに選択旅行の詳細表示

#### Sprint 4 以降に持ち越し（未着手）
- 移動手段・同行者・タグのマルチセレクト UI と値プール管理
- 宿泊所・メモのテキスト入力
- 写真の DataURL 添付
- 編集・削除フロー
- 絞り込み・ソート
- 都道府県自動判定
- ヒートマップ・オートズーム

### 既知の留意点

- **Nominatim CDN は API キー不要だが利用規約あり**：個人利用範囲なら問題ないが、商用化や複数端末同期に進む際は要再確認
- **`crypto.randomUUID()` の互換性**：Chrome / Edge / Firefox / Safari 全モダンブラウザ対応。IE 非対応（プロジェクト方針上 Chrome 前提なので問題なし）
- **ピックモード中のフォーム操作**：モーダル右下縮小中も入力欄は使えるが、検索ドロップダウンが画面外に出る可能性あり。実用上の併用は稀なので未対応
- **地図ピン（draft）の見た目**：Sprint 3 で本実装するピンとは別ロジックなので、Sprint 3 着手時に統合する余地あり
- **保存後のピン表示は Sprint 3 で実装**：Sprint 2 ではモーダル編集中（draft）のみピン表示。保存後は仕様通りピンが消え、Sprint 3 で「旅行カード選択 → ピン表示」のロジックが入る

### レビュー指摘と修正（Sprint 2、2026-05-14）

| 指摘 | 原因 | 修正 |
|------|------|------|
| ピンが地図上に表示されない（住所検索選択時・地図クリック時の両方） | `L.marker()` が読み込もうとする標準マーカー画像（`marker-icon.png` 等）が、`file://` + CDN ロードの組み合わせで CORS 制約により path 自動検出に失敗。画像 0 バイトでマーカー実体が見えなくなっていた | 標準マーカーを使わず、`L.divIcon` + Tabler Icons の `ti-map-pin-filled` でカスタムピンを実装。外部画像 CDN への依存を排除し、プロジェクトのアイコンシステムと統一。CSS `.tabibako-pin` で `.leaflet-div-icon` の白背景を打ち消し |
| ピン数バッジ「○か所」が常に 0 のまま動かない | `renderDraftPins` 時にバッジ更新ロジックが無かった。Sprint 2 完了条件に含まれるべき UI 連動を見落とし | `<span id="pin-count-value">` を分離し、`renderPinCount()` メソッドを追加。`renderDraftPins()` から呼ぶことで draft 追加・削除・モーダル開閉に追従。Sprint 3 で「選択中の旅行のピン」表示時にも同じメソッドを再利用できる構造 |
| `_throttle` の競合：並行呼び出し（search 中に reverse が走る等）で両方が 0 待機で発火しうる | `_lastApiRequestAt` の参照と更新が atomic でなく、複数の await が同じ値を見てしまう | `_throttleChain: Promise.resolve()` を導入し、各呼び出しを `.then()` で直列接続。チェーンの拒否は `.catch(() => {})` で握り潰して後続を守る |
| `searchAddress` のレスポンス競合：debounce + throttle の合間に入力が変わると古い結果が UI に出る可能性 | レスポンス到着時に「この結果はまだ最新か」をチェックしていなかった | `_searchRequestId` カウンタを導入し、呼び出し開始時に `myRequestId = ++this._searchRequestId` で予約。throttle 後・レスポンス後の各時点で ID 一致を確認、不一致なら描画スキップ。空入力時もカウンタをインクリメントして pending 検索を無効化 |
| CSS の生色コード 5 種（`#d9e9f7` / `#c53030` / `#fef2f2` / `#fecaca` / `#8ab`）が CLAUDE.md 方針違反 | Sprint 2 で `.btn-primary:hover` とエラーバナーを新規追加した際、対応するトークンを定義せずに直書きした | `:root` に `--color-background-info-hover` / `--color-background-error` / `--color-border-error` / `--color-text-error` を追加。`--color-text-photo-placeholder` も 3 桁 `#8ab` から 6 桁 `#88aabb` に統一。該当 4 箇所を `var(...)` 参照に置換 |

### Sprint 2 動作確認用チェックリスト

- [ ] ＋ボタンでモーダルが開く
- [ ] 旅行名・開始日・終了日が入力できる
- [ ] 住所検索で候補が表示され、クリックで追加できる
- [ ] 「地図から選ぶ」でピックモードに入り、地図クリックで場所が追加される（モーダル右下に縮小）
- [ ] 追加した場所が地図上に青ピンで表示される
- [ ] 場所の「×」で削除できる（ピンも消える）
- [ ] 未入力で保存するとエラーバナーが出る
- [ ] 保存すると左パネルに旅行カードが表示される
- [ ] ブラウザリロード後も旅行カードが残っている
- [ ] Esc キーでピックモード / モーダルが順に閉じる

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
