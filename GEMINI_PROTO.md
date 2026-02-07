# 目的
kaoiro プロジェクトの Phase 1 (Browser Extension MVP) を実装する。
カメラ映像から顔のジェスチャー（YES/NO）を判定し、ブラウザ上で動作するプロトタイプを完成させる。

# 実装目標
1. **コアエンジンの構築**: OpenCV(またはRustネイティブライブラリ)を用いた顔検出とジェスチャー判定ロジックの実装。
2. **WASM対応**: RustコードをWASMにビルドし、ブラウザで動作可能にする。
3. **egui UI**: 判定状態を表示するサイドパネルUIの実装。
4. **ブラウザ連携**: 判定結果をブラウザのタブに送り、自動入力・送信を行うPoC。

# 技術スタック
- **言語**: Rust
- **GUI**: `eframe` / `egui` (WASM対応)
- **画像処理**: `opencv` クレート (Native用) / `web_sys` + WASM向け軽量処理 (Web用)
- **ビルドツール**: `wasm-pack`, `cargo-make`

# 主要ファイル構成（予定）
- `Cargo.toml`: 依存関係の整理（`opencv`, `eframe`, `wasm-bindgen`等）
- `src/core/`: ジェスチャー判定ロジック
- `src/ui/`: eguiによるインターフェース
- `src/platform/`: WASM/Nativeの切り替えレイヤー
- `extension/`: ブラウザ拡張用の manifest.json や js ラッパー

# 重点課題
- WASM環境でのカメラアクセスと画像処理パフォーマンスの確保。
- ジェスチャー判定の閾値調整（誤判定の抑制）。
