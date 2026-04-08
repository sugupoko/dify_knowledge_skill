# スキル間状態管理 — .kb_state.yaml

## 目的

各スキル（/kb-assess → /kb-design → /kb-build → /kb-eval → /kb-report → /kb-operate）の出力を構造化し、次のスキルが自動的に前の結果を参照できるようにする。

## 使い方

各スキルは実行時に `workspace/<project>/v1/.kb_state.yaml` を読み書きする。
**.kb_state.yaml はバージョンフォルダ（v1/, v2/ 等）内に配置する。**

```
/kb-assess  → v1/.kb_state.yaml の assess セクションを書く
/kb-design  → assess セクションを読み、design セクションを書く（spec.md も生成）
/kb-build   → design セクションを読み、build セクションを書く
/kb-eval    → build セクションを読み、eval セクションを書く
/kb-report  → 全セクションを読み、レポートを生成
/kb-operate → 全セクションを読み、運用設計書を生成
```

## スキーマ定義

```yaml
# workspace/<project>/v1/.kb_state.yaml
version: 1                    # スキーマバージョン（スキーマ変更時に更新）
kb_version: "v1"              # プロジェクトのバージョン（v1, v2, ...）
project_name: "プロジェクト名"
created_at: "2026-04-06T10:00:00"
updated_at: "2026-04-06T15:30:00"

# --- /kb-assess の出力 ---
assess:
  status: "completed"  # pending | in_progress | completed | blocked
  completed_at: "2026-04-06T10:30:00"
  
  # 目標と背景
  goal: "社内ヘルプデスクの問い合わせ対応を自動化したい"
  target_users: "社員全員（約500名）"
  current_situation: "メールで問い合わせ、担当者が手動で回答。月100件程度"
  
  # ドキュメント分析
  documents:
    - name: "社内規程集.pdf"
      type: "policy"        # policy | manual | faq | qa_log | spreadsheet | other
      pages: 120
      language: "ja"
      quality: "良好"
    - name: "よくある質問.xlsx"
      type: "faq"
      items: 85
      language: "ja"
      quality: "要変換"      # 良好 | 要変換 | 要クリーニング | 要確認事項あり
      issues:
        - "人向けFAQ。1項目に複数論点が含まれる"
        - "条件・例外が不明確"
  
  # 手段選定（2軸）
  user_state: "聞きたいことは分かっているがキーワードを知らない"
    # 何を聞けばいいか分からない | 聞きたいことは分かっているがキーワードを知らない
    # | キーワードは分かっている | 自分のケースがどれに当たるか分からない | 聞きに来ない
  user_state_rationale: "社員が部署別に問い合わせを把握しているが、適切な窓口・言い方が分からない"
  info_structure: "条件で分岐する"
    # 1箇所で完結 | 条件で分岐する | 複数箇所を組み合わせる
  info_structure_rationale: "部門別にFAQが存在し、テーマ別ルーティングが必要"
  
  # 推奨手段
  recommended_means: "chatflow"  # chatbot | chatflow | agent | search | push | workflow
  means_rationale: "部門別に100件以上のFAQがあり、テーマ別ルーティングが必要"
  alternative_means: "chatbot"
  alternative_rationale: "まず1部門でPoCするならチャットボットで十分"
  
  # 必要なナレッジの種類
  knowledge_types:
    faq:
      needed: true
      estimated_count: 150
      source: "既存FAQ(85件)を変換 + マニュアルから生成(65件)"
    support_docs:
      needed: true
      types: ["手順書", "規程", "用語集"]
    operation_info:
      needed: true
      types: ["エスカレーションルール", "更新責任者"]
  
  # 仮定と不足情報
  assumptions:
    - key: "faq_count"
      value: 150
      confidence: "medium"
      note: "既存FAQ85件 + マニュアルから推定65件"
    - key: "query_volume"
      value: 100
      unit: "件/月"
      confidence: "high"
      note: "現在のメール問い合わせ実績"
  missing_info:
    - "部門ごとの問い合わせ内訳"
    - "よくある問い合わせのトップ20"
    - "エスカレーション基準の明文化"
  
  # 確認事項
  questions_for_stakeholder:
    - "チャットボットで答えてはいけない範囲はありますか？"
    - "既存のFAQは最新の状態ですか？"
    - "回答の根拠を表示する必要がありますか？"

# --- /kb-design の出力 ---
design:
  status: "completed"
  completed_at: "2026-04-06T12:00:00"
  
  # ナレッジベース設計
  knowledge_bases:
    - name: "総務FAQ"
      type: "faq"
      estimated_items: 60
      source_docs: ["社内規程集.pdf", "よくある質問.xlsx"]
      chunking:
        strategy: "1faq_1chunk"
        target_tokens: "128-512"
    - name: "IT FAQ"
      type: "faq"
      estimated_items: 50
      source_docs: ["IT手順書.docx"]
      chunking:
        strategy: "1faq_1chunk"
        target_tokens: "128-512"
    - name: "用語集"
      type: "glossary"
      estimated_items: 30
      source_docs: ["社内規程集.pdf"]
      chunking:
        strategy: "1term_1chunk"
        target_tokens: "64-256"
  
  # FAQ生成仕様
  faq_generation:
    method: "two_step"  # simultaneous | two_step | conversion
    prompt_template: "reference/prompt_templates.md#パターンB"
    review_required: true
    output_format: "csv"
    output_file: "knowledge/faq_draft.csv"
  
  # メタデータ設計
  metadata:
    fields:
      - name: "category"
        description: "部門カテゴリ"
        values: ["総務", "IT", "経理", "人事"]
      - name: "source_doc"
        description: "参照元文書名"
      - name: "last_updated"
        description: "最終更新日"
  
  # ルーティング設計（チャットフロー用）
  routing:
    method: "question_classifier"
    categories: ["総務", "IT", "経理", "人事", "その他"]
    fallback: "「担当部署にお問い合わせください」"
  
  # プロンプト設計
  prompt_design:
    system_prompt: |
      あなたは社内ヘルプデスクのアシスタントです。
      ナレッジベースの情報に基づいて回答してください。
      該当する情報がない場合は「担当部署にお問い合わせください」と回答してください。
    constraints:
      - "ナレッジベースの情報のみを使う"
      - "3-5文で簡潔に回答"
      - "人事評価や給与の具体的な数字は回答しない"

# --- /kb-build の出力 ---
build:
  status: "completed"
  completed_at: "2026-04-06T14:00:00"
  
  # 構築方法
  method: "ui_guide"  # api | ui_guide | hybrid
  
  # Dify設定
  dify_config:
    app_type: "chatflow"
    app_name: "社内ヘルプデスクBot"
    model: "gpt-4o-mini"
    knowledge_bases:
      - name: "総務FAQ"
        status: "created"
        document_count: 60
        retrieval_mode: "semantic"
        top_k: 3
        score_threshold: 0.5
      - name: "IT FAQ"
        status: "created"
        document_count: 50
        retrieval_mode: "semantic"
        top_k: 3
        score_threshold: 0.5
  
  # 構築手順の記録
  steps_completed:
    - "ナレッジベース作成"
    - "FAQデータアップロード"
    - "アプリ作成"
    - "プロンプト設定"
    - "ナレッジベース接続"
  
  build_guide_file: "reports/build_guide.md"

# --- /kb-eval の出力 ---
eval:
  status: "completed"
  completed_at: "2026-04-06T15:00:00"
  
  # テスト結果
  test_queries: 30
  results:
    retrieval:
      accuracy: 0.83      # top_kに正解が含まれる割合
      details: "25/30 正しいFAQをtop_3に含む"
    groundedness:
      score: 0.90         # 回答がFAQに基づいている割合
      details: "27/30 FAQの情報のみで回答"
    completeness:
      score: 0.73         # 条件・例外が含まれている割合
      details: "22/30 必要な条件を全て含む"
    resolution:
      estimated: 0.70     # 推定解決率
      note: "実運用データなし、テスト質問での推定"
  
  # ギャップ分析
  gaps:
    - type: "retrieval_miss"
      description: "IT関連の略語で検索すると正しいFAQが拾えない"
      count: 3
      fix: "IT用語の言い換え・別名を追加"
    - type: "completeness_miss"
      description: "経理FAQで例外条件の記載が不足"
      count: 5
      fix: "経理FAQに例外条件を追記"
  
  # 改善提案
  improvements:
    - priority: "high"
      action: "IT用語の言い換え辞書を追加"
      expected_impact: "retrieval +10%"
    - priority: "medium"
      action: "経理FAQの例外条件を追記"
      expected_impact: "completeness +15%"
  
  eval_report_file: "reports/eval_report.md"
  
  # 不可能性の判断（手段変更トリガー）
  impossibility_findings:
    - finding: "Hit Rate@3 が70%で改善の余地なし"
      implication: "チャットボットでは精度目標を達成できない"
      recommendation: "チャットフローへの移行を検討"
    # 該当なしの場合: []

# --- /kb-report の出力 ---
report:
  status: "completed"
  completed_at: "2026-04-06T16:00:00"
  
  output_file: "reports/v1_proposal.md"
  
  # 展開提案
  deployment:
    target_users: "総務部・IT部（第1フェーズ）"
    access_method: "社内ポータルにウィジェット埋め込み"
    launch_date: "2026-05-01"
  
  # 運用計画
  maintenance:
    faq_review_cycle: "月次"
    responsible_team: "DX推進室"
    update_trigger:
      - "新しい規程・制度の追加"
      - "問い合わせが多いトピックの追加"
      - "解決率が70%を下回った場合"
  
  # コスト見積もり
  cost_estimate:
    initial:
      description: "初期構築（ナレッジ設計 + Dify設定）"
      person_days: 5
    monthly:
      description: "月次メンテナンス + API利用料"
      api_cost: "約5,000円/月（gpt-4o-mini、月100件想定）"
      maintenance_hours: 4
  
  # 提案の優先順位
  proposals:
    - id: "A"
      name: "第1フェーズ: 総務+IT FAQ（チャットフロー）"
      cost: "5人日 + 月5,000円"
      effect: "問い合わせの50%を自動化（推定）"
      difficulty: "低（すぐ開始可能）"
      recommended: true
    - id: "B"
      name: "第2フェーズ: 全部門展開 + エージェント化"
      cost: "15人日 + 月15,000円"
      effect: "問い合わせの80%を自動化（推定）"
      difficulty: "中（3ヶ月後目標）"
      recommended: false
```

## ルール

### 読み書きのルール
1. 各スキルは自分のセクション**のみ**を書き込む
2. 前のスキルのセクションは**読み取り専用**
3. `.kb_state.yaml` がなければ新規作成（/kb-assess が最初に作る）
4. 既存のセクションがあれば上書き（同じスキルを再実行した場合）

### status の遷移
```
pending → in_progress → completed
                      → blocked（追加情報待ち）
```

### バージョン管理との連携
```
追加ドキュメントや制約変更が来た場合:
  1. 新しいバージョンフォルダ（v2/）を作成
  2. 前バージョンの spec.md を複製して更新
  3. 新しい v2/.kb_state.yaml で /kb-assess から開始
  → v1/.kb_state.yaml と v2/.kb_state.yaml で Before/After の比較が可能
  → v1/spec.md と v2/spec.md を比較すれば変更点がわかる
```

## スキルでの参照方法

各スキルのコマンドファイルに以下の指示を含める:

```
## 状態の読み込み
実行開始時に workspace/<project>/.kb_state.yaml を読み込む。
前のスキルの出力（assess, design 等）を参照して、コンテキストを引き継ぐ。

## 状態の書き込み
実行完了時に自分のセクションを .kb_state.yaml に書き込む。
```
