# Gesture Analysis Engine (gesture_engine.md)

ジェスチャー認識の中核となるエンジンの設計たたき台。
モジュラー性を重視し、将来的なアルゴリズムの追加・変更（機械学習、ヒューリスティック等）に柔軟に対応できる構造を目指す。

## 1. コンセプト
- **Strategy Pattern**: 検出アルゴリズムを `Strategy` トレイトとして定義し、実行時に切り替え可能にする。
- **Pipeline Architecture**: 入力（カオアンカー / KaoAnchor）→ 前処理（スムージング）→ 特徴抽出 → 判定 → 出力（Action）の流れをパイプライン化する。
- **Stateful Analysis**: 単一フレームではなく、過去Nフレームの履歴（時系列データ）に基づいて判定を行う。

## 2. Core Traits

```rust
/// 検出されたカオアンカー (KaoAnchor) 情報
/// MediaPipe Face Detection の6点ランドマークに対応
pub struct FaceLandmarks {
    pub nose_tip: Point2D,
    pub left_eye: Point2D,
    pub right_eye: Point2D,
    pub mouth_center: Point2D,
    pub left_ear: Point2D,
    pub right_ear: Point2D,
    pub timestamp: u64,
}

/// 判定結果
#[derive(Debug, Clone, PartialEq)]
pub enum GestureAction {
    None,
    Yes(f32), // Confidence (0.0 - 1.0)
    No(f32),
    Lost,
}

/// ジェスチャー判定戦略の共通インターフェース
pub trait GestureStrategy {
    /// 新しいフレームデータを入力し、判定結果を返す
    fn process(&mut self, landmarks: &FaceLandmarks) -> GestureAction;
    
    /// 内部状態のリセット（顔を見失った直後など）
    fn reset(&mut self);
    
    /// パラメータ調整（感度など）
    fn set_sensitivity(&mut self, value: f32);
}
```

## 3. 実装候補 (Algorithms)

### 3.1. `KaoMotionStrategy` (詳細定義)

**プロトタイプでの誤検知（前後移動をYES/NOと誤認）を解決するための主力アルゴリズム。**
MediaPipe Face Detection の主要6点のカオアンカー (KaoAnchor) のみを使用し、顔の「回転成分（Pitch/Yaw）」と「移動成分（X/Y/Z）」を幾何学的に分離する。

#### A. 幾何学的モデル (Geometric Model)

1.  **基準ベクトル (Basis Vector)**:
    - 両目のベクトル $\vec{v}_{eyes} = \vec{P}_{right\_eye} - \vec{P}_{left\_eye}$
    - 目の幅 $W_{eyes} = |\vec{v}_{eyes}|$ （これを正規化の単位とする）
    - 顔の中心 $P_{center} = (\vec{P}_{left\_eye} + \vec{P}_{right\_eye}) / 2$

2.  **鼻先相対ベクトル (Nose Relative Vector)**:
    - 顔中心から鼻先へのベクトル $\vec{v}_{nose} = \vec{P}_{nose} - P_{center}$
    - 正規化ベクトル $\vec{n} = (n_x, n_y) = \vec{v}_{nose} / W_{eyes}$
    
    > **Note**: $n_y$ は顔の上下角 (Pitch) に、$n_x$ は左右角 (Yaw) に強く相関する。
    > 顔がカメラに近づいても、$W_{eyes}$ も $\vec{v}_{nose}$ も同じ比率で拡大するため、$\vec{n}$ は不変（前後移動キャンセル）。

#### B. 動的キャリブレーション (Dynamic Zero-Point Calibration)

ユーザーの「真正面」は、カメラの設置位置や姿勢、骨格によって異なるため、固定の原点 $(0, 0)$ を基準にすると誤判定の原因となる。
これを防ぐため、**「ユーザーが今、最も長く留まっている姿勢」を常に「基準（ゼロ点）」として動的に更新し続ける**。

1.  **必要性**:
    - カメラ位置（PCの上/下/斜め）や、座り方（猫背/深座り）によるオフセットを吸収する。
    - ユーザーが入れ替わったり、頬杖をつくなど姿勢を変えても、数秒で自動追従し、再設定の手間をなくす。

2.  **アルゴリズム**:
    - **ローパスフィルタ (Low-Pass Filter)**: 
        - 鼻先ベクトル $\vec{n}$ の「基準点 $\vec{n}_{base}$」を、時定数 $\tau \approx 3.0s$ 程度の遅延を持って追従させる。
        - $\vec{n}_{base}[t] = \alpha \cdot \vec{n}[t] + (1 - \alpha) \cdot \vec{n}_{base}[t-1]$
        - ここで $\alpha$ は非常に小さな値（例: 0.01 @ 60fps）とし、急激な動き（ジェスチャー）には反応させず、ゆっくりした姿勢変化のみを反映させる。
    
3.  **判定への反映**:
    - 判定に用いるのは、この基準点からの偏差 $\Delta \vec{n} = \vec{n}[t] - \vec{n}_{base}[t]$ である。
    - これにより、例えば「少し下を向いたまま（$\vec{n}_{base}$ が下にズレた状態）」で頷いても、そこからの「相対的な下方向への動き」として正しく検出される。

#### C. 判定ロジック (Intent Stabilizer)

閾値判定には **インテントスタビライザー (Intent Stabilizer / 一般呼称: ヒステリシス)** を導入し、チャタリングを防ぐ。

1.  **YES (Nod) 判定**:
    - 条件: $|\Delta n_y| > Th_{pitch\_high}$ かつ $|\Delta n_x| < Th_{yaw\_low}$
    - 解除: $|\Delta n_y| < Th_{pitch\_low}$

2.  **NO (Shake) 判定**:
    - 条件: $|\Delta n_x| > Th_{yaw\_high}$ かつ $|\Delta n_y| < Th_{pitch\_low}$
    - 解除: $|\Delta n_x| < Th_{yaw\_low}$

#### D. 擬似コード (Pseudo Code)

```rust
struct KaoMotionStrategy {
    history: VecDeque<Vec2>, // Normalized vectors
    state: State, // Idle, Nodding, Shaking
}

impl GestureStrategy for KaoMotionStrategy {
    fn process(&mut self, landmarks: &FaceLandmarks) -> GestureAction {
        // 1. Calculate Normalized Vector
        let eye_width = (landmarks.left_eye - landmarks.right_eye).length();
        let face_center = (landmarks.left_eye + landmarks.right_eye) / 2.0;
        let nose_vec = (landmarks.nose_tip - face_center) / eye_width;
        
        // 2. Update History & Calculate Average (Zero Point)
        self.history.push_back(nose_vec);
        if self.history.len() > WINDOW_SIZE { self.history.pop_front(); }
        let avg_vec = self.history.iter().sum() / self.history.len();
        
        // 3. Calculate Deviation
        let delta = nose_vec - avg_vec;
        
        // 4. Threshold Check (with State Machine)
        match self.state {
            State::Idle => {
                if delta.y.abs() > TH_PITCH_HIGH && delta.x.abs() < TH_YAW_LOW {
                    self.state = State::Nodding;
                    return GestureAction::Yes(delta.y.abs());
                }
                if delta.x.abs() > TH_YAW_HIGH && delta.y.abs() < TH_PITCH_LOW {
                    self.state = State::Shaking;
                    return GestureAction::No(delta.x.abs());
                }
            },
            State::Nodding => {
                if delta.y.abs() < TH_PITCH_LOW { self.state = State::Idle; }
            },
            State::Shaking => {
                if delta.x.abs() < TH_YAW_LOW { self.state = State::Idle; }
            }
        }
        GestureAction::None
    }
}
```

### 3.2. `MachineLearningStrategy` (将来拡張)
- **原理**: RNN/LSTM または 1D-CNN による時系列パターン認識。
- **メリット**: 複雑なジェスチャーや、個人差のある動きに対応可能。数百〜数千程度のパラメータに抑えることで、CPU負荷を最小化しリアルタイム動作を実現する。
- **デメリット**: 学習データの収集とモデルの組み込みが必要。

### 3.3. `HybridStrategy` (共存運用)
- **原理**: `KaoMotionStrategy` (3.1) と `MachineLearningStrategy` (3.2) を並列または段階的に実行し、判定精度を最大化する。
- **Tiered Approach (段階的)**: 
    - まず軽量な 3.1 でジェスチャーの候補（疑わしい動き）を検出し、その区間のみ 3.2 で詳細解析を行う。これにより、常時MLを回す負荷を回避しつつ、誤検知（眼鏡を直す動作など）を排除できる。
- **Ensemble Approach (合議制)**:
    - 両方のアルゴリズムから出力される「確信度 (Confidence Score)」を重み付け平均し、総合スコアが閾値を超えた場合のみアクションを確定する。
    - ルールベース（3.1）の弱点を学習ベース（3.2）が補い、その逆もまた然りとなる相互補完関係を構築する。

## 4. データフローと前処理

### 4.1. Pre-processing Pipeline
入力データ（Raw Landmarks）の信頼性を高めるパイプライン。

1.  **デプスニュートラライゼーション (Depth Neutralization)**: 
    - 両目の距離 $D_{eyes}$ を基準単位として全座標を無次元化する。
    - これにより、カメラに顔を近づけても座標値が発散せず、常に一定のスケールで扱える。
2.  **Smoothing (EMA)**: 
    - 指数移動平均フィルタを適用し、Webカメラ特有の微細なジッター（震え）除去する。
    - $\text{Pos}_{smooth} = \alpha \cdot \text{Pos}_{raw} + (1-\alpha) \cdot \text{Pos}_{prev}$

### 4.2. Post-processing (Debounce)
判定後のチャタリング防止。

- **Cool-down**: アクション確定後、一定時間（1.5秒）は入力を無視。
- **State Locking**: 判定中（例: YESの途中）は逆方向の判定（NO）を抑制する排他制御。

## 5. 設定パラメータ (Tunable Parameters)

- `Pitch Threshold`: 頷き検知の閾値（正規化済みベクトル変化量）。
- `Yaw Threshold`: 首振り検知の閾値。
- `Z-Axis Suppression`: 目の距離の変化率が一定以上のとき、全判定をロックする安全装置。
