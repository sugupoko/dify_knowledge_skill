# Dify Console API リファレンス — ナレッジベース・アプリ構築用

## 概要

Dify Console API を使って、ナレッジベースの作成・ドキュメントアップロード・アプリ設定をプログラムから行う。
手動（UIガイド）でも構築可能だが、大量のFAQを投入する場合はAPIが効率的。

**認証**: 全てのAPIリクエストに `Authorization: Bearer {api_key}` ヘッダーが必要。

**ベースURL**:
- Dify Cloud: `https://api.dify.ai/v1`
- Self-hosted: `http://{your-host}/v1`

---

## 1. 認証

### APIキーの取得

1. Dify管理画面にログイン
2. 右上のアカウント → 設定 → API Keys
3. 新しいAPIキーを作成

### リクエストヘッダー

```
Authorization: Bearer {your_api_key}
Content-Type: application/json
```

---

## 2. ナレッジベース（Dataset）API

### 2.1 ナレッジベースの作成

```
POST /datasets

Request Body:
{
  "name": "総務FAQ",
  "description": "総務部門向けのFAQナレッジベース",
  "indexing_technique": "high_quality",
  "permission": "only_me"
}

Response:
{
  "id": "dataset_id",
  "name": "総務FAQ",
  "description": "...",
  "created_at": 1712345678
}
```

| パラメータ | 型 | 必須 | 説明 |
|-----------|---|:---:|------|
| name | string | Yes | ナレッジベース名 |
| description | string | No | 説明 |
| indexing_technique | string | No | `high_quality` (推奨) or `economy` |
| permission | string | No | `only_me` or `all_team_members` |

### 2.2 ドキュメントのアップロード（テキスト）

```
POST /datasets/{dataset_id}/document/create-by-text

Request Body:
{
  "name": "FAQ_001_経費精算の提出期限",
  "text": "Q: 経費精算の提出期限はいつですか？\nA: 毎月末日が締め切りです。月末が休日の場合は前営業日になります。\n条件: 全社員共通\n例外: 海外出張の場合は帰国後10営業日以内",
  "indexing_technique": "high_quality",
  "process_rule": {
    "mode": "custom",
    "rules": {
      "pre_processing_rules": [
        {"id": "remove_extra_spaces", "enabled": true}
      ],
      "segmentation": {
        "separator": "###",
        "max_tokens": 500
      }
    }
  }
}

Response:
{
  "document": {
    "id": "document_id",
    "name": "FAQ_001_経費精算の提出期限",
    "indexing_status": "indexing"
  }
}
```

### 2.3 ドキュメントのアップロード（ファイル）

```
POST /datasets/{dataset_id}/document/create-by-file

Content-Type: multipart/form-data

Form Data:
  file: (ファイル)
  data: {
    "indexing_technique": "high_quality",
    "process_rule": {
      "mode": "automatic"
    }
  }
```

対応フォーマット: txt, markdown, pdf, html, xlsx, xls, docx, csv

### 2.4 ナレッジベースの一覧取得

```
GET /datasets?page=1&limit=20

Response:
{
  "data": [
    {
      "id": "dataset_id",
      "name": "総務FAQ",
      "document_count": 60,
      "word_count": 15000,
      "created_at": 1712345678
    }
  ],
  "total": 1,
  "page": 1,
  "limit": 20
}
```

### 2.5 ドキュメントの一覧取得

```
GET /datasets/{dataset_id}/documents?page=1&limit=20

Response:
{
  "data": [
    {
      "id": "document_id",
      "name": "FAQ_001",
      "indexing_status": "completed",
      "word_count": 150,
      "created_at": 1712345678
    }
  ]
}
```

### 2.6 ドキュメントの削除

```
DELETE /datasets/{dataset_id}/documents/{document_id}

Response:
{
  "result": "success"
}
```

---

## 3. Retrieval Test

Dify UIで利用可能。APIでの直接呼び出しは限定的だが、以下の方法でテスト可能:

### UIでのテスト手順
1. ナレッジベースを開く
2. 「Retrieval Test」タブを選択
3. テスト質問を入力
4. 検索結果（top_k）とスコアを確認
5. 正しいFAQがtop_kに含まれているか判定

### スクリプトでの自動テスト（アプリAPI経由）

```python
import requests

API_KEY = "your_api_key"
BASE_URL = "https://api.dify.ai/v1"

def test_retrieval(query, expected_keywords):
    """アプリAPI経由でretrievalをテスト"""
    response = requests.post(
        f"{BASE_URL}/chat-messages",
        headers={
            "Authorization": f"Bearer {API_KEY}",
            "Content-Type": "application/json"
        },
        json={
            "inputs": {},
            "query": query,
            "response_mode": "blocking",
            "user": "test_user"
        }
    )
    result = response.json()
    answer = result.get("answer", "")
    
    # 期待するキーワードが回答に含まれるか確認
    hit = all(kw in answer for kw in expected_keywords)
    return {
        "query": query,
        "answer": answer,
        "hit": hit,
        "expected": expected_keywords
    }

# テストケース
test_cases = [
    {"query": "経費精算の提出期限は？", "expected": ["毎月末日"]},
    {"query": "有休の申請方法は？", "expected": ["申請フォーム"]},
]

for tc in test_cases:
    result = test_retrieval(tc["query"], tc["expected"])
    status = "OK" if result["hit"] else "NG"
    print(f"[{status}] {result['query']}")
```

---

## 4. アプリ関連API

### 4.1 チャットメッセージの送信

```
POST /chat-messages

Request Body:
{
  "inputs": {},
  "query": "経費精算の提出期限を教えてください",
  "response_mode": "blocking",
  "conversation_id": "",
  "user": "user_001"
}

Response:
{
  "id": "message_id",
  "answer": "経費精算の提出期限は毎月末日です。...",
  "conversation_id": "conv_id",
  "created_at": 1712345678
}
```

| パラメータ | 型 | 必須 | 説明 |
|-----------|---|:---:|------|
| query | string | Yes | ユーザーの質問 |
| response_mode | string | Yes | `blocking` or `streaming` |
| conversation_id | string | No | 既存会話を継続する場合 |
| user | string | Yes | ユーザー識別子 |

### 4.2 会話履歴の取得

```
GET /messages?conversation_id={conv_id}&user=user_001&limit=20

Response:
{
  "data": [
    {
      "id": "message_id",
      "query": "経費精算の提出期限は？",
      "answer": "...",
      "created_at": 1712345678
    }
  ]
}
```

### 4.3 メッセージフィードバック

```
POST /messages/{message_id}/feedbacks

Request Body:
{
  "rating": "like",
  "user": "user_001"
}
```

| rating | 説明 |
|--------|------|
| `like` | 役に立った |
| `dislike` | 役に立たなかった |
| `null` | フィードバックを取り消し |

---

## 5. バッチ処理のパターン

### FAQ一括アップロード

```python
import csv
import requests
import time

API_KEY = "your_api_key"
BASE_URL = "https://api.dify.ai/v1"
DATASET_ID = "your_dataset_id"

def upload_faq(name, text):
    """1件のFAQをドキュメントとしてアップロード"""
    response = requests.post(
        f"{BASE_URL}/datasets/{DATASET_ID}/document/create-by-text",
        headers={
            "Authorization": f"Bearer {API_KEY}",
            "Content-Type": "application/json"
        },
        json={
            "name": name,
            "text": text,
            "indexing_technique": "high_quality",
            "process_rule": {"mode": "automatic"}
        }
    )
    return response.json()

# CSVからFAQを読み込んでアップロード
with open("knowledge/faq_draft.csv", "r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for i, row in enumerate(reader):
        name = f"FAQ_{i+1:03d}_{row['category']}_{row['question'][:20]}"
        text = f"Q: {row['question']}\nA: {row['answer']}"
        if row.get('conditions'):
            text += f"\n条件: {row['conditions']}"
        if row.get('exceptions'):
            text += f"\n例外: {row['exceptions']}"
        if row.get('next_action'):
            text += f"\n次の行動: {row['next_action']}"
        if row.get('reference'):
            text += f"\n参照元: {row['reference']}"
        
        result = upload_faq(name, text)
        print(f"Uploaded: {name} -> {result.get('document', {}).get('id', 'error')}")
        time.sleep(0.5)  # レートリミット対策
```

---

## 6. 注意事項

### レートリミット
- Dify Cloud: 通常100リクエスト/分程度（プランによる）
- バッチ処理時は `time.sleep(0.5)` 程度の間隔を空ける

### ドキュメントのインデクシング
- アップロード後、インデクシングが完了するまで検索に反映されない
- `indexing_status` が `completed` になるまで待つ
- 大量のドキュメントをアップロードした場合、数分かかることがある

### Embeddingモデル
- `high_quality` モード: 外部のEmbeddingモデル（text-embedding-3-small等）を使用
- `economy` モード: 簡易的なインデクシング（精度は低い）
- 本番環境では `high_quality` を推奨

### Self-hosted環境
- APIのベースURLが異なる（`http://{your-host}/v1`）
- APIキーの取得方法は同じ
- Embeddingモデルの設定が別途必要
