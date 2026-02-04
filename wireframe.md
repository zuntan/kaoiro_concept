# ワイヤーフレーム (wireframe.md)

## 1. メインウィンドウ (通常時)
親しみやすさを重視した、ミニマルな構成。

```mermaid
graph TD
    %% スタイル定義 (視認性向上)
    classDef window fill:#f9f9f9,stroke:#333,stroke-width:2px,color:#000
    classDef area fill:#e0e0e0,stroke:#666,stroke-width:1px,color:#000
    classDef overlay fill:#ffeb3b,stroke:#fbc02d,stroke-width:2px,color:#000,stroke-dasharray: 5 5
    classDef text fill:#fff,stroke:none,color:#333

    subgraph MainWindow ["Main Window"]
        direction TB
        subgraph PreviewArea ["Camera Preview (Processed)"]
            IconOverlay["Reaction Icon (Face Icon)"]
        end
        StatusText["Current Action (e.g., 'Typing YES...')"]
    end

    %% クラス適用
    class MainWindow window
    class PreviewArea area
    class IconOverlay overlay
    class StatusText text
```

- **PreviewArea**: プライバシー加工済みのカメラ映像。
- **IconOverlay**: 判定状態に応じた大きな顔アイコン（👍, 🙅‍♂️, ☺ 等）。
- **StatusText**: 実行中のアクションをテキストで補助表示。

## 2. 設定パネル (展開時)
カメラ選択やウィンドウ選択などを直感的に配置。

```mermaid
graph TD
    %% スタイル定義
    classDef panel fill:#f0f4c3,stroke:#333,stroke-width:2px,color:#000
    classDef section fill:#fff,stroke:#999,stroke-width:1px,color:#000
    classDef element fill:#e1f5fe,stroke:#0277bd,stroke-width:1px,color:#000
    classDef button fill:#ffccbc,stroke:#d84315,stroke-width:2px,color:#000

    subgraph SettingsPanel ["Settings Panel"]
        direction TB
        
        subgraph Section1 [Devices]
            CamSelect[Camera Dropdown]
            MicSelect[Mic Dropdown]
        end
        
        subgraph Section2 [Target]
            WindowSelect[Target Window Dropdown]
        end
        
        subgraph Section3 [Adjust]
            SensSlider[Response Sensitivity]
        end
        
        PauseButton[Global Pause Button]
    end

    %% クラス適用
    class SettingsPanel panel
    class Section1,Section2,Section3 section
    class CamSelect,MicSelect,WindowSelect,SensSlider element
    class PauseButton button
```

## 3. 状態遷移表示
顔アイコンと色による視覚的フィードバック。

| State | Face Icon | Theme Color | Meaning |
| :--- | :--- | :--- | :--- |
| **Idle** | ☺ (Wait) | Blue/Gray | 待機中（穏やかな顔） |
| **YES** | 😄 (Yes!) | Mint Green | 肯定（笑顔・頷き） |
| **NO** | 😟 (No...) | Coral Red | 否定（困り顔・首振り） |
| **Paused** | 💤 (Sleep) | Gray | 一時停止中 |
