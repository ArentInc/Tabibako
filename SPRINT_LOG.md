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

## Sprint 3：地図ピン表示と詳細パネル（完了：2026-05-14）

### 実装内容

- **旅行カード選択 UI**
  - `.trip-card` に `cursor: pointer` / `:hover` / `.selected` スタイル追加（Sprint 2 で保留していたもの）
  - `.selected` は info 系トークン（青枠 + 薄青背景）で視覚化
- **App.state.selectedTripId** を追加（null 初期値、`crypto.randomUUID()` の文字列）
- **`App.selectTrip(tripId)`** メソッド：同じ ID 再クリックで **toggle 解除**
- **`App.getSelectedTrip()`** ヘルパ：state から実体を引く
- **trip-list へ event delegation** で click ハンドラを 1 つだけ設置（動的に追加されるカードに自動対応）
- **`App.selectedTripMarkers`** 配列：draft とは別レイヤーで管理。同時表示可能
- **`App.renderSelectedTripPins()`**：selectedTrip があれば places をマーカー化。Popup で名称表示。createPinIcon を再利用
- **`App.renderPinCount()` を draft + selected の合算に更新**：選択中の旅行のピンと、モーダル編集中の draft ピンを両方カウント
- **右パネル詳細レンダリング**：`detail-body` を空状態と選択状態で切替
  - 空状態：「旅行を選択すると、ここに詳細が表示されます。」
  - 選択状態：旅行名（タイトル）/ 日程 / 訪問場所リスト（アイコン + 名称）
- **CSS 追加**：`.detail-content` / `.detail-title` / `.detail-section` / `.detail-place-list` / `.detail-place-item`

### 設計上の決定事項

| 決定 | 理由 |
|------|------|
| 選択は **toggle 方式**（同じカードを再クリックで解除） | UX シンプル。明示的な「解除ボタン」を持たず、デザイン省スペース |
| `selectedTripMarkers` を `draftMarkers` と **別配列で管理** | モーダル編集中も選択中の旅行のピンを残せる。両方同時表示で「既存の旅と新しい旅の位置関係」が見える |
| ピン数バッジは **両者合算** | 「地図に表示中のピン」を一律にカウントする方針。selected のみ・draft のみといった条件分岐は将来 Sprint 6 のヒートマップ実装時に再検討 |
| 右パネル詳細は Sprint 3 範囲のフィールドのみ表示（旅行名 / 日程 / 訪問場所） | 訪問都道府県は Sprint 5、移動手段・同行者・宿泊所・タグ・メモ・写真は Sprint 4。各フィールドは Sprint 4 着手時に section として追加 |
| 編集・削除ボタンは引き続き `disabled` | Sprint 5 でハンドラを wire up する |
| trip-list を event delegation 化 | 動的描画されるカードに `addEventListener` を毎回張り直すパターンを回避（Sprint 2 で `place-list` / `search-results` でやっている即時再バインドの罠を回避） |

### 残課題（Sprint 4 以降）

#### Sprint 4 でやること
- 移動手段・同行者・タグのマルチセレクト UI（Issue #2 で仕様確定後）
- 宿泊所・メモのテキスト入力
- 写真添付（Issue #1 で戦略確定後）
- 右パネル詳細に上記フィールドのセクションを追加
- 写真ビューアの矢印切替・カウンター

#### Sprint 5 以降に持ち越し
- 編集ボタンから旅行データの編集
- 削除ボタン → 確認ダイアログ → 削除（削除時に selectedTripId のクリアも必要）
- 訪問都道府県の自動判定（緯度経度 → 都道府県名）
- 絞り込み 3 セクション・ソートドロップダウン
- ヒートマップ・fitBounds オートズーム（Sprint 6）

### 既知の留意点

- **localStorage の旧データ互換**：今後 Sprint 4 でフィールドを増やしたとき、Sprint 3 までに保存された trip データには新フィールドが空文字 or 空配列で残る。`createBlankTrip` で先行初期化済みなので問題なし
- **trip-list の再レンダリング**：選択状態が変わるたびに全カードを再描画している。trip 数が増えると無駄が出るが、Sprint 5 の絞り込み実装時に DOM diff の必要性を再評価する
- **fitBounds は未実装**：選択時に旅行のピンが全部画面内に収まる保証はない。Sprint 6 のオートズームで対応予定

### Sprint 3 動作確認用チェックリスト

- [ ] 旅行カードをクリック → 青枠でハイライト
- [ ] 選択中の旅行の訪問場所が地図にピン表示される
- [ ] ピンクリック → Popup で場所名表示
- [ ] 右パネルに旅行名・日程・訪問場所リストが表示される
- [ ] ピン数バッジが選択中のピン数を反映
- [ ] 同じカードを再クリック → 選択解除（カードのハイライト、地図ピン、右パネル詳細がすべて消える）
- [ ] 別のカードをクリック → 旧選択が解除され、新選択が反映される
- [ ] モーダルで新規旅行を登録中も、選択中の旅行のピンは残っている（両者合算でカウント表示）

### レビュー指摘と修正（Sprint 3、2026-05-14）

| 指摘 | 原因 | 修正 |
|------|------|------|
| 旅行カードがキーボード操作非対応（`<div>` + click のみ。Tab で到達できず、Enter/Space で選択できない） | `<div role="button">` を付けず素の div で実装していた。Sprint 1 で `role="application"` を a11y アンチパターンとして除去した経緯と整合性が取れていなかった | `<div>` に `role="button"` `tabindex="0"` `aria-pressed="<bool>"` を追加。trip-list の event delegation に keydown ハンドラを追加して Enter/Space で `selectTrip` 呼び出し。`.trip-card:focus-visible` でキーボード操作時のみフォーカスリング表示 |

---

## Sprint 4：残りのプロパティ入力と写真（完了：2026-05-14）

### 実装内容

#### IndexedDB（写真ストレージ）
- `App.photoDB` ラッパー実装：`init` / `put` / `get` / `getMany` / `delete`
- DB 名 `tabibako` / ストア `photos` / キー `id`、値 `{ id, dataURL, createdAt }`
- `App.init()` を `async` 化し、`photoDB.init()` を `await`

#### マルチセレクト UI（Notion 型ドロップダウン）
- `App.multiSelect` モジュール：3 プロパティ（transport / companions / tags）で UI 共有、値プールは独立
- 値プールキー：`tabibako_transport_pool` / `tabibako_companions_pool` / `tabibako_tags_pool`
- 移動手段の初期値：徒歩・自転車・車・電車・バス・飛行機・船
- フィールドクリック → ドロップダウン展開、チップトグル、テキスト入力フィルター、「+ "xxx" を新規追加」末尾表示
- キーボード：↑↓ / Enter / Tab / Esc / Backspace（空入力で最後のチップ削除）
- IME composition 対応：`e.isComposing` チェックで変換中の再描画事故を回避（`compositionend` で確定後の再描画）
- 外クリック検出は root 側の `e.stopPropagation()` でドキュメント外クリックハンドラに到達させない（DOM 差し替え時の `closest()` null 問題を回避）

#### 値プール削除機能（仕様書「将来対応」の前倒し）
- ドロップダウン候補のホバー時に × ボタン表示
- クリック → 確認ダイアログ → `multiSelect.deleteFromPool()`
- 保存済みの旅行データには **影響しない**（過去の旅行のチップは残る）

#### モーダル拡張
- 5 フィールド追加：移動手段 / 同行者 / 宿泊所 / タグ / メモ / 写真
- 訪問場所の必須を解除（仕様書では必須だったが UX 柔軟性のため）
- `modalSave` を `async` 化し、写真の IDB コミットを await

#### 写真アップロード
- `App.photoUpload` モジュール
- `createImageBitmap` + Canvas で 1600px / JPEG 0.80 にリサイズ
- EXIF 方向は `imageOrientation: 'from-image'` で自動反映
- ドラッグ&ドロップ + ファイル選択ボタン
- アップロード前プレビュー（× で個別削除）
- 同時 10 枚上限、超過時アラート
- 保存時に IDB に `put`、draft.photos には `{ id }` 参照のみ

#### 旅行カードのタグ表示
- 移動手段 / 同行者 / タグの全件を 1 行に横スクロール表示
- 文字 10px、padding 2px 6px、border-radius 4px、色は青 / 緑 / 紫
- 件数上限なし、件数が多ければ横スクロールで対応

#### 右パネル詳細拡張
- セクション追加：移動手段（青タグ）/ 同行者（緑タグ）/ 宿泊所 / タグ（紫タグ）/ メモ
- 空のプロパティはセクションごと **非表示**（情報量を絞る）

#### 写真ビューア（右パネル上部）
- 選択中の旅行の photos を IDB から非同期で読み込み
- 1 枚ずつ表示、左右矢印ボタンで切替
- 写真が 2 枚以上のときのみ矢印 + カウンター（N / M）表示
- キーボード ← → 対応（input/textarea フォーカス時とモーダル開いている時は無効）
- `object-fit: contain` で全体表示、`createImageBitmap` リサイズ済みなので軽量

### 設計上の決定事項

| 決定 | 理由 |
|------|------|
| 写真は IndexedDB に分離、旅行データは localStorage 維持 | 想定規模（30 旅行 × 5 枚）では localStorage 上限を超える。Issue #1 で決定 |
| 写真スキーマは `[{ id }]` の参照形式 | 将来サムネと原本を分離した際にスキーマ拡張可能（`{ id, dataURL: thumbnail, originalRef }` 等） |
| `createImageBitmap` + `imageOrientation: 'from-image'` | EXIF 回転の自前パース不要。Chrome 88+ で標準対応 |
| 写真処理は同時 10 枚を上限 | 1 操作で大量アップロード時のメモリ・時間消費を抑制 |
| マルチセレクトの値プール削除を前倒し | 仕様書「将来対応」だがユーザー要望。実装が軽く、保存済みデータに影響しない設計のためリスク低 |
| 訪問場所の必須を解除 | UX 柔軟性。「日程と名前だけ先に記録」のような使い方を許容 |
| マルチセレクトの IME 対応は `e.isComposing` + `compositionend` | `input` イベントを変換中に発火させない標準パターン |
| 右パネル詳細の空セクション非表示 | 情報量制御。空欄が並ぶより視認性が高い |
| 写真ビューアのキーボード操作はモーダル中は無効化 | モーダル内で Tab 移動・テキスト入力中に意図せず写真切替が起きないように |

### 残課題（Sprint 5 以降）

#### Sprint 5 でやること
- **編集機能**：保存済み旅行データのモーダル再オープン + 上書き保存（multiSelect・photo の事前ロード含む）
- **削除機能**：確認ダイアログ → 旅行データ削除 + IDB の写真も連動削除（**整合性重要**）
- **絞り込み 3 セクション**（移動手段・同行者・タグ）の独立 ON/OFF
- **ソートドロップダウン**（日程新しい順 / 古い順 / 旅行名順）
- **訪問都道府県の自動判定**（緯度経度 → 都道府県名の逆引き、Nominatim or 静的データ）

#### Sprint 6 以降に持ち越し
- ヒートマップ（Leaflet.heat）
- 選択時 `fitBounds` オートズーム

### 既知の留意点

- **IDB 非同期化の影響**：`App.init()` が async になったため、init 完了前に `App.state.trips` を読む処理はないが、Sprint 5 編集機能で写真読み込み待ち中の UI 制御に注意
- **写真リサイズの所要時間**：高解像度 4K 写真 1 枚で約 0.5〜1 秒。10 枚同時アップロード時は最大 10 秒ほど待たされるが、プログレス表示で UX 担保
- **localStorage 容量警告は未実装**：旅行データだけなら 5 MB は十分だが、Sprint 5+ で値プールが肥大化したら検討
- **写真の IDB 残骸問題**：Sprint 5 の削除実装で「旅行データ削除 → 写真 ID 参照消失 → IDB に孤立した photo」が発生する可能性。Sprint 5 で削除時に対応する photoId も `photoDB.delete` する必要あり

### Sprint 4 動作確認用チェックリスト

#### マルチセレクト
- [ ] モーダル内の移動手段・同行者・タグの 3 フィールドが表示される
- [ ] 移動手段に初期値 7 種（徒歩・自転車・車・電車・バス・飛行機・船）が候補表示
- [ ] 候補クリックでチップ追加、再クリックで解除
- [ ] テキスト入力で候補絞り込み、未存在なら「+ "xxx" を新規追加」表示
- [ ] **日本語 IME 変換が正常動作**（「sa」→「さ」変換中に画面が壊れない）
- [ ] チップ × で個別削除、Backspace（空入力）で最後のチップ削除
- [ ] キーボード ↑↓ Enter Tab Esc 動作
- [ ] 候補ホバー時の × で値プール削除（確認ダイアログ）
- [ ] 値プール削除後、リロードしても候補に再出現しない
- [ ] 削除前にその値を含んでいた旅行のチップは残る

#### 宿泊所・メモ
- [ ] テキスト入力できる、改行可能（メモ）、長文は折り返し表示

#### 写真
- [ ] 「クリックまたはドラッグ&ドロップ」エリアでファイル選択可能
- [ ] ドラッグ&ドロップでも追加可能
- [ ] 縦向き写真が縦のまま表示される（EXIF 反映）
- [ ] サムネイル表示、× で個別削除
- [ ] 11 枚目を追加しようとするとアラート
- [ ] 保存後リロードしても写真が残る

#### 右パネル
- [ ] 選択中の旅行の全フィールドが表示される
- [ ] 空のフィールドは表示されない（セクションごと非表示）
- [ ] 移動手段は青、同行者は緑、タグは紫タグ表示
- [ ] 写真ビューアで複数写真を矢印で切替、カウンター表示
- [ ] キーボード ← → でも切替

#### 旅行カード
- [ ] 全タグ（移動手段・同行者・タグ）が色分けして表示
- [ ] 件数が多いと横スクロール

### レビュー指摘と修正（Sprint 4、2026-05-14）

| 指摘 | 原因 | 修正 |
|------|------|------|
| 写真ビューア矢印が `‹/›` グリフでアイコンシステムから外れる | 単純に文字グリフを直書きしていた | Tabler Icons の `ti-chevron-left` / `ti-chevron-right` に置換 |
| `canvas.getContext('2d')` が null を返した場合 `drawImage` で TypeError | 低メモリ環境で null になる可能性が考慮されていなかった | null チェック → 例外スロー（呼び出し側でユーザーへのフィードバックに繋がる） |
| `commitToDraft` の IDB put 失敗を握り潰していた | catch で console.error のみ。失敗が呼び出し側に伝わらず「保存成功と見えるが写真が抜ける」事故リスク | 失敗カウンタを保持し、1 枚でも失敗があれば例外スロー。`modalSave` 側で catch して `form-error` に表示 |
| `multiSelect` ルートクリックで `e.stopPropagation()` 多用、将来のグローバル挙動追加で詰む可能性 | DOM 差し替え後の `e.target` が detached になり `closest()` が機能しないので無効化していた | 外クリック検出を `click` ではなく **`mousedown`** に変更（差し替え前に判定）。stopPropagation を除去 |
| document レベルのリスナーが `multiSelect.bindEvents` / `photoViewer.bindEvents` 内に分散 | モジュールごとに自前で `document.addEventListener` していた | `App.bindEvents` に集約。各モジュールは `closeAll()` / `handleArrowKey(e)` のような公開メソッドを提供 |

### 着手前の決定事項（Issue から転記）

#### Issue #2：マルチセレクト UI コンポーネント仕様（2026-05-14 決定済み）

| 項目 | 決定 |
|---|---|
| 既存値の表示形式 | **(b) Notion 型ドロップダウン** — フィールドクリックで展開、チップ表示でトグル選択 |
| 新規値の追加 | テキスト入力でフィルター、未存在なら末尾に「+ "xxx" を新規追加」表示 |
| 重複検知 | `trim()` + 完全一致（大小文字・全半角は区別） |
| 値の解除 | チップ右側 × ＋候補再クリックで toggle ＋ Backspace（空入力時に最後の値削除） |
| キーボード操作 | ↑↓ で候補移動、Enter で確定、Tab で次入力、Esc で閉じる |
| 紫系カラートークン | **モック v6 から抽出して `:root` に追加**（実装時に値確定） |
| 旅行カードへのタグ表示 | **(a) mock 通り全タグ表示**（移動手段=青・同行者=緑・タグ=紫）。文字 10px、padding 2px 6px、border-radius 4px、日程行の下に **横スクロール** で配置、件数上限なし |

#### Issue #1：写真ストレージ戦略（2026-05-14 決定済み）

| 項目 | 決定 |
|---|---|
| **戦略** | **案 B：IndexedDB で写真管理**（30 旅行 × 5 枚規模で localStorage 上限を超えるため、案 A だけでは不可能と判断） |
| IDB データベース名 | `tabibako` |
| IDB ストア名 | `photos` |
| キー設計 | photo uuid（`crypto.randomUUID()`）。値は `{ id, dataURL, createdAt }` |
| 旅行データ photos プロパティ | `[{ id: 'photo-uuid' }]` の参照のみ。本体は IDB |
| 写真削除時の整合性 | 旅行削除時に対応する photoId を IDB からも削除（Sprint 5 削除実装時に対応） |
| 容量警告 UI | Sprint 4 では実装せず。実用負荷が見えてから検討 |
| **リサイズ最大長辺** | **1600px**（容量問題は IDB で解決済みだが、原本数 MB の保存は無駄なので品質と効率のバランス点） |
| **JPEG 品質** | **0.80** |
| **EXIF 方向保持** | **必須**（Canvas で読み込むと回転情報が失われるため明示処理） |
| **同時アップロード上限** | **10 枚 / 1 操作** |
| **アップロード前プレビュー** | **あり**（誤添付防止） |
| **写真ビューア：矢印キー** | **あり** |
| **写真ビューア：全画面 lightbox** | **なし**（Sprint 4 はサムネ + 横スクロール）。実用後に再検討 |
| **写真ビューア：スワイプ** | **なし**（デスクトップ Chrome 前提） |

---

## Sprint 5：編集・削除・絞り込み・ソート（完了：2026-05-14）

### 実装内容

#### 編集機能
- `modalOpen(tripId)` 引数で「新規 / 編集」を分岐
- 編集モード時：trip をディープクローンして draft に展開、全フォーム要素・multiSelect・photoUpload を既存値で初期化
- `App.state.editingTripId` / `editingOriginalPhotoIds` で編集対象を追跡
- `_preModalSelectedId` で modal open 前の選択を保持、close で復元
- 編集中の重複ピン回避：selection を一時クリア（draft pins のみ表示）
- モーダルタイトルを「旅行を追加」/「旅行を編集」で動的切替
- `photoUpload._pending` に `isNew` フラグを追加：既存写真は IDB に再 put しない（無駄を排除）
- 保存時に削除された写真を IDB からも自動削除

#### 削除機能
- `deleteSelectedTrip()`：確認ダイアログ → IDB 写真クリーンアップ → `state.trips` から除去
- 選択状態をクリアし、地図ピン・右パネル・写真ビューア・action ボタンを全再描画
- `updateActionButtons()` で編集・削除ボタンを `selectedTripId` の有無に同期

#### 訪問地域自動判定（旧称：都道府県）
- 47 都道府県の静的リスト + `extractRegion` ヘルパ
- 日本国内 → 都道府県、海外 → display_name 末尾の国名
- `place.region` / `trip.regions` フィールド
- マイグレーション：旧 `prefecture` / `prefectures` を `region` / `regions` にリネーム + 削除
- 旅行カード 2 行目に訪問地域表示（仕様書通り「訪問場所 N か所」から切替）
- 右パネル詳細に「訪問地域」セクション

#### ソートドロップダウン
- `App.sort` モジュール：3 種類（日程新しい順・古い順・旅行名あいうえお順）
- 選択状態を `tabibako_sort` に保存・次回復元
- 既存のソートボタンクリックで開閉、外クリックで閉じる
- `renderTripList` で `sort.apply()` 適用

#### 絞り込み 3 セクション
- `App.filter` モジュール：移動手段・同行者・タグの 3 系統独立
- 左パネル `.filter-section` にチップ表示、クリックで toggle
- カラータグ（青・緑・紫）でカテゴリ識別
- ロジック：同カテゴリ内は **OR**、カテゴリ間は **AND**
- 値プールに何もない場合はセクション非表示
- 該当 0 件の場合は「絞り込み条件に該当する旅行がありません」表示
- フィルタ状態を `tabibako_filter` に保存・次回復元
- キーボード（Enter / Space）対応

### 設計上の決定事項

| 決定 | 理由 |
|------|------|
| 編集時に selectedTripId を一時クリア | 既存ピン（selectedTripMarkers）と draft ピンの重複表示を回避。close 時に `_preModalSelectedId` から復元 |
| `photoUpload._pending` に isNew フラグ | 既存写真の再 put を回避（IDB 書き込み負荷削減） |
| 削除時に IDB 写真も連動削除 | Sprint 4 で予告していた残骸対応。失敗してもログのみ・state 削除は完遂 |
| 訪問地域：日本=都道府県・海外=国名 | ユーザー要望。空文字 display_name や判定不能のケースは「訪問地域未判定」フォールバック |
| フィールド名を `region`/`regions` に統一 | 国内外を区別しない命名。マイグレーションで旧 `prefecture`/`prefectures` を自動移行 |
| 絞り込み：同カテゴリ OR / カテゴリ間 AND | 「自転車 OR 徒歩 AND 家族 AND 日帰り」のような自然な条件式 |
| フィルタ状態は localStorage に永続化 | リロード後も継続。意図的に開きっぱなしの絞り込みを許容 |

### 残課題（Sprint 6 以降）

#### Sprint 6 でやること
- ヒートマップ（Leaflet.heat プラグイン）
- 旅行選択時の `map.fitBounds()` オートズーム

#### Sprint 7 以降に持ち越し / Issue 化
- 左右パネルのドラッグリサイズ（Issue #3）
- 期間絞り込み（年・月）— 仕様書に記載あるが Sprint 5 では未実装
- 設定画面（値プール削除を集約した管理画面）
- データのエクスポート / インポート（JSON）

### 既知の留意点

- **編集時の写真メモリ**：既存写真が DataURL で `_pending` に展開されるため、写真 10 枚の編集だと 2〜3MB のメモリ使用。実用上問題なし
- **renderTripList の全再描画**：絞り込み・ソート・編集すべてで innerHTML 全置換。trip 数が増えると無駄。Sprint 7+ で DOM diff を検討
- **フィルタ UI の値プール表示**：プールに削除済みでも、選択中（`_selected` に残っている）の値があると不整合。実用上の影響は低いが、将来 `_selected` をプールと同期する処理を追加検討
- **絞り込みなしで該当 0 件**：state.trips.length === 0 と filter 後 0 件で異なるメッセージを表示（仕様通り）

### レビュー指摘と修正（Sprint 5、2026-05-14）

| 指摘 | 原因 | 修正 |
|------|------|------|
| プール削除後、filter._selected に残った値で「該当 0 件ロック」する可能性 | フィルタ選択とプールが独立管理されていた | `filter._syncWithPool()` を `render()` 冒頭で呼び、プールに存在しない値を `_selected` から自動掃除。`multiSelect.deleteFromPool` から `App.renderTripList()` を呼んで同期 |
| `sort.getCurrent()` が localStorage の不正値をそのまま返して未ソート状態に | バリデーション無しで raw 値を使用 | `OPTIONS.some` でホワイトリスト判定し、不一致なら `DEFAULT` フォールバック。`setCurrent` も同様にバリデート |
| 編集時：旧写真の IDB 削除が localStorage 保存より前 → save 失敗時に写真消失 | 順序が不適切 | `storage.save()` を throw 化し、modalSave / deleteSelectedTrip は「state 更新 → save → IDB 削除」の順に整理。save 失敗時に state を rollback |
| `place-search` の住所検索が IME 変換中の中間文字でも走る | Sprint 4 で multiSelect のみ IME 対応、search input には未対応 | `compositionstart` / `compositionend` フラグ + デバウンス関数内で isComposing 判定 |
| 編集モーダルが写真ロード完了まで表示されず体感遅延 | `modal-backdrop.hidden = false` が `await loadFromTrip` の後 | 写真エリアを先に空状態描画 → モーダル表示 → 写真は非同期でバックグラウンドロード |
| `extractRegion` が逆引き失敗時のフォールバック座標表示「(35.0000, 139.0000)」を誤判定 | 単純な `split(',')` ベース | 冒頭で正規表現 `/^\([-\d.]+,\s*[-\d.]+\)$/` ガード追加 → null 返却 |
| `filter-chip` に `aria-pressed` 未設定（active 状態が SR に伝わらない） | chip 描画 HTML から漏れ | active 状態と同期して `aria-pressed="true|false"` 付与 |

### Sprint 5 動作確認用チェックリスト

#### 編集
- [ ] カード選択 → 編集ボタン有効化
- [ ] 編集ボタンでモーダル open、全フィールド・写真が反映される
- [ ] 編集中に写真追加・削除可能、保存で反映
- [ ] キャンセルで元の状態に戻る、選択も復元
- [ ] 削除した写真は IDB からも消える

#### 削除
- [ ] カード選択 → 削除ボタン有効化
- [ ] 削除確認ダイアログ → OK で削除、地図ピン・右パネル消失
- [ ] キャンセルで何も起きない

#### 訪問地域
- [ ] 国内旅行：都道府県表示（例：京都府）
- [ ] 海外旅行：国名表示（例：France）
- [ ] 既存データもリロード後に区分けされている

#### ソート
- [ ] ソートボタン → ドロップダウン展開、現在の選択肢に ✓
- [ ] 各オプション選択で旅行カードの順序が変わる
- [ ] リロード後も選択状態が保持される

#### 絞り込み
- [ ] 左パネルにフィルタ 3 セクションが表示（プールに値があるもののみ）
- [ ] チップクリックで active 化、該当する旅行のみカード表示
- [ ] 複数選択：同カテゴリ OR、カテゴリ間 AND
- [ ] 該当 0 件のとき適切なメッセージ
- [ ] リロード後もフィルタ状態が保持される

---

## Sprint 6（オプション）：ヒートマップ・オートズーム（完了：2026-05-14）

### 実装内容

#### Leaflet.heat プラグイン
- `https://unpkg.com/leaflet.heat@0.2.0/dist/leaflet-heat.js` を CDN 読み込み
- SRI は省略（プラグインゆえ + file:// プロトコル前提）

#### App.heatmap モジュール
- `_enabled` / `_layer` の状態管理、`tabibako_heatmap` に永続化
- `getPoints()`：フィルタ適用後の全旅行の places を `[lat, lng]` 配列に展開（F-17 自動充足）
- `render()`：`L.heatLayer(points, { radius: 25, blur: 18, maxZoom: 18 })`
- `toggle()`：state 反転 → render → 個別ピン再描画 → ボタン active 切替

#### ヒートマップボタン
- 既存 `#heatmap-btn` をクリックハンドラ wire up
- `.map-heatmap-btn.active`：info 色系（青）で active 表示
- `aria-pressed` を状態と同期

#### モード排他切替（ON 時は個別ピンを隠す）
- `renderSelectedTripPins` 内で `heatmap.isEnabled()` チェック
- draft ピンは編集中の visual feedback として常時表示

#### fitBounds オートズーム（F-19）
- `App.fitMapToSelectedTrip()`：選択中の旅行の全 places が画面内に収まるよう調整
- 場所 1 件：`setView` + zoom 12
- 場所 2+ 件：`fitBounds` + padding 40
- ヒートマップ ON 時は俯瞰モード優先で fit しない

#### 全種更新への自動追従
- `renderTripList` の末尾で `heatmap.render()` を呼ぶ → フィルタ変更・旅行追加削除・編集すべてに追従

### 設計上の決定事項

| 決定 | 理由 |
|------|------|
| ヒートマップ ON 時に個別ピンを隠す | 排他切替で視覚情報を絞り込み。両表示すると密集時に判読困難 |
| データソースは「フィルタ適用後の全旅行」 | F-16（全旅行）と F-17（フィルタ連動）を同時に満たす。フィルタなしなら全件・あれば該当のみ |
| fitBounds 中はヒートマップ ON で実行しない | ヒートマップは俯瞰のための表示なので、特定旅行へのズームと相性が悪い |
| 1 件のみ setView + zoom 12 | fitBounds に単一座標を渡すと過度にズームインするため明示的に固定 |
| 永続化（`tabibako_heatmap`） | UX 一貫性：他のソート / フィルタと同様に状態を覚える |

### 残課題（仕様書範囲のオプション機能）

#### 未実装の F-14〜F-18（必要が出てから対応）
- F-14：複数旅行の同時選択 → 現状は単一選択 toggle
- F-15：複数旅行の詳細切り替え（右パネル左右矢印） → F-14 が前提
- F-18：ズーム連動切り替え（ズームアウトで自動ヒートマップ・ズームインで自動ピン）
- F-17（ヒートマップフィルター）の一部：「移動手段・期間でヒートマップ対象を絞る」のうち期間（年・月）絞り込み未実装

#### Issue 化 / Sprint 7+
- Issue #3：左右パネルのドラッグリサイズ
- 期間絞り込み（年・月）
- データのエクスポート / インポート

### 既知の留意点

- **Leaflet.heat の負荷**：ポイント数が数千を超えると描画が重くなる可能性。Tabibako の規模（150 か所程度）では問題なし
- **空ポイント時の挙動**：`getPoints()` が 0 件なら何も描画しない（layer 生成自体スキップ）
- **CDN 依存**：Leaflet.heat 0.2.0 は実質的に最終リリース版（2015 年以降更新少）。安定だがメンテリスクは認識
- **ヒートマップとピンの併用要望が出たら**：別 layer として共存可能。現状の排他切替を `expanded mode` 等で拡張する余地あり
- **ヒートマップパラメータ調整**：少量データでも視認できるよう `radius: 35` / `blur: 25` / `minOpacity: 0.35` に設定。デフォルトより強め

### レビュー指摘と修正（Sprint 6、2026-05-14）

| 指摘 | 原因 | 修正 |
|------|------|------|
| ヒートマップボタンの初期 `aria-pressed` / `aria-label` 欠落（SR が状態認識不可） | HTML に書かず JS で動的設定していた | HTML に初期値を直書き |
| `fitMapToSelectedTrip` の NaN / 範囲外座標耐性なし | 不正座標で `L.latLngBounds` が無音失敗 or 地図破壊 | `Number.isFinite` + 緯度経度範囲チェックで除外、`bounds.isValid()` 確認 |
| `heatmap.render()` の `App.map` null チェックなし | init 順序によっては NPE のリスク | 冒頭ガード `if (!App.map) return;` |
| `devClearAll` の localStorage キーをハードコード重複 | 各モジュールの定数を再定義 | 各モジュールの `STORAGE_KEY` / `POOL_KEYS` を参照する形に集約 |
| `init` 末尾で `heatmap.render()` が二重実行 | `renderTripList` 内でも呼ばれているため | init 側の明示呼び出しを削除（コメントで意図明示） |

### GitHub Issues #6 / #7 / #8 マージ（2026-05-15）

GitHub Actions Claude にブランチ実装を依頼 → 第三者コードレビュー → 修正後マージの流れで処理。

#### Issue #8：画面上部のピックモード案内バナー削除（マージ済み）
- pick-mode-banner 要素・スタイル・参照を一括削除
- ピックモード中の視覚フィードバックは body.pick-mode (cursor:crosshair) + modal-backdrop.pick-mode（右下縮小）で担保

#### Issue #6：技術的負債整理（マージ済み + レビュー修正適用）
- `renderSelectedTripPins` → `renderAllTripPins`、`selectedTripMarkers` → `allTripMarkers` にリネーム
- `state.modalActive` フラグを新設（旧来は `state.draft` を抑制フラグに転用していた）
- `renderPinCount`：ヒートマップ ON 時はバッジ自体を非表示に
- ポップアップ文言に空文字フォールバック
- **レビュー修正**：`.map-pin-count[hidden] { display: none }` を追加（display: flex が hidden 属性を打ち消していたバグ）

#### Issue #7：宿泊所を地図から選択可能に（マージ済み + レビュー修正適用 + rebase）
- 宿泊所フィールドを訪問地直下に移動（モーダル UI 順序）
- 住所検索 + 「地図から選ぶ」UI を訪問地と同じパターンで実装
- accommodation: string → { name, lat, lng } に拡張、storage.load で旧データ自動マイグレーション
- 宿泊所ピンを ti-bed の緑系アイコンで表示
- ピックモードに `pickTarget`（'place' | 'accommodation'）を追加、`onMapClick` で target に応じて分岐
- `fitMapToSelectedTrip` に宿泊所座標も含める
- **レビュー修正**：
  - 右パネル詳細でも宿泊所セクションを訪問場所の直下に移動（モーダル UI 順と整合）
  - setPickMode で `on=false` 時に pickTarget をデフォルトにリセット（target 取り違え防止）
  - Issue #6 のリネーム / Issue #8 のバナー削除と整合させる rebase 対応

#### 残課題（Sprint 7 候補）

- 宿泊所検索 UI（searchAddress / renderSearchResults 等）が訪問地検索と約 100 行重複。`createSearchController` 風のファクトリで一本化推奨
- 宿泊所ピンに variant（dim/selected）が無く、非選択時に他旅行の宿泊所が表示されない（Issue #5 の方針と乖離）
- 逆引き失敗時の `(35.0000, 139.0000)` 文字列が宿泊所名に入る違和感

### Issue #3 対応：左右パネルのドラッグリサイズ（2026-05-14）

機能追加：左右パネルの境界をドラッグして幅を変更可能に。

#### 実装内容

- `.app` の `grid-template-columns` を **5 列構造**に変更：`<left>px 4px 1fr 4px <right>px`
- `.resize-handle[data-side="left|right"]` を 2 本配置（`role="separator"` + `aria-orientation="vertical"`）
- **`App.panelResize`** モジュール
  - `_left` / `_right` を内部状態として保持、`tabibako_panel_widths` に JSON で永続化
  - `applyWidths()`：`grid-template-columns` を書き換え、`requestAnimationFrame` で `App.map.invalidateSize()` を間引き呼び出し
  - `startDrag` → `onDragMove` → `endDrag` のフローで document に mousemove/mouseup リスナーを張る（要素外でも追従）
  - 最小幅制限：左 240px / 右 280px / 中央 320px。中央の最小を割らないよう左右を巻き戻し
  - **ダブルクリックで初期値（384/390）リセット**
- `body.resizing` クラスで `cursor: col-resize !important` + `user-select: none` を画面全体に適用

#### 設計上の決定事項

| 決定 | 理由 |
|------|------|
| grid を 5 列構造に拡張 | ハンドルを grid アイテムとして配置することで `position: absolute` の煩雑さを回避 |
| ハンドル幅 4px | 細すぎず誤クリックも起きにくいバランス |
| `requestAnimationFrame` で invalidateSize 間引き | mousemove は秒間 60+ 回発火するが、地図再計算は描画フレームに 1 回で十分 |
| 最小幅違反時は対辺を巻き戻す | 中央地図を維持するため、ドラッグ方向のパネルだけが拡大を諦める動作 |
| タッチ対応はスコープ外 | CLAUDE.md の Chrome デスクトップ前提に従う |

#### 残課題

- タッチデバイス対応（必要が出てから検討）
- リサイズ中の Nominatim popup 位置追従（マイナー）
- aria-valuemin/valuemax/valuenow + キーボード矢印キーリサイズ（separator の完全 a11y。Sprint 7 候補）

#### レビュー指摘と修正

| 指摘 | 修正 |
|------|------|
| `startDrag` の二重起動でリスナーが残留するリスク | startDrag 冒頭で `_dragging` truthy なら `endDrag()` を先に呼ぶ防御を追加 |
| ウィンドウ外で mouseup された場合に `body.resizing` が残る | `window.addEventListener('blur', endDrag)` をドラッグ中のみ張る。endDrag で確実に解除 |
| 小ウィンドウで `MIN_LEFT + MIN_RIGHT + MIN_CENTER > usable` となり地図が 0px に潰れる | `_clampToWindow()` メソッドで段階的に縮小。中央地図最優先・左右は MIN まで縮められる |
| `load()` の上限チェック欠落（巨大値や旧大画面値で破綻） | load 末尾で `_clampToWindow()` を呼ぶ |
| ウィンドウリサイズ未追従 | `window.addEventListener('resize', ...)` で clamp + applyWidths |
| DEFAULT_* と `_left`/`_right` の二重管理 | load 冒頭で `DEFAULT_*` で初期化してから localStorage 上書きする一元化 |

### F-18（ズーム連動切替）について

仕様書のオプション機能 F-18「ズーム連動でヒートマップ／ピン自動切替」は **不要として据え置き**。現状の手動トグルで十分との判断。実装ニーズが出てから検討。

### Issue #5 対応：非選択時の全旅行ピン表示（2026-05-14）

仕様変更。Sprint 6 完了後の機能追加：

- **未選択時**：フィルター済み全旅行の訪問場所を **青ピン** で表示（地図でフィルター結果を俯瞰）
- **選択時**：選択旅行のピンを **オレンジ + 一回り大きく**、他旅行を **グレー青** で色分け
- **ポップアップ**：「旅行名：場所名」形式で識別性向上
- **モーダル中**：`state.draft` truthy で全ピン表示を抑制（draft pins に集中）
- **ヒートマップ中**：従来通り個別ピン全非表示

実装ポイント：
- `createPinIcon(variant)` に `'default' | 'dim' | 'selected'` の 3 バリアント追加
- `renderTripList()` から `renderSelectedTripPins()` を呼ぶ設計に変更 → フィルター・ソート・追加削除すべてに自動連動
- `:root` にピン色トークン `--color-pin-dim` / `--color-pin-selected` 追加

これにより、地図とリストの **連動性** が大きく向上。ヒートマップとは「個別の点（場所）」vs「密度（頻度）」で役割が分離される。

### Sprint 7 着手前の検討項目（レビュー指摘の据え置き分）

- **dev ヘルパの本番分離方針**：`DEV` フラグ or コメント目印で削除しやすくする検討
- **Leaflet.heat の SRI ハッシュ**：将来 GitHub Pages 公開時の整合性のため付与検討
- **localStorage キー集約**：複数モジュールに散らばっているキーをルート定数に集める
- **`heatmap.render()` の呼び頻度**：`renderTripList` 都度ではなく差分更新で十分なケースが多い
- **ヒートマップ ON 時 + 旅行選択時の UX**：現状「fit しない + 個別ピンも出ない」で挙動不明瞭。可視ハイライト追加余地
- **devLoadSamples 配列の本体混在**：80 行のサンプル配列を別ブロックに切る or 別ファイル化（単一 HTML 方針との整合は要検討）

### Dev ヘルパ（コンソールから手動実行）

Sprint 6 実装中、データが少ないとヒートマップの視認性が低いため、テスト用のヘルパを実装：

- **`window.devLoadSamples()`**：日本各地 + 海外を含むサンプル 12 旅行（計 30 か所程度）を一括投入。値プールにも値を追加
- **`window.devClearAll()`**：localStorage の Tabibako 関連キー全削除 + IDB の photos ストア clear。確認ダイアログあり

使い方：ブラウザの DevTools コンソール（F12 → Console タブ）で `devLoadSamples()` と入力 Enter。リロードせず即反映される。クリアは `devClearAll()` → リロード。

これは開発・デモ用途のヘルパであり、UI には公開しない。将来 Sprint で「データインポート/エクスポート」機能を実装したら撤去候補。

### Sprint 6 動作確認用チェックリスト

- [ ] 「ヒートマップ」ボタン → 押下で active 化（青背景）
- [ ] 全旅行（フィルタ適用後）の places が赤/黄/緑の濃淡で表示
- [ ] active 時：選択中の旅行の個別ピンが消える、ヒートマップが見える
- [ ] フィルタを切り替えると、ヒートマップ も対象データが更新される
- [ ] 旅行カード選択時、地図が **自動でズーム** されて該当 places を画面内に収める
- [ ] 場所 1 件の旅行を選択：その場所が中心、ズーム 12
- [ ] 場所複数の旅行を選択：全ピンが余白付きで収まる
- [ ] ヒートマップ ON 時に旅行を選択しても **fit しない**（俯瞰維持）
- [ ] ブラウザリロード後もヒートマップの ON/OFF 状態が保持される
