# ソースコード構造 (src_structure.md)

デスクトップ版/ブラウザ拡張機能版 両対応を見据えた、シンプルかつ実践的な Cargo Workspace 構成。
過度な分割を避け、共通部分は `kaoiro-lib` に集約する。

## 1. ワークスペース構成 (Workspace Structure)

```text
kaoiro/
├── Cargo.toml              # Workspace 定義
├── kaoiro-lib/             # [Shared] 共通ロジック & UIコンポーネント
├── kaoiro-desktop/         # [Desktop] デスクトップ版エントリーポイント
├── kaoiro-web/             # [WASM]   ブラウザ拡張機能版エントリーポイント & Worker
├── kaoiro-assets/          # [Shared] 埋め込みアセット (SVG, ONNXモデル)
└── ...
```

## 2. 各クレートの責務詳細

### 2.1. `kaoiro-lib` (Library)
アプリケーションの中核。ロジックとUIの両方を管理する。
Web Worker等のUI不要環境では、UI関連の関数を呼び出さないことで、リンカによるDead Code Elimination（不要コード削除）に委ねる。

- **主要な依存**:
    - `egui`: 即時モードGUIライブラリ。軽量で高速な描画。
    - `nalgebra`: 線形代数ライブラリ。ジェスチャーの幾何学的判定に使用。
    - `serde`: データのシリアライズ・デシリアライズ（設定保存等）。
- **内容**:
    - `gesture/`: 判定ロジック (`RelativePositionStrategy` 等)。
    - `state/`: アプリケーションの状態管理。
    - `ui/`: UIコンポーネント (`AvatarWidget`, `SettingsPanel`)。

### 2.2. `kaoiro-desktop` (Binary)
デスクトップ版の具体的な実装。
- **主要な依存**:
    - `ort`: ONNX Runtime の Rust バインディング。高速な推論エンジン。
    - `nokhwa`: クロスプラットフォームなカメラアクセスライブラリ。
    - `vosk`: オフライン音声認識エンジン。
    - `enigo`: OSレベルでのキー入力シミュレーション。
- **内容**:
    - `main.rs`: アプリ起動、ウィンドウ設定。
    - `bridge/`: カメラ・マイク入力のアダプタ。
    - `mcp/`: MCPサーバー機能。

### 2.3. `kaoiro-web` (Binary / CDylib)
ブラウザ拡張機能版の具体的な実装。
- **主要な依存**:
    - `tract-onnx`: Rust 純製の ONNX 推論エンジン。WASM環境に最適。
    - `wasm-bindgen`: Rust と JavaScript の相互連携用ツール。
    - `web-sys`: Web API (DOM, Canvas, Web Cam) へのアクセス。
- **内容**:
    - `lib.rs`: WASMのエントリーポイント（UIスレッド）。
    - `worker.rs`: 推論用 Web Worker のエントリーポイント。

### 2.4. `kaoiro-assets` (Library)
バイナリに静的に埋め込むリソースを一元管理するクレート。
`include_bytes!` マクロを使用し、コンパイル時に全てのデータをメモリ上に展開可能な状態で保持する。

- **内容**:
    - `src/lib.rs`: 各アセットへのアクセサ定数を公開。
        - `pub const FACE_DETECTION_MODEL: &[u8] = include_bytes!("../models/face_detection_short_range.onnx");`
        - `pub const AVATAR_FACE_SVG: &[u8] = include_bytes!("../images/avatar_set_face.svg");`
    - `models/`: ONNXモデルファイル。
        - `face_detection_short_range.onnx`: MediaPipe Face Detection (Short Range) のモデル。
    - `images/`: UI用画像アセット。
        - `avatar_set_*.svg`: アバターのバリエーション（Face, Dog, Cat, etc.）。
    - `scripts/`: アセット管理補助スクリプト。
        - `prepare_models.py`: モデルのダウンロードからONNX変換までを一括実行する Python スクリプト。

## 3. ディレクトリ詳細マップ

```text
kaoiro/
├── kaoiro-assets/
│   ├── src/
│   │   └── lib.rs      # include_bytes! definitions
│   ├── models/         # .onnx files
│   ├── images/         # .svg files
│   └── scripts/        # Conversion & Download scripts
│
├── kaoiro-lib/
│   └── src/
│       ├── lib.rs
│       ├── gesture/    # Logic
│       ├── state.rs    # State
│       └── ui/         # UI
│           ├── mod.rs
│           ├── avatar.rs
│           └── app.rs
│
├── kaoiro-desktop/
│   └── src/
│       ├── main.rs
│       └── ...
│
└── kaoiro-web/
    ├── index.html
    ├── Trunk.toml
    └── src/
        ├── lib.rs      # Main Thread Entry
        └── worker.rs   # Worker Thread Entry
```
