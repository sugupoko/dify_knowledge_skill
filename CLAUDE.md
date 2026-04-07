# CLAUDE.md — Dify Knowledge Skill Pack

## 概要

Difyを使ったナレッジ活用手段（チャットボット/チャットフロー/エージェント/検索型/プッシュ型/ワークフロー）の設計から構築・評価・展開までのワークフローをスキル化したもの。
FAQ設計、チャンキング戦略、RAG評価手法の調査から得た知見を凝縮している。

**核心**: LLMは「FAQを作る」のではなく「ナレッジの設計方針を考え、検索・判断・引き継ぎまで含めた知識基盤を設計する」ために使う。

## セットアップ

```bash
# 1. クローン
git clone https://github.com/xxx/dify_knowledge_skill.git
cd dify_knowledge_skill

# 2. このフォルダでClaude Codeを起動
claude

# 3. ドキュメントやデータを workspace/ に置いて作業開始
mkdir -p workspace/my_project
cp /path/to/client_docs/* workspace/my_project/
```

## ディレクトリ構成

```
dify_knowledge_skill/
├── CLAUDE.md                          ← このファイル（プロジェクトガイド）
├── KNOWLEDGE_DESIGN_MINDSET.md        ← 6つの思考回路 + チェックリスト
├── README.md                          ← 日本語版README
├── .claude/commands/                  ← 5つのスキル（/kb-xxx で呼び出し）
│   ├── kb-assess.md                   ← 目標・文書分析 → 推奨手段の選定
│   ├── kb-design.md                   ← ナレッジ構造設計 → FAQ生成仕様
│   ├── kb-build.md                    ← Difyアプリ構築（API or UIガイド）
│   ├── kb-eval.md                     ← 評価（検索・根拠・網羅性）
│   └── kb-report.md                   ← 展開提案書の作成
├── reference/                         ← ナレッジ設計のベストプラクティス集
│   ├── app_type_selection_guide.md    ← 手段選定ガイド（2軸）
│   ├── faq_design_guide.md            ← ボット向けFAQ設計ガイド
│   ├── knowledge_structure_guide.md   ← 3層ナレッジ設計
│   ├── chunking_strategy_guide.md     ← チャンキング戦略
│   ├── prompt_templates.md            ← FAQ生成用プロンプト集
│   ├── evaluation_guide.md            ← RAG評価ガイド
│   ├── hearing_sheet_general.md       ← ヒアリングシート（汎用）
│   ├── hearing_sheet_faq.md           ← ヒアリングシート（FAQ特化）
│   ├── dify_api_reference.md          ← Dify Console API リファレンス
│   └── state_schema.md               ← .kb_state.yaml スキーマ定義
└── workspace/                         ← ★ここで作業する
    ├── examples/                      ← サンプルプロジェクト
    │   └── simple_faq/                ← 単純FAQボットの例
    │       ├── .kb_state.yaml
    │       └── data/
    │           └── faq_sample.xlsx    ← プレースホルダー
    └── my_project/                    ← プロジェクトごとにフォルダを作成
        ├── docs/                      ← クライアントから受け取ったドキュメント
        ├── .kb_state.yaml             ← スキル間の状態管理ファイル
        ├── knowledge/                 ← 設計したナレッジデータ
        └── reports/                   ← 提案書・評価レポート
```

## 使い方

### Step 0: ヒアリング（ドキュメントを受け取る前に）
```
reference/hearing_sheet_general.md   ← 汎用ヒアリングシート
reference/hearing_sheet_faq.md       ← FAQボット向けヒアリングシート
→ 業務担当者に記入してもらい、要件を整理する。
```

### Step 1: アセスメント
```
/kb-assess workspace/my_project/docs/
→ 目標分析 → ユーザーの状態・情報の構造を分析 → 推奨手段の選定 → 必要なナレッジ種別の特定
```

### Step 2: ナレッジ設計
```
/kb-design workspace/my_project/docs/
→ FAQ生成仕様 → チャンキング戦略 → メタデータ設計 → 支援文書の特定
```

### Step 3: 構築
```
/kb-build workspace/my_project/
→ ナレッジベース作成 → データアップロード → アプリ設定（API or UIガイド）
```

### Step 4: 評価
```
/kb-eval workspace/my_project/
→ 検索テスト → 根拠性チェック → 網羅性チェック → ギャップ特定
```

### Step 5: 展開提案
```
/kb-report workspace/my_project/
→ 利用者・アクセス方法・運用計画・コスト見積もりの提案書
```

## 出力ルール（全スキル共通）

**各スキルは結果をMarkdownドキュメントとして `reports/` に保存すること。**
ナレッジデータは `knowledge/`、評価結果は `reports/` に保存する。

| スキル | 出力ファイル |
|--------|-----------|
| kb-assess | `reports/assess_report.md`（推奨手段・2軸分析含む） |
| kb-design | `reports/design_spec.md` + `knowledge/faq_draft.csv`（FAQ下書き） |
| kb-build | `reports/build_guide.md`（構築手順・設定値の記録） |
| kb-eval | `reports/eval_report.md` + `reports/eval_results.json`（評価数値） |
| kb-report | `reports/v1_proposal.md`（展開提案書、バージョン連番） |

これにより:
- 後続スキルが前のスキルの結果を参照できる
- クライアントに成果物として渡せる
- Git で変更履歴を追跡できる

## スキル一覧

| スキル | コマンド | いつ使うか |
|--------|---------|----------|
| **kb-assess** | `/kb-assess [docs]` | ドキュメントやデータを受け取った直後 |
| **kb-design** | `/kb-design [docs]` | assessの後 |
| **kb-build** | `/kb-build [project]` | designの後 |
| **kb-eval** | `/kb-eval [project]` | buildの後 |
| **kb-report** | `/kb-report [project]` | 評価が終わった後 |

## ワークフロー

```
ヒアリング → ドキュメント受領 → /kb-assess → /kb-design → /kb-build → /kb-eval → /kb-report
                                                                        ↑    ↓
                                                                        └── 改善サイクル
```

## 追加情報が来た時の進め方

ナレッジ設計は1回で終わらない。追加ドキュメントやフィードバックが来るたびにサイクルを回す。

### 何が来たかによる分岐

```
追加情報が来た
  ├── A. ヒアリングの回答
  │     「この業務はこういうルールです」「例外ケースがあります」
  │     → ナレッジを修正 → /kb-eval をもう1周
  │
  ├── B. 追加ドキュメント（マニュアル、規程、過去のQA履歴等）
  │     → /kb-assess で追加ドキュメントを分析
  │     → /kb-design でナレッジを拡充
  │     → /kb-build で再構築
  │
  ├── C. 要件の変更
  │     「エージェント型に変更したい」「ツール連携が必要になった」
  │     「チャットボットではなく検索型にしたい」「定期レポートを追加したい」
  │     → /kb-assess からやり直し
  │
  └── D. 評価フィードバック
        「この質問に答えられない」「回答が不正確」
        → /kb-eval で原因分析 → /kb-design で修正
```

## 対応する手段の種類

| 手段 | 向いている用途 | ナレッジの特徴 |
|------|-------------|-------------|
| **チャットボット** | 単純FAQ、100件以下、単一ナレッジベース | FAQ形式、言い換え耐性が重要 |
| **チャットフロー** | 分類+ルーティング、複数ナレッジベース、条件分岐 | テーマ別に分割、ルーティング設計 |
| **エージェント** | 自律判断、ツール呼び出し、多段推論 | FAQ+手順+ポリシー+メタデータ |
| **検索型** | キーワードを知っているユーザー、候補を比較したい | FAQ形式 + セマンティック検索設定 |
| **プッシュ型（ワークフロー）** | 定期レポート、アラート、ドキュメント自動生成 | テンプレート + データ取得設計 |

→ 詳細は `reference/app_type_selection_guide.md` の2軸選定フレームワークを参照

## 5つの原則

1. **人向けFAQとボット向けFAQは別物** — そのまま入れても体験は上がらない
2. **1FAQ1テーマ、結論先出し** — 検索されて切り出されても崩れない形
3. **FAQだけでは足りない** — 支援文書（手順書・ルール）と運用情報も必要
4. **評価は「自然か」ではない** — 検索精度・根拠性・網羅性を分けて測る
5. **メタデータの重要度は手段で変わる** — エージェントほど重要
