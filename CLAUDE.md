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
├── .claude/commands/                  ← 8つのスキル（/kb-xxx で呼び出し）
│   ├── kb-assess.md                   ← 目標・文書分析 → 推奨手段の選定
│   ├── kb-design.md                   ← ナレッジ構造設計 → FAQ生成仕様 + spec.md生成
│   ├── kb-generate.md                 ← FAQ / ナレッジデータの自動生成
│   ├── kb-build.md                    ← Difyアプリ構築（API or UIガイド）+ QAチェック
│   ├── kb-eval.md                     ← 評価（検索・根拠・網羅性）+ 不可能性判断
│   ├── kb-report.md                   ← 展開提案書の作成
│   ├── kb-request.md                  ← 確認依頼書の生成（業務担当者向け）
│   └── kb-operate.md                  ← 運用設計（監視・更新・フォールバック）
├── reference/                         ← ナレッジ設計のベストプラクティス集
│   ├── app_type_selection_guide.md    ← 手段選定ガイド（2軸）
│   ├── faq_design_guide.md            ← ボット向けFAQ設計ガイド
│   ├── knowledge_structure_guide.md   ← 3層ナレッジ設計
│   ├── chunking_strategy_guide.md     ← チャンキング戦略
│   ├── prompt_templates.md            ← FAQ生成用プロンプト集
│   ├── evaluation_guide.md            ← RAG評価ガイド + 不可能性判断基準 + 手段比較テーブル
│   ├── hearing_sheet_general.md       ← ヒアリングシート（汎用）
│   ├── hearing_sheet_faq.md           ← ヒアリングシート（FAQ特化）
│   ├── hearing_sheet_chatflow.md      ← ヒアリングシート（チャットフロー向け）
│   ├── hearing_sheet_agent.md         ← ヒアリングシート（エージェント向け）
│   ├── dify_api_reference.md          ← Dify Console API リファレンス
│   ├── spec_template.md               ← ナレッジ設計仕様書テンプレート（プロジェクトの「今の正」）
│   └── state_schema.md               ← .kb_state.yaml スキーマ定義
└── workspace/                         ← ★ここで作業する
    ├── examples/                      ← サンプルプロジェクト
    │   └── simple_faq/                ← 単純FAQボットの例
    │       ├── v1/
    │       │   ├── spec.md            ← ★ 仕様書（このバージョンの条件）
    │       │   ├── .kb_state.yaml     ← スキル間の状態管理ファイル
    │       │   ├── knowledge/         ← 設計したナレッジデータ
    │       │   └── reports/           ← 提案書・評価レポート
    │       └── data/
    │           └── faq_sample.xlsx    ← プレースホルダー
    └── my_project/                    ← プロジェクトごとにフォルダを作成
        ├── docs/                      ← クライアントから受け取ったドキュメント
        ├── v1/                        ← バージョンごとに一式まとまる
        │   ├── spec.md                ← ★ 仕様書（このバージョンの条件）
        │   ├── .kb_state.yaml         ← スキル間の状態管理ファイル
        │   ├── knowledge/             ← 設計したナレッジデータ
        │   └── reports/               ← 提案書・評価レポート
        └── v2/                        ← 追加ドキュメントや要件変更で新バージョン
            ├── spec.md                ← 更新された仕様書（変更点を記載）
            ├── .kb_state.yaml
            ├── knowledge/
            └── reports/
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
→ FAQ生成仕様 → チャンキング戦略 → メタデータ設計 → 支援文書の特定 → spec.md生成
```

### Step 3: ナレッジデータ生成
```
/kb-generate workspace/my_project/
→ spec.mdに従ってFAQを自動生成 → 品質チェック → レビュー用に出力
  （ドキュメントそのまま投入の場合はスキップ）
```

### Step 4: 構築
```
/kb-build workspace/my_project/
→ ナレッジベース作成 → データアップロード → アプリ設定（API or UIガイド）→ QAチェック
```

### Step 5: 評価
```
/kb-eval workspace/my_project/
→ 検索テスト → 根拠性チェック → 網羅性チェック → ギャップ特定 → 不可能性判断
```

### Step 6: 展開提案
```
/kb-report workspace/my_project/
→ 利用者・アクセス方法・運用計画・コスト見積もりの提案書
```

## 出力ルール（全スキル共通）

**全成果物はバージョンフォルダ（`v1/`, `v2/`, ...）内に出力する。**
最新バージョンの `spec.md` が「今の正」。テンプレートは `reference/spec_template.md` を参照。

| スキル | 出力ファイル（vN/ 内） |
|--------|-----------|
| kb-assess | `reports/assess_report.md`（推奨手段・2軸分析含む） |
| kb-design | `spec.md`（初版生成） + `reports/design_report.md` |
| kb-generate | `knowledge/faq_draft.csv` + `knowledge/generation_log.md` |
| kb-build | `reports/build_guide.md`（構築手順・設定値・QAチェック結果） |
| kb-eval | `reports/eval_report.md` + `reports/eval_results.json`（不可能性フィンディング含む） |
| kb-report | `reports/proposal.md`（展開提案書） |
| kb-request | `reports/request.md`（確認依頼書） |
| kb-operate | `reports/operate_design.md`（運用設計書） |

これにより:
- **spec.md を見れば「今どの条件で設計しているか」が常にわかる**
- バージョン間の spec.md を比較すれば変更点がわかる
- 各バージョンが独立しているので、いつでも再実行できる
- クライアントに成果物として渡せる

## スキル一覧

| スキル | コマンド | いつ使うか |
|--------|---------|----------|
| **kb-assess** | `/kb-assess [docs]` | ドキュメントやデータを受け取った直後 |
| **kb-design** | `/kb-design [docs]` | assessの後 |
| **kb-generate** | `/kb-generate [project]` | designの後（FAQ生成。ドキュメントそのまま投入ならスキップ） |
| **kb-build** | `/kb-build [project]` | generateの後 |
| **kb-eval** | `/kb-eval [project]` | buildの後 |
| **kb-report** | `/kb-report [project]` | 評価が終わった後 |
| **kb-request** | `/kb-request` | 途中で業務担当者への確認が必要な時 |
| **kb-operate** | `/kb-operate [project]` | 提案承認後、運用設計する時 |

## ワークフロー

```
ヒアリング → ドキュメント受領 → /kb-assess → /kb-design → /kb-generate → /kb-build → /kb-eval → /kb-report → /kb-operate
                                                                          ↑    ↓            ↑    ↓
                                                                          └── /kb-request ←─── 改善サイクル
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

## バージョンフォルダ管理

バージョンごとに一式をまとめる。spec.md が各バージョンの仕様書。

```
workspace/my_project/
├── docs/                   ← クライアントから受け取ったドキュメント（全バージョン共通）
├── v1/
│   ├── spec.md             ← v1 の仕様書（このバージョンの設計条件）
│   ├── .kb_state.yaml      ← v1 のスキル間状態管理
│   ├── knowledge/          ← v1 のナレッジデータ（FAQ CSV等）
│   └── reports/            ← v1 の全レポート
├── v2/
│   ├── spec.md             ← v2 の仕様書（v1 からの変更点を記載）
│   ├── .kb_state.yaml
│   ├── knowledge/
│   └── reports/
└── ...
```

前のバージョンを残す理由:
- v1 と v2 の spec.md を比較すれば「何が変わったか」がわかる
- 各バージョンの eval_report.md を比較すれば Before/After が出せる
- クライアントに変化の経緯を説明できる

### Git ブランチ命名規則

```bash
# プロジェクト開始時: ブランチを作成
git checkout -b kb/<project>/v1-initial

# 作業が一段落したら: タグで結果を記録
git tag kb/<project>/v1 -m "初回設計: FAQ150件, Hit Rate@3=0.83"

# 追加ドキュメントや要件変更が来た時: 新しいブランチを作成
git checkout -b kb/<project>/v2-add-chatflow

# Before/After の確認
git diff kb/<project>/v1..kb/<project>/v2 -- workspace/
```

**命名規則:**
- ブランチ: `kb/<project>/<version>-<description>`
- タグ: `kb/<project>/<version>`
- タグメッセージ: 主要な評価結果の数値を含める

---

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
