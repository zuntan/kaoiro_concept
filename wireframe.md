# ワイヤーフレーム (wireframe.md)

## 1. メインウィンドウ構成 (Main Window)
`egui` の `CentralPanel` をベースとした、レイヤー構造のデザイン。

```mermaid
graph TD
    subgraph Layers
        direction BT
        L1[Base Layer: Background with Blur/Transparency]
        L2[Glow Layer: Status Color Glow - Mint/Coral/Blue]
        L3[Content Layer: Hero Avatar Icon - SVG]
        L4[Indicator Layer: Progress Ring - Visual Feedback]
        L5[Overlay Layer: Status Text & PiP Camera View]
        L6[Control Layer: Hover Controls - Settings/Mute/Close]
        
        L1 --> L2 --> L3 --> L4 --> L5 --> L6
    end
```

### 1.1. 通常表示 (Idle / Active)
ウィンドウ中央に大きくアバターを表示。背景の光彩（Glow）とアバターの表情で状態を直感的に伝える。

```
+--------------------------------------------------+
| [Drag Grip]                                   [X]| <- Hover Only
|                                                  |
|                                                  |
|          (        Avatar        )                | <- Center Piece
|          (      Smile / Idle    )                |
|                                                  |
|                                                  |
|               [ Status Text ]                    |
|             "YES SENT!" (Fade Out)               |
|                                     [ PiP ]      | <- 1/4 size
| [Settings] [Mute]                   [Camera]     |
+--------------------------------------------------+
   (Glow Effect: Mint Green for YES, Coral for NO)
```

- **Avatar**: ウィンドウの中心。ステートに応じて `avatar_set_*.svg` を切り替え。
- **PiP (Picture in Picture)**: カメラ映像。プライバシー保護のため、エッジ検出やシルエット加工を施して表示。
- **Glow Effect**: `egui` の `PaintCallback` や `Mesh` を用いて、ウィンドウ背後にぼかした色面を描画。

## 2. 設定パネル構成 (Settings Modal)
`egui::Window` または `egui::Area` を用いて、メイン画面上にオーバーレイまたは横に展開。

```
+--------------------------------------------------+
| Settings                                      [X]|
+--------------------------------------------------+
| [ Camera Preview (Larger / Silhouette)         ] |
+--------------------------------------------------+
| [ Device Selection ]                             |
| Camera: [ Default Camera       v ]               |
|                                                  |
| [ Detection Settings ]                           |
| Sensitivity: [-------|-------] (Slider)          |
| Strategy:    (o) FFT  ( ) Simple                 |
|                                                  |
| [ Avatar Selection ]                             |
| (o) Face  ( ) Dog  ( ) Cat  ( ) Flower  ( ) Ghost|
+--------------------------------------------------+
| [ Trust Badge: Local Edge AI Processing ]        |
+--------------------------------------------------+
```

## 3. 画面遷移フロー
状態遷移は `app_looks.md` の定義に準拠。

```mermaid
stateDiagram-v2
    [*] --> Idle
    
    Idle --> YesState : Gesture_Nod
    Idle --> NoState : Gesture_Shake
    Idle --> LostState : Camera_FaceLost
    
    YesState --> Idle : Timeout_1.5s
    NoState --> Idle : Timeout_1.5s
    LostState --> Idle : Camera_FaceFound
    
    Idle --> Paused : Mute_Toggle
    Paused --> Idle : Unmute_Toggle
    
    Idle --> Settings : Settings_Open
    Settings --> Idle : Settings_Close
```
