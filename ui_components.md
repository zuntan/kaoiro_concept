# UIコンポーネント (ui_components.md)

`egui` (Rust) を用いたコンポーネント設計。
`app_looks.md` のデザイン定義および `wireframe.md` のレイヤー構造に基づく。

## 1. Atoms (基本要素)

### 1.1. `AvatarWidget`
- **役割**: `egui_extras::RetainedImage` または `egui::Shape` を用いてSVGアバターを描画。
- **Props**:
  - `set`: `AvatarSet` (Face, Dog, Cat, etc.)
  - `emotion`: `Emotion` (Idle, Smile, Sad, Sleeping, Search)
- **Visual**: 線画SVG。ステート変化時に `f32` ベースの補間値で表情をアニメーションさせる。

### 1.2. `GlowBackground`
- **役割**: ウィンドウ全体に光彩（Backdrop Glow）を描画する。
- **Props**:
  - `color`: `egui::Color32`
  - `intensity`: `f32`
- **Impl**: `egui::Painter::rect_filled` に透明度グラデーションを適用、またはカスタムシェーダー。

### 1.3. `ActionRing`
- **役割**: アバター周囲の進行状況円環。
- **Props**:
  - `progress`: `f32` (0.0 - 1.0)
  - `style`: `LineStyle` (Solid, Dashed)
- **Visual**: `Lost` 状態では破線が回転し、認識確定時はゲージが溜まる演出。

### 1.4. `CameraThumbnail` (PiP)
- **役割**: 処理済みカメラフレームの表示。
- **Props**:
  - `texture`: `egui::TextureHandle`
- **Visual**: 小さな矩形または円形にクリップされた映像。プライバシーフィルタ適用済み。

### 1.5. `StatusLabel`
- **役割**: 判定テキストの表示。
- **Props**:
  - `text`: `&str`
  - `opacity`: `f32`
- **Visual**: `egui::RichText` による大きなフォント。

### 1.6. `GlassButton`
- **役割**: ホバー時に浮かび上がる操作ボタン。
- **Props**:
  - `icon`: `Icon` (Close, Settings, Mute)
  - `action`: `Callback`
- **Visual**: 通常は非表示または高透明度。ホバーで不透明度・背景色が変化。アイコン描画には `egui-phosphor` を使用。

### 1.7. `MicrophoneIndicator`
- **役割**: マイクの稼働状態と入力レベルを表示。
- **Props**:
  - `state`: `MicState` (Idle, Listening, Processing, Error)
  - `level`: `f32` (0.0 - 1.0)
- **Visual**: 波形アニメーション (Visualizer) または色変化するマイクアイコン。

## 2. Molecules (複合要素)

### 2.1. `MainContent`
- **構成**:
  - `GlowBackground`
  - `AvatarWidget`
  - `ActionRing`
- **役割**: アプリケーションのメインビジュアル。ステート駆動で各AtomsのPropsを制御。

### 2.2. `HoverControls`
- **構成**:
  - `MuteToggle` (Button)
  - `SettingsButton` (Button)
  - `CloseButton` (Button)
  - `DragHandle` (Area)
- **役割**: マウスホバー時のみ出現する操作系UI。

### 2.3. `AvatarGallery`
- **構成**:
  - `egui::ScrollArea`
  - `AvatarWidget` (Preview size)
- **役割**: 設定パネル内でのアバター選択リスト。

## 3. Organisms (画面全体 / 構造)

### 3.1. `KaoiroMainWindow`
- **構成**:
  - `MainContent` (Layer 1-4)
  - `StatusLabel` (Layer 5)
  - `CameraThumbnail` (Layer 5)
  - `HoverControls` (Layer 6)
- **役割**: `egui::CentralPanel` 内で全レイヤーを重ね合わせ描画するルートコンポーネント。

### 3.2. `SettingsOverlay`
- **構成**:
  - `egui::Window`
  - `DeviceSelector` (Molecules)
  - `SensitivitySlider` (Molecules)
  - `AvatarGallery`
- **役割**: 設定管理画面。
