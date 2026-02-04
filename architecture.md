# アーキテクチャ設計 (architecture.md)

## 1. システム全体構成

Native版とWASM版でコードを最大限共有できるよう、コアロジックを分離する。

```mermaid
graph TD
    subgraph "Shared Core (Native & WASM)"
        GestureLogic[Gesture Logic]
        SharedState[Shared State (Arc RwLock)]
        UI_Components[egui Components]
    end

    subgraph "Native Specific"
        OpenCV[OpenCV Camera]
        Enigo[Enigo Input]
        EframeNative[eframe Native Runner]
    end

    subgraph "WASM Specific (Future)"
        WebCam[Web API Camera]
        WebCanvas[Web Canvas Runner]
    end

    %% Data Flow (Native)
    OpenCV -->|Processed Frame| SharedState
    SharedState -->|Current State| UI_Components
    UI_Components -->|Draw Commands| EframeNative
    
    GestureLogic -->|Event| Enigo
```

## 2. コンポーネント詳細

### 2.1. Infrastructure Layer (Native)
- **OpenCV**: 顔検出に加え、プライバシー保護のための画像加工（Gaussian Blur / Masking）もここで行う。
- **Enigo**: キーボードイベントの送信。

### 2.2. Core Layer (Shared)
- **UI Components**: デバイス選択ドロップダウンやプレビュー表示などのUI部品。`egui::Ui` を受け取る関数として実装し、プラットフォーム依存コードを含まないようにする。
- **Privacy Filter**: 画像データを受け取り、設定（ぼかし強度など）に基づいて加工するロジック。

### 2.3. Interface Layer
- **eframe (egui)**:
    - **デバイス管理**: 起動時に利用可能なデバイスをスキャンし、SharedStateのデバイスリストを更新する。
    - **動的切り替え**: ユーザーがデバイスを変更した際、非同期タスクで古いストリームを閉じ、新しいデバイスを開き直す。
