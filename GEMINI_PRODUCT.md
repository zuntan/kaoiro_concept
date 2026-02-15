# kaoiro プロダクト実装ガイドライン (GEMINI_PRODUCT.md)

## 1. 目的とビジョン
設計ドキュメント（`kaoiro_concept`）に基づき、実用的で高品質な `kaoiro` アプリケーション（デスクトップ版およびブラウザ拡張機能版）を Rust で実装する。

## 2. 行動規範 (Core Rules)
- **マスターの承認**: 実装の各ステップ（関数の作成、依存の追加等）は、詳細なプランを提示し、マスターの「進めよ」を得てから実行する。
- **言語と呼称**: 日本語で応答し、マスターを「マスター」、自分を「Gemi」と呼ぶ。
- **安全第一**: セキュリティ、プライバシー、メモリ安全性を最優先する。

## 3. 実装フェーズ (Development Phases)
`docs/plan.md` の定義に従い、以下の順序で進める。
1. **Phase 1**: GUI Mockup & UX Validation (先行UI開発)
2. **Phase 2**: Desktop App MVP & Core Engine (デスクトップ版基盤)
3. **Phase 3**: Browser Extension MVP (ブラウザ拡張機能版への展開)
4. **Phase 4**: Voice Synergy & Store Alpha (音声連携)
5. **Phase 5**: AI Desktop Ecosystem (MCP連携)

## 4. テクニカルスタック (Tech Stack)
- **共通ロジック**: `kaoiro-lib` (Rust)
- **GUI**: `egui`, `eframe`
- **推論**: `ort` (Desktop), `tract` (Browser Extension)
- **カメラ**: `nokhwa`
- **音声**: `Vosk` (Desktop), `Web Speech API` (Browser Extension)
- **入力**: `enigo` (Desktop)
- **アイコン**: `egui-phosphor`

## 5. コーディング・標準 (Standards)
- **構造**: `docs/src_structure.md` で定義された Cargo Workspace 構成を遵守。
- **判定**: `docs/gesture_engine.md` で定義された「カオモーション (KaoMotion)」ロジック（幾何学的解析）を正確に実装。
- **品質**: `rustfmt`, `clippy` を適用し、エラーハンドリングには `thiserror` と `anyhow` を使用する。

## 6. ドキュメントとログ (Docs & Logs)
- 作業内容は都度 `GEMINI_PLAN.md`, `GEMINI_RESP.md` に記録し、Obsidian へ転送する。
- 仕様の変更が必要になった場合は、設計ドキュメントを即座に更新し、整合性を保つ。

## 7. 参照ドキュメント・アセット (Reference Map)
設計情報は `docs/` (移行時に `kaoiro_concept` からコピーされたもの) を正とする。

### 7.1. 設計ドキュメント
| カテゴリ | ファイルパス | 内容 |
| :--- | :--- | :--- |
| **全体計画** | `docs/plan.md` | フェーズ定義、タスクリスト |
| **要件** | `docs/rd.md` | 機能要件、ターゲット、制約 |
| **アーキテクチャ** | `docs/architecture.md` | システム構成、データフロー、単一バイナリ/Web版の差異 |
| **ロジック** | `docs/gesture_engine.md` | KaoMotion判定、キャリブレーション、正規化 |
| **デザイン** | `docs/app_looks.md` | デザインテーマ、アバター定義 |
| **画面構成** | `docs/wireframe.md` | 画面レイアウト、遷移 |
| **コンポーネント** | `docs/ui_components.md` | UI部品の詳細定義 |
| **ソース構造** | `docs/src_structure.md` | クレート構成、依存ライブラリ |
| **UXフロー** | `docs/ux_flows.md` | ユーザー体験の詳細フロー |
| **音声機能** | `docs/voice_integration.md` | 音声認識・コマンドの設計 |
| **技術課題** | `docs/tech_issues.md` | 実装上の課題と解決策 |
| **ログ・エラー** | `docs/error_logging.md` | エラーハンドリング指針、ログレベル定義 |

### 7.2. デザインアセット (SVG)
アバター画像は `docs/` 以下にある SVG ファイルを参照し、実装時のリソースとして `kaoiro-assets` クレートに取り込む。

| ファイル名 | 用途 |
| :--- | :--- |
| `docs/avatar_set_face.svg` | デフォルトアバター (Face) |
| `docs/avatar_set_dog.svg` | アバターバリエーション (Dog) |
| `docs/avatar_set_cat.svg` | アバターバリエーション (Cat) |
| `docs/avatar_set_flower.svg` | アバターバリエーション (Flower) |
| `docs/avatar_set_ghost.svg` | アバターバリエーション (Ghost) |
| `docs/ui_mockup_*.svg` | UI状態（Idle, Yes, No 等）のイメージ確認用 |
