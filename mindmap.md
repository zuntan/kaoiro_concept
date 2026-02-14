# プロジェクト・マインドマップ (mindmap.md)

`kaoiro` プロジェクトの全貌を俯瞰するためのマインドマップ。

```mermaid
graph LR
  kaoiro((kaoiro))

  subgraph Core_Vision_and_Concepts [コア_ビジョンとコンセプト]
    Core_Vision_and_Concepts_Main["コア_ビジョンとコンセプト"]
    Core_Vision_and_Concepts_Main --> Vision["ビジョン: Casual AI Communication"]
    Core_Vision_and_Concepts_Main --> Avatar["アバター: 状態の可視化"]
    Core_Vision_and_Concepts_Main --> KaoMotion_Intent["KaoMotion: 意図の発見"]
    Core_Vision_and_Concepts_Main --> KaoAnchor_Base["KaoAnchor: 構造的基盤"]
  end

  subgraph Architecture_and_Platforms [アーキテクチャとプラットフォーム]
    Architecture_and_Platforms_Main["アーキテクチャとプラットフォーム"]
    Architecture_and_Platforms_Main --> Desktop_Version["デスクトップ版"]
    Desktop_Version --> Single_Binary["単一バイナリ"]
    Desktop_Version --> ort_Inference["ortによる推論"]
    Desktop_Version --> Vosk_Voice["Voskによる音声認識"]
    Desktop_Version --> Enigo_Input["Enigoによる入力"]
    Desktop_Version --> MCP_Server["MCPサーバー"]

    Architecture_and_Platforms_Main --> Browser_Extension["ブラウザ拡張機能版"]
    Browser_Extension --> WASM_tract["WASM_tract"]
    Browser_Extension --> Web_Worker["Web_Worker"]
    Browser_Extension --> Web_Speech_API["Web_Speech_API"]
  end

  subgraph KaoMotion_Engine [KaoMotion_Engine（カオモーション・エンジン）]
    KaoMotion_Engine_Main["KaoMotion_Engine（カオモーション・エンジン）"]
    KaoMotion_Engine_Main --> Strategy["判定戦略: KaoMotionStrategy"]
    KaoMotion_Engine_Main --> Pre_processing["前処理"]
    Pre_processing --> Depth_Neutralization["Depth_Neutralization（デプスニュートラライゼーション）"]
    Pre_processing --> Dynamic_Calibration["Dynamic_Calibration（動的キャリブレーション）"]
    KaoMotion_Engine_Main --> Decision_Logic["決定ロジック"]
    Decision_Logic --> Intent_Stabilizer["Intent_Stabilizer（インテントスタビライザー）"]
    KaoMotion_Engine_Main --> Future_Expansion["将来の拡張"]
    Future_Expansion --> Hybrid_Strategy_ML["Hybrid_Strategy_ML"]
  end

  subgraph UI_UX_and_Design [UI_UXとデザイン]
    UI_UX_and_Design_Main["UI_UXとデザイン"]
    UI_UX_and_Design_Main --> Theme["テーマ: Gadget_and_Avatar"]
    UI_UX_and_Design_Main --> Layout["レイアウト"]
    Layout --> MainContent["メインコンテンツ"]
    Layout --> Privacy_View_PiP["プライバシービュー_PiP"]
    Layout --> SettingsPanel["設定パネル"]
    UI_UX_and_Design_Main --> States["状態"]
    States --> Idle_YES_NO["アイドル_YES_NO"]
    States --> Lost_Paused["ロスト_ポーズ"]
  end

  subgraph Roadmap_and_Strategy [ロードマップと戦略]
    Roadmap_and_Strategy_Main["ロードマップと戦略"]
    Roadmap_and_Strategy_Main --> Phases["フェーズ"]
    Phases --> Phase_1["Phase_1: GUI_モック"]
    Phases --> Phase_2["Phase_2: デスクトップ版_MVP"]
    Phases --> Phase_3["Phase_3: 拡張機能版_MVP"]
    Phases --> Phase_4["Phase_4: 音声連携"]
    Phases --> Phase_5["Phase_5: AI_エコシステム"]
    Roadmap_and_Strategy_Main --> Key_Issues["主要課題"]
    Key_Issues --> Performance_Optimization["パフォーマンス最適化"]
    Key_Issues --> Privacy_Protection["プライバシー保護"]
    Key_Issues --> OS_Permissions["OS_権限管理"]
  end

  kaoiro --> Core_Vision_and_Concepts_Main
  kaoiro --> Architecture_and_Platforms_Main
  kaoiro --> KaoMotion_Engine_Main
  kaoiro --> UI_UX_and_Design_Main
  kaoiro --> Roadmap_and_Strategy_Main

