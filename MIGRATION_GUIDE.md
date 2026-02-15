# プロダクト開発移行ガイド (MIGRATION_GUIDE.md)

設計フェーズ（`kaoiro_concept`）を終了し、実装フェーズ（プロダクト開発）へ移行するための手順書。

## 1. ディレクトリ作成と初期化

現在の設計リポジトリとは別に、プロダクト用の新規ディレクトリを作成し、開発ルールを適用する。

```bash
# 1. 親ディレクトリへ移動
cd /opt2/_dev/rust

# 2. 新しいプロダクト用ディレクトリを作成
mkdir kaoiro
cd kaoiro

# 3. 設計ディレクトリからガイドラインをコピーし、GEMINI.md (憲法) として配置
cp ../kaoiro_concept/GEMINI_PRODUCT.md ./GEMINI.md

# 4. 設計ドキュメントを参照用としてコピー
#    (Geminiに参照させるだけでなく、実装リポジトリ内にも最新の仕様書として保持するため)
mkdir docs
cp ../kaoiro_concept/*.md ./docs/
```

## 2. Gemini CLI の起動

実装リポジトリ（`kaoiro`）をカレントとしつつ、設計リポジトリ（`kaoiro_concept`）も参照可能な状態で起動する。

```bash
# kaoiro ディレクトリ直下で実行
gemini-cli run . ../kaoiro_concept
```

これにより、Gemini は以下の2つのコンテキストを持つことになる。
- `.`: 実装を行うターゲット。ここにソースコードを生成する。
- `../kaoiro_concept`: 読み取り専用の設計図ライブラリ。

## 3. 開発の開始 (Initial Prompt)

Gemini 起動後、以下のプロンプトを入力して Phase 1 を開始する。

```text
GEMINI.md を読み込み、Phase 1 の開発を開始せよ。
まずは src_structure.md (../kaoiro_concept/src_structure.md) に基づき、cargo workspace の初期化を行いたい。
```
