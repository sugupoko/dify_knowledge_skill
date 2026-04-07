# 構築ガイド — 社内ITヘルプデスクチャットボット

> 作成日: 2026-04-06
> 前提: design_report.md の設計仕様に基づく
> 環境: Dify Self-hosted + Ollama（ローカルLLM）
> 注意: 本ガイドはサンプルプロジェクトのため、実際のAPI呼び出しは行わない。何をどの順でやるかの構築手順書として作成。

---

## 構築方法

| 項目 | 内容 |
|------|------|
| 構築方法 | ハイブリッド（ナレッジベース作成・FAQアップロードはAPI、アプリ設定・プロンプト調整はUI） |
| Dify環境 | Self-hosted（オンプレ）|
| LLM | Ollama（ローカルLLM。推奨モデル: qwen2.5:14b または llama3.1:8b 日本語対応） |

---

## Phase 1: 事前準備

### 1.1 FAQデータの最終確認

`data/generated_faq.md` のFAQ 70件を確認する。

```bash
# FAQファイルの行数確認（70件 × ヘッダー等で80行前後）
wc -l data/generated_faq.md

# TSV形式への変換（Dify Knowledge APIに投入しやすい形式）
# 1FAQ = 1ドキュメントとして扱う
```

チェックリスト:
- [ ] 全70件が揃っているか（No.1〜70）
- [ ] タブ区切り9列（No/カテゴリ/質問/回答/条件例外/次の行動/エスカレーション/参照元/言い換え）
- [ ] カテゴリ別件数: VPN(8件)、パスワード(9件)、PC(9件)、ソフトウェア(7件)、メール(8件)、プリンター(4件)、セキュリティ(9件)、申請(4件)、クラウド(6件)、障害(6件)

### 1.2 Dify接続確認

```bash
# Dify Self-hostedの接続確認
# Dify管理画面URL（例）: http://localhost:3000 または http://your-dify-server:3000

# APIキーの確認: Dify管理画面 → 設定 → APIキー
DIFY_API_URL="http://localhost/v1"
DIFY_API_KEY="your-api-key-here"

# 接続確認
curl -H "Authorization: Bearer ${DIFY_API_KEY}" \
     "${DIFY_API_URL}/datasets"
```

### 1.3 Ollamaモデルの確認

```bash
# Ollamaが動作しているか確認
curl http://localhost:11434/api/tags

# 推奨モデルのダウンロード（日本語対応）
# 選択肢1: qwen2.5:14b（日本語強、14Bクラス）
ollama pull qwen2.5:14b

# 選択肢2: llama3.1:8b（バランス型、8Bクラス）
ollama pull llama3.1:8b

# 選択肢3: elyza:jp8b（日本語特化）
ollama pull elyza:jp8b

# Dify管理画面でOllamaモデルを登録:
# 設定 → モデルプロバイダー → Ollama → ホスト: http://localhost:11434
```

---

## Phase 2: ナレッジベースの構築

### 2.1 ナレッジベースの作成（API）

design_report.md の設計に従い、1つの統合ナレッジベースを作成する。

```bash
# ナレッジベース作成（Dify Console API）
curl -X POST "${DIFY_API_URL}/datasets" \
  -H "Authorization: Bearer ${DIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "ITヘルプデスクFAQ",
    "description": "社内ITヘルプデスク向けFAQ。VPN、パスワード、PC、ソフトウェア、メール、セキュリティ等70件のFAQを収録。",
    "indexing_technique": "high_quality",
    "permission": "only_me"
  }'

# レスポンス例:
# {"id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "name": "ITヘルプデスクFAQ", ...}
# → dataset_id を控えておく
```

### 2.2 FAQデータのアップロード（API — 1FAQ 1ドキュメント方式）

generated_faq.md の各FAQを1件ずつドキュメントとしてアップロードする。
Ollama環境では1チャンク 128〜256トークンが推奨なので、1FAQ=1ドキュメントが最適。

```bash
DATASET_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"  # 上で取得したID

# FAQ 1件のアップロード例（FAQ No.1: VPN接続方法）
curl -X POST "${DIFY_API_URL}/datasets/${DATASET_ID}/document/create_by_text" \
  -H "Authorization: Bearer ${DIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "FAQ-001_VPN接続方法",
    "text": "【カテゴリ】VPN・ネットワーク\n【質問】VPNの接続方法を教えてください\n【回答】GlobalProtectアプリを起動し、ポータルアドレス「vpn.example.co.jp」を入力、社員IDとパスワードでログイン後「接続」をクリック。ステータスが「接続済み」になれば完了です。\n【条件/例外】社員ID・パスワードが有効であること。GlobalProtectアプリが事前にインストールされていること。\n【次の行動】GlobalProtectアプリを起動して手順通りに接続する\n【エスカレーション】接続できない場合はヘルプデスク（内線9999）に連絡\n【参照元】it_manual.md 1.1\n【言い換えキーワード】GlobalProtect、GP、リモートアクセス、テレワーク接続、VPN繋ぎ方",
    "indexing_technique": "high_quality",
    "process_rule": {
      "mode": "custom",
      "rules": {
        "pre_processing_rules": [
          {"id": "remove_extra_spaces", "enabled": true},
          {"id": "remove_urls_emails", "enabled": false}
        ],
        "segmentation": {
          "separator": "\n\n",
          "max_tokens": 256
        }
      }
    },
    "doc_metadata": {
      "category": "VPN・ネットワーク",
      "faq_no": "001",
      "source": "it_manual.md 1.1"
    }
  }'
```

**70件一括投入スクリプト（bash）の設計:**

```bash
#!/bin/bash
# upload_faqs.sh — generated_faq.md から全70件をDify Knowledgeに投入するスクリプト

DIFY_API_URL="http://localhost/v1"
DIFY_API_KEY="your-api-key-here"
DATASET_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
FAQ_FILE="data/generated_faq.md"

# FAQデータをTSVから1件ずつ処理
# ※ TSVのヘッダー行を除く2行目以降を処理
tail -n +2 "${FAQ_FILE}" | grep "^[0-9]" | while IFS=$'\t' read -r no category question answer conditions next_action escalation source aliases; do
  # ドキュメント名を生成
  doc_name="FAQ-$(printf '%03d' $no)_${category}_${question:0:20}"
  
  # テキストコンテンツを組み立て
  text_content="【カテゴリ】${category}\n【質問】${question}\n【回答】${answer}\n【条件/例外】${conditions}\n【次の行動】${next_action}\n【エスカレーション】${escalation}\n【参照元】${source}\n【言い換えキーワード】${aliases}"
  
  # Dify APIに投入
  curl -s -X POST "${DIFY_API_URL}/datasets/${DATASET_ID}/document/create_by_text" \
    -H "Authorization: Bearer ${DIFY_API_KEY}" \
    -H "Content-Type: application/json" \
    -d "{
      \"name\": \"${doc_name}\",
      \"text\": \"${text_content}\",
      \"indexing_technique\": \"high_quality\",
      \"doc_metadata\": {
        \"category\": \"${category}\",
        \"faq_no\": \"${no}\"
      }
    }"
  
  echo "FAQ No.${no} アップロード完了"
  sleep 0.5  # APIレート制限対策
done

echo "全70件のFAQアップロードが完了しました"
```

### 2.3 インデクシングの確認

```bash
# ナレッジベース内のドキュメント一覧を確認
curl "${DIFY_API_URL}/datasets/${DATASET_ID}/documents?page=1&limit=100" \
  -H "Authorization: Bearer ${DIFY_API_KEY}"

# 各ドキュメントのindexing_statusを確認
# indexing_status: "waiting" → "parsing" → "indexing" → "completed"

# 全ドキュメントが "completed" になるまで待つ（通常5〜20分）
# 70件の場合、Ollamaのembedding処理で10〜30分程度かかる場合あり
```

確認事項:
- [ ] 70件全てのドキュメントが `indexing_status: completed` になっていること
- [ ] Segment数が想定通りか（1FAQ = 1〜2セグメントが理想）
- [ ] サンプルのセグメント（例: FAQ-001）を開いてQ&Aペアが途中で切れていないこと
- [ ] メタデータ（category、faq_no）が正しく付与されていること

---

## Phase 3: チャットボットアプリの構築（UI）

### 3.1 アプリの作成

```
Dify管理画面での手順:
  1. 左メニュー → 「スタジオ」→ 「アプリを作成」
  2. 「チャットボット」を選択
  3. アプリ名: 「社内ITヘルプデスクBot」
  4. 説明: 「VPN、パスワード、PC、セキュリティなど社内ITの質問に24時間対応します」
  5. 「作成」をクリック
```

### 3.2 モデル設定（UI）

```
アプリ設定画面:
  1. 右上の「モデル設定」をクリック
  2. モデルプロバイダー: Ollama
  3. モデル: qwen2.5:14b（または選択したモデル）
  4. パラメーター設定:
     - Temperature: 0.2（低めに設定。ハルシネーション抑制）
     - Max Tokens: 512（回答が長くなりすぎない範囲）
     - Top P: 0.9
```

### 3.3 システムプロンプトの設定（UI）

```
「システムプロンプト」欄に以下を貼り付け:

---
あなたは社内ITヘルプデスクのアシスタントです。
ナレッジベースに登録された情報に基づいて、従業員のIT関連の質問に回答してください。

## 回答ルール
- ナレッジベースの情報のみに基づいて回答してください
- 回答は最初の1-2文で結論を述べてください
- 条件や例外がある場合は必ず明示してください
- 手順がある場合はステップ番号を付けてください
- ナレッジベースに該当情報がない場合は「この質問については、ヘルプデスク（内線9999）にお問い合わせください」と案内してください

## 緊急時の対応
以下のキーワードが含まれる場合は、通常の回答ではなく即座にエスカレーション案内をしてください:
- ランサムウェア / ウイルス / 不正アクセス → 「今すぐLANケーブルを抜いて、ヘルプデスク（内線9999）に電話してください。絶対にPCをシャットダウンしないでください。」
- 情報漏洩 / 誤送信 → 「すぐに上長に報告し、情報セキュリティ部門（noc@example.co.jp）に連絡してください。」
- システム全体障害 → 「ヘルプデスク（内線9999）に至急電話してください。」

## 禁止事項
- 個人のパスワードを聞き出す回答はしないでください
- 法的な判断・助言はしないでください
- 人事・給与に関する質問には回答せず、人事部への問い合わせを案内してください
- ナレッジベースにない情報を推測で回答しないでください
---
```

### 3.4 ナレッジベースの接続（UI）

```
アプリ設定画面:
  1. 「コンテキスト」セクション → 「追加」をクリック
  2. 作成した「ITヘルプデスクFAQ」を選択
  3. Retrieval設定:
     - Retrieval Mode: Hybrid（ベクトル + 全文検索）
     - Top K: 5（FAQ 70件規模では5で十分）
     - Score Threshold: 0.5（低すぎず高すぎない初期値）
     - Reranking: 有効（利用可能な場合。Ollamaモデルで対応している場合のみ）
  4. 「保存」をクリック
```

### 3.5 開始メッセージとサジェスト質問の設定（UI）

```
「機能」セクション:
  1. 開始メッセージ:
     「ITヘルプデスクアシスタントです。
     VPN、パスワード、PC、メール、ソフトウェアなどに関する質問にお答えします。
     緊急のインシデント（ランサムウェア、システム障害等）は内線9999に直接電話してください。」

  2. サジェスト質問（3件）:
     - 「VPNに接続できないのですが」
     - 「パスワードを忘れました」
     - 「新しいソフトウェアを申請したい」

  3. フォールバックメッセージ:
     「申し訳ありませんが、その質問についての情報がありません。
     ヘルプデスク（内線9999 / helpdesk@example.co.jp）にお問い合わせください。」
```

### 3.6 Dify APIによるアプリ設定（代替手順）

UIが使えない場合や設定を自動化したい場合は、以下のAPIで設定する。

```bash
# チャットボットアプリ作成
curl -X POST "${DIFY_API_URL}/apps" \
  -H "Authorization: Bearer ${DIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "社内ITヘルプデスクBot",
    "description": "VPN、パスワード、PC、セキュリティなど社内ITの質問に24時間対応",
    "mode": "chat",
    "icon": "🖥️",
    "icon_background": "#1C64F2"
  }'

# アプリのモデル設定を更新
# APP_ID は上で取得したIDを使用
APP_ID="yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"

curl -X POST "${DIFY_API_URL}/apps/${APP_ID}/model-config" \
  -H "Authorization: Bearer ${DIFY_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": {
      "provider": "ollama",
      "name": "qwen2.5:14b",
      "completion_params": {
        "temperature": 0.2,
        "max_tokens": 512
      }
    },
    "dataset_configs": {
      "retrieval_model": "multiple",
      "datasets": {
        "datasets": [{"dataset": {"enabled": true, "id": "'"${DATASET_ID}"'"}}]
      },
      "top_k": 5,
      "score_threshold_enabled": true,
      "score_threshold": 0.5
    }
  }'
```

---

## Phase 4: 初期テスト

### 4.1 基本動作確認チェックリスト

Difyプレビュー画面（またはAPIエンドポイント）で以下を実施する。

#### 正常系テスト（各カテゴリから1件）

| テスト質問 | 期待する回答のキーワード | 結果 |
|-----------|----------------------|------|
| VPNに接続できないのですが | GlobalProtect、vpn.example.co.jp | 未実施 |
| パスワードを忘れました | セルフサービスポータル、password.example.co.jp | 未実施 |
| ソフトウェアを申請したい | 社内ポータル、IT申請、上長承認 | 未実施 |
| PCがブルースクリーンになります | 代替機、内線9999 | 未実施 |
| プリンターが詰まった | 紙詰まり処理手順 | 未実施 |

#### 言い換えテスト

| テスト質問（言い換え） | 正解FAQ | 結果 |
|----------------------|--------|------|
| GPに繋がらない | FAQ-001（VPN接続） | 未実施 |
| パスワード忘れた | FAQ-008（パスワードリセット） | 未実施 |
| ロックされた | FAQ-009（アカウントロック） | 未実施 |

#### フォールバックテスト

| テスト質問 | 期待する動作 | 結果 |
|-----------|------------|------|
| 社長の給料はいくらですか | 「お答えできません」+ 人事部案内 | 未実施 |
| 明日の天気は？ | ヘルプデスクへの案内 | 未実施 |
| 私のパスワードを教えてください | 禁止事項として拒否 | 未実施 |

#### エスカレーションテスト（重要）

| テスト質問 | 期待する動作 | 結果 |
|-----------|------------|------|
| ランサムウェアに感染したかもしれません | LANケーブルを抜く + 内線9999に電話 | 未実施 |
| ファイルを誤送信してしまいました | 上長報告 + 情報セキュリティ部門連絡 | 未実施 |
| システムが全員ログインできません | 内線9999に至急電話 | 未実施 |

### 4.2 Retrieval Test（Dify管理画面で実施）

```
Dify管理画面 → ナレッジ → ITヘルプデスクFAQ → Retrieval Test

テスト質問を入力して、top_5の検索結果を確認する:
  1. 「VPN繋がらない」 → FAQ-001, FAQ-002 が上位に来るか
  2. 「パスワード忘れた」 → FAQ-008 が上位に来るか
  3. 「ランサムウェア」 → FAQ-障害インシデント系が上位に来るか

Score 0.5以上で正解FAQがhitしていることを確認する。
```

### 4.3 問題が発生した場合の調整

```
■ 正しいFAQが検索されない場合
  → Retrieval Test で何がhitしているか確認
  → 言い換えキーワードが不足 → FAQの言い換えキーワード列に追記
  → チャンクが大きすぎる → max_tokensを256から128に縮小
  → Score Thresholdを下げる（0.5 → 0.4）

■ ハルシネーションが発生する場合
  → システムプロンプトに「ナレッジベース外の情報は絶対に回答しない」を強調
  → Temperatureを下げる（0.2 → 0.1）
  → Top Kを減らす（5 → 3）で余計なコンテキストを排除

■ フォールバックが動かない場合
  → Score Thresholdを上げる（0.5 → 0.6）
  → システムプロンプトの「該当情報がない場合」の記述を強調

■ 回答が冗長な場合
  → システムプロンプトに「回答は3文以内で簡潔に」を追加
  → Max Tokensを下げる（512 → 256）
```

---

## Phase 5: 構築結果サマリー

### ナレッジベース

| KB名 | ドキュメント数 | Retrieval Mode | Top K | Score Threshold |
|------|-------------|---------------|-------|----------------|
| ITヘルプデスクFAQ | 70件 | Hybrid（ベクトル+全文） | 5 | 0.5 |

### アプリ設定

| 項目 | 設定値 |
|------|--------|
| 手段 | チャットボット |
| モデル | Ollama（qwen2.5:14b 推奨） |
| Temperature | 0.2 |
| Max Tokens | 512 |
| インデックス方式 | high_quality |

### プロンプト設計の要約

- 結論先出しを指示
- 条件・例外を必ず含める
- ナレッジベース外の情報を回答禁止
- 緊急インシデント（ランサムウェア・情報漏洩）は即座にエスカレーション
- 人事・給与・法的判断は回答禁止

### 初期テスト結果（計画）

| テスト種別 | 計画件数 | 目標合格率 |
|-----------|---------|----------|
| 正常系 | 5件 | 5/5 |
| 言い換え | 3件 | 3/3 |
| フォールバック | 3件 | 3/3 |
| エスカレーション | 3件 | 3/3（最重要） |
| 禁止事項 | 2件 | 2/2 |

### 調整した項目（想定）

- Ollamaモデル選定: qwen2.5:14b（日本語対応。8B以下では日本語品質が不安定になりやすいため14Bを推奨）
- チャンクサイズ: 256トークンに設定（Ollama向けに小さめ）
- Retrieval Mode: Hybrid（キーワード「GlobalProtect」「スプリットトンネル」等の技術用語でも検索できるよう全文検索を併用）
- Score Threshold: 0.5（初期値。テスト後に調整）

---

## 次のステップ

構築完了後、`/kb-eval` で以下の詳細評価を実施する。

1. テスト質問20件を用いたRetrieval評価（Hit Rate@3の計測）
2. Groundedness（根拠性）評価（ハルシネーション検出）
3. Completeness（網羅性）評価（条件・例外の抜け落ち確認）
4. ギャップ分析（問い合わせログにあってFAQにない質問の特定）
