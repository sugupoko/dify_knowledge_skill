# Dify Knowledge Skill Pack (Proto)

> **Proto版**: これはプロトタイプです。Difyアプリのナレッジ設計を調査しながら作成したもので、実務での効果は検証中です。「こういうアプローチがあるんだな」程度の参考としてご覧ください。

Claude Code でDifyを使ったナレッジ活用手段（チャットボット/チャットフロー/エージェント/検索型/ワークフロー等）のナレッジ設計・構築・評価を行うためのスキルパック。

RAG論文（Lewis et al.）、Lost in the Middle（Liu et al.）、RAGAS評価フレームワーク、Dify公式ドキュメント、Pineconeのチャンキングガイドなどの知見を凝縮し、**ナレッジ設計スペシャリストの思考パターンを6つに抽出してスキル化**した。

「社内FAQボットを作りたい」「業務マニュアルから回答できるようにしたい」と言われた時に、**ドキュメントを受け取ってから運用提案を出すまで**を再現性のある手順で進められる。

## こういう人向け

- 「Difyで何か作りたいけど、チャットボットが最適かも分からない」という人
- ナレッジベースは作れるが、検索精度が安定しない人
- チャットボット/チャットフロー/エージェント/検索型/ワークフローの使い分けが分からない人
- 現場からドキュメントを受け取って、動くシステムを提案する立場の人

## 何ができるか

1. **ドキュメントを受け取って、最適な手段を推奨**（2軸分析：ユーザーの状態 × 情報の構造）
2. **ナレッジの構造を設計し、仕様書（spec.md）を生成**（FAQ生成仕様、チャンキング戦略、メタデータ設計）
3. **DifyアプリをAPI or UIガイドで構築**（QAチェック：設計↔実装の突合を含む）
4. **RAG評価を実施**（検索精度・根拠性・網羅性・不可能性の判断）
5. **展開提案書を生成**（利用者・アクセス方法・運用計画・コスト見積もり）
6. **業務担当者への確認依頼書を生成**（「こう理解していますが合っていますか？」形式）
7. **継続運用の設計**（3層監視・FAQ更新サイクル・手段変更トリガー）

さらに、ヒアリングシート（汎用・FAQ特化・チャットフロー向け・エージェント向けの4種類）で**業務担当者から要件を引き出す**ことができる。

## セットアップ

```bash
git clone https://github.com/xxx/dify_knowledge_skill.git
cd dify_knowledge_skill
```

Claude Code でこのフォルダを開いて使う。

## クイックスタート

`workspace/examples/simple_faq/` にサンプルプロジェクトがあります。

```
# スキルを順に実行
/kb-assess workspace/examples/simple_faq/
/kb-design workspace/examples/simple_faq/
```

## 使い方

### 1. ヒアリング（ドキュメントを受け取る前に）

`reference/` にヒアリングシートがある。業務担当者に記入してもらう。

| シート | 対象 |
|--------|------|
| `hearing_sheet_general.md` | 汎用（チャットフロー、エージェント含む） |
| `hearing_sheet_faq.md` | FAQボット向け |
| `hearing_sheet_chatflow.md` | チャットフロー向け（分類・ルーティング・NG回答パターン） |
| `hearing_sheet_agent.md` | エージェント向け（ツール・ポリシー・暗黙のNGパターン） |

### 2. ドキュメントを受け取ったら

```bash
mkdir -p workspace/my_project/docs
cp /path/to/client_docs/* workspace/my_project/docs/
```

Claude Code で以下のスキルを順に実行する:

```
/kb-assess workspace/my_project/docs/   → 推奨手段（2軸分析） + ナレッジ種別特定
/kb-design workspace/my_project/docs/   → ナレッジ構造設計 + FAQ生成仕様 + spec.md生成
/kb-generate workspace/my_project/      → FAQ自動生成 + 品質チェック（ドキュメントそのままならスキップ）
/kb-build workspace/my_project/         → Difyアプリ構築 + QAチェック
/kb-eval workspace/my_project/          → RAG評価 + ギャップ特定 + 不可能性の判断
/kb-report workspace/my_project/        → 展開提案書
/kb-operate workspace/my_project/       → 運用設計（承認後）
```

途中で業務担当者への確認が必要になった場合:
```
/kb-request   → 確認依頼書を生成（「こう理解していますが合っていますか？」形式）
```

## ディレクトリ構成

```
dify_knowledge_skill/
├── README.md                          ← このファイル（日本語）
├── CLAUDE.md                          ← Claude Code 向けの詳細ガイド
├── CHANGELOG.md                       ← 変更履歴
├── KNOWLEDGE_DESIGN_MINDSET.md        ← 6つの思考回路 + チェックリスト
├── .claude/commands/                  ← 8つのスキル
│   ├── kb-assess.md                   ← アセスメント
│   ├── kb-design.md                   ← ナレッジ設計 + spec.md生成
│   ├── kb-generate.md                 ← FAQ / ナレッジデータ自動生成
│   ├── kb-build.md                    ← 構築 + QAチェック
│   ├── kb-eval.md                     ← 評価 + 不可能性の判断
│   ├── kb-report.md                   ← 展開提案
│   ├── kb-request.md                  ← 確認依頼書生成
│   └── kb-operate.md                  ← 運用設計
├── reference/                         ← ナレッジ設計ベストプラクティス集
│   ├── app_type_selection_guide.md    ← 手段選定ガイド（2軸フレームワーク）
│   ├── faq_design_guide.md            ← ボット向けFAQ設計
│   ├── knowledge_structure_guide.md   ← 3層ナレッジ設計
│   ├── chunking_strategy_guide.md     ← チャンキング戦略
│   ├── prompt_templates.md            ← FAQ自動生成プロンプト集
│   ├── evaluation_guide.md            ← RAG評価ガイド + 不可能性判断 + 手段比較
│   ├── hearing_sheet_general.md       ← 汎用ヒアリングシート
│   ├── hearing_sheet_faq.md           ← FAQ向けヒアリングシート
│   ├── hearing_sheet_chatflow.md      ← チャットフロー向けヒアリングシート
│   ├── hearing_sheet_agent.md         ← エージェント向けヒアリングシート
│   ├── spec_template.md               ← ナレッジ設計仕様書テンプレート
│   ├── dify_api_reference.md          ← Dify API リファレンス
│   └── state_schema.md               ← 状態管理スキーマ（バージョンフォルダ対応）
└── workspace/                         ← ここで作業する
    ├── examples/                      ← サンプルプロジェクト
    │   └── simple_faq/                ← 単純FAQボットの例
    └── my_project/                    ← プロジェクトごとにフォルダを作成
        ├── docs/                      ← ドキュメント（全バージョン共通）
        ├── v1/                        ← バージョン1の成果物一式
        │   ├── spec.md                ← ★ 仕様書（このバージョンの設計条件）
        │   ├── .kb_state.yaml
        │   ├── knowledge/
        │   └── reports/
        └── v2/                        ← バージョン2（要件変更・追加ドキュメント）
```

## 対応する手段の種類

| 手段 | 向いている用途 | ナレッジの特徴 |
|------|-------------|-------------|
| **チャットボット** | 定型FAQ、100件以下、単純Q&A | FAQ形式、言い換え耐性が命 |
| **チャットフロー** | 部門別FAQ、条件分岐、複数KB | テーマ別分割 + ルーティング設計 |
| **エージェント** | 自律調査、ツール連携、多段推論 | FAQ + 手順 + ポリシー + メタデータ |
| **検索型** | キーワードを知っているユーザー向け | FAQ形式 + セマンティック検索 |
| **プッシュ型（ワークフロー）** | 定期レポート、アラート、自動生成 | テンプレート + データ取得設計 |

→ 選び方は「ユーザーの状態」×「情報の構造」の2軸で判断（`reference/app_type_selection_guide.md` 参照）

## 5つの原則

1. **人向けFAQとボット向けFAQは別物** — そのまま入れても体験は上がらない
2. **1FAQ1テーマ、結論先出し** — 検索されて切り出されても崩れない形
3. **FAQだけでは足りない** — 支援文書と運用情報の3層で持つ
4. **評価は「自然か」ではない** — 検索精度・根拠性・網羅性を分けて測る
5. **メタデータの重要度は手段で変わる** — エージェントほど重要

## ライセンス

MIT

## 謝辞

このスキルパックは Claude Code (Claude Opus 4.6) との協働で開発し、内容は人間がチェック・編集しています。
