# UIコンポーネント (ui_components.md)

eguiを用いて実装する主要なUIコンポーネント。

## 1. Atoms (基本要素)

### 1.1. ReactionIcon
- **役割**: 現在の判定状態（YES/NO/IDLE）を「顔アイコン」で直感的に表現する。
- **Props**: `state: enum { Yes, No, Idle }`
- **Behavior**: 判定が確定した瞬間に、バウンスするアニメーションと共に表情が変化する。

### 1.2. StatusIndicator
- **役割**: シンプルな色付きドット等で、システムの稼働状態（ON/OFF）を示す。

### 1.3. CameraPreview
- **役割**: カメラバッファ（RGBA画像）を描画する。
- **Feature**: プライバシー保護のためのマスク・ぼかし処理が適用された画像を表示。

### 1.4. IconicButton
- **役割**: 設定開閉、一時停止、最小化などの操作。

### 1.5. DragHandle
- **役割**: ウィンドウ移動を可能にすることを示す視覚的な手がかり（グリップ）。
- **Behavior**: マウスホバー時のみ出現。

### 1.6. CloseButton
- **役割**: アプリケーションを終了する。
- **Behavior**: マウスホバー時のみ出現。右上の特定エリアに配置。

## 2. Molecules (複合要素)

### 2.1. SensitivitySlider
- **役割**: 「反応のよさ」調整用のスライダー。

### 2.2. DeviceSelector
- **役割**: 使用するカメラ・マイクデバイスを選択する。

### 2.3. WindowSelector
- **役割**: 入力対象とするウィンドウ（プロセス）を選択する。

## 3. Organisms (全体構造)

### 3.1. MainView
- アプリケーションの基本表示。
- `CameraPreview` を中心に配置し、その四隅または中央に `ReactionIcon` をオーバーレイ表示。

### 3.2. SettingsView
- デバイス選択、感度調整、ターゲットウィンドウ設定を統合したパネル。