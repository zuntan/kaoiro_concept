# ソースディレクトリ構成 (src_structure.md)

Rustの標準的な慣習に従いつつ、関心の分離（SoC）を意識した構成とする。

```text
src/
├── main.rs              # エントリポイント。各スレッドの起動と統合
├── core/                # ドメインロジック (Core Layer)
│   ├── mod.rs
│   ├── detector.rs      # ジェスチャー判定アルゴリズム (Pitch/Yaw分析)
│   ├── state.rs         # 共有状態 (State Manager) 定義
│   └── processor.rs     # フレーム変換・前処理
├── infra/               # 外部サービス・デバイス連携 (Infrastructure Layer)
│   ├── mod.rs
│   ├── camera.rs        # カメラドライバ (nokhwa連携)
│   └── face_engine.rs   # 顔認識エンジン (mediapipe/opencv連携)
├── interface/           # 外部インターフェース (Interface Layer)
│   ├── mod.rs
│   ├── gui/             # GPUIによるフロントエンド
│   │   ├── mod.rs
│   │   ├── app.rs       # メインウィンドウ管理
│   │   └── components.rs# 再利用可能なUI部品
│   └── mcp/             # MCPサーバー実装
│       ├── mod.rs
│       ├── handlers.rs  # MCPリクエストハンドラ
│       └── protocol.rs  # プロトコル定義・通信路
└── utils/               # 共通ユーティリティ
    ├── mod.rs
    ├── logger.rs        # ログ設定
    └── config.rs        # 設定ファイル処理 (TOML)
```

## 依存ライブラリの想定
- `gpui`: UIフレームワーク
- `tokio`: 非同期ランタイム
- `nokhwa`: カメラアクセス
- `mcp-sdk-rs`: (仮定) MCPプロトコルサポート
- `anyhow`, `thiserror`: エラーハンドリング
- `serde`: シリアライズ/デシリアライズ
