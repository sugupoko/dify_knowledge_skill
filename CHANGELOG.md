# CHANGELOG

## [Unreleased] — 2026-04-06

### 追加

- **reference/spec_template.md**: ナレッジ設計仕様書テンプレートを新規追加
  - 情報源テーブル（R1=データ、R2=ヒアリング、R3=文書、R4=仮定）
  - ナレッジ設計テーブル（種類・設定値・根拠・Ref）
  - プロンプト設計テーブル（セクション・内容・根拠・Ref）
  - 仮定テーブル（仮定・値・根拠・Ref・検証状況）
  - 受け入れ要件（利用者の利用方法・表示形式・FAQ更新方法）
  - QAチェックリスト（設計↔実装の突合）

- **reference/hearing_sheet_chatflow.md**: チャットフロー向けヒアリングシートを新規追加
  - 分類カテゴリの定義
  - 条件分岐のルール
  - ルーティング先の一覧
  - 暗黙のNG回答パターン（最重要）セクション

- **reference/hearing_sheet_agent.md**: エージェント向けヒアリングシートを新規追加
  - 使用するツール/APIの定義
  - ポリシー・制約条件
  - 暗黙のNG回答パターン（最重要）セクション
  - エスカレーション先の一覧

- **.claude/commands/kb-request.md**: 確認依頼書生成スキルを新規追加
  - 4パターンの質問形式（FAQ確認・エスカレーション確認・仮定確認・運用確認）
  - 「こう理解していますが合っていますか？」形式の依頼書テンプレート
  - 出力: `reports/request.md`

- **.claude/commands/kb-operate.md**: 運用設計スキルを新規追加
  - 3層の監視設計（ナレッジベース健全性・アプリ健全性・手段の妥当性）
  - FAQ更新サイクルの設計
  - フォールバック設計（Dify/LLM停止時の対応）
  - 手段変更の判断トリガー
  - 出力: `reports/operate_design.md`

### 変更

- **CLAUDE.md**:
  - ディレクトリ構成をバージョンフォルダ（v1/, v2/）形式に更新
  - 出力先を `workspace/<project>/vN/` 配下に変更
  - スキル一覧に kb-request, kb-operate を追加
  - ワークフロー図を新スキルに合わせて更新
  - バージョンフォルダ管理セクションを追加
  - Git ブランチ命名規則（`kb/<project>/<version>-<description>`）を追加

- **reference/state_schema.md**:
  - `kb_version` フィールドを追加（v1, v2, ...）
  - `.kb_state.yaml` の配置場所をバージョンフォルダ内（`v1/.kb_state.yaml`）に変更
  - `eval` セクションに `impossibility_findings` フィールドを追加
  - バージョン管理との連携ガイドを更新
  - `/kb-operate` スキルの参照を追加

- **reference/evaluation_guide.md**:
  - 「不可能性の判断基準」セクションを追加（手段変更トリガーの判断基準表）
  - 「手段バリエーション比較テーブル」セクションを追加（チャットボット/チャットフロー/エージェントの比較）

- **.claude/commands/kb-design.md**:
  - Phase 7 に `spec.md` の生成を追加（`reference/spec_template.md` を参照）

- **.claude/commands/kb-build.md**:
  - Phase 4.5 として QA チェックセクションを追加（設計↔実装の突合5項目）
  - 出力レポートに QA チェック結果テーブルを追加

- **.claude/commands/kb-eval.md**:
  - Phase 6.5 として「不可能性の判断」セクションを追加
  - 評価レポートの出力に「不可能性の判断」テーブルを追加
  - 次のステップに「手段の見直し」の案内を追加

- **README.md**:
  - スキル一覧を7スキルに更新
  - ディレクトリ構成を新ファイルに合わせて更新
