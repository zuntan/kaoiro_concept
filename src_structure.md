# ソースコード構造案 (src_structure.md)

`egui` (eframe) を採用し、Native版とWASM版でコードを最大限共有するためのディレクトリ構成案。

## 1. 構成図

```text
kaoiro/
├── Cargo.toml
├── Makefile.toml           # cargo-make用タスク定義
├── src/
│   ├── main.rs             # Native版エントリーポイント
│   ├── lib.rs              # WASM版/共通ライブラリのルート
│   ├── app.rs              # メインアプリケーション（eframe::App実装）
│   ├── core/               # プラットフォーム非依存のロジック
│   │   ├── mod.rs
│   │   ├── gesture.rs      # 頷き・首振り判定アルゴリズム
│   │   ├── state.rs        # 共有状態 (IDLE/YES/NO等)
│   │   └── privacy.rs      # 画像加工ロジックのインターフェース
│   ├── ui/                 # egui コンポーネント
│   │   ├── mod.rs
│   │   ├── widgets.rs      # カスタムウィジェット（ボタン、スライダー）
│   │   ├── panels.rs       # 設定パネル、メインパネルのレイアウト
│   │   └── icons.rs        # SVGアイコンの埋め込み・描画
│   └── platform/           # プラットフォーム依存の実装
│       ├── mod.rs
│       ├── native/         # OpenCVカメラ, Enigo入力
│       └── web/            # web_sysカメラ, DOM操作
├── extension/              # ブラウザ拡張機能用アセット
│   ├── manifest.json
│   └── pkg/                # wasm-packの出力先
└── assets/                 # SVG, フォント等の静的リソース
```

## 2. 各ディレクトリの役割

### 2.1. `src/core/`
UIやプラットフォームから独立した純粋な計算・ロジック層。
- `gesture.rs`: 顔の座標データを受け取り、時間軸での変化から「頷き」か「首振り」かを返すステートマシン。
- `state.rs`: `Arc<RwLock<AppData>>` 等で管理される、アプリ全体の共有ステート。

### 2.2. `src/ui/`
`egui` を用いた描画ロジック。
- プラットフォームに依存せず、`egui::Ui` オブジェクトに対して描画命令を出す。
- `icons.rs`: 準備した `avatar_set_*.svg` を読み込み、状態に合わせて描画する。

### 2.3. `src/platform/`
カメラ映像の取得と、最終的な「YES/NO」の送信（キーストロークまたはDOM操作）を、ターゲットプラットフォームに合わせて切り替える。
- `native/`: OpenCVを用いてカメラバッファを取得。
- `web/`: `web_sys` を通じてブラウザの `getUserMedia` を呼び出し。

## 3. 実装上のポイント
- **Conditional Compilation**: `#[cfg(target_arch = "wasm32")]` を活用し、依存関係をクリーンに保つ。
- **Asynchronous Processing**: カメラ映像の取得や重い画像処理は、GUIスレッドをブロックしないよう非同期（または別スレッド）で実行し、結果を `SharedState` 経由で UI に伝える。
