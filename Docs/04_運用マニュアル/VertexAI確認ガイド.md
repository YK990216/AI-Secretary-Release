# Vertex AI 実態確認ガイド

**目的**: Vertex AIが実際にどのように動作し、ログを出力しているか、Google Cloud Console上で確認するためのガイドです。

## 1. 動作ログの確認 (Cloud Logging)
AIがどのようなプロンプトを受け取り、何を返したかという「思考の過程」を確認できます。

*   **ツール**: ログ エクスプローラ
*   **URL**: [https://console.cloud.google.com/logs?project=ai-secretary-e7f4e](https://console.cloud.google.com/logs?project=ai-secretary-e7f4e)
*   **確認方法**:
    1. クエリ欄に `resource.type="cloud_function"` と入力する。
    2. [クエリを実行] をクリックする。
    3. 時系列で `generateBriefing` や `generateProposal` のログが表示されます。

## 2. AI使用量・統計の確認 (Vertex AI)
APIの呼び出し回数や、エラー率などの統計情報を確認できます。

*   **ツール**: Vertex AI ダッシュボード
*   **URL**: [https://console.cloud.google.com/vertex-ai/generative/language/metrics?project=ai-secretary-e7f4e](https://console.cloud.google.com/vertex-ai/generative/language/metrics?project=ai-secretary-e7f4e)

## 3. プログラム本体の確認 (Cloud Functions)
AIを呼び出している「コード（実体）」の稼働状況を確認できます。

*   **ツール**: Cloud Functions 一覧
*   **URL**: [https://console.cloud.google.com/functions/list?project=ai-secretary-e7f4e](https://console.cloud.google.com/functions/list?project=ai-secretary-e7f4e)
    *   **generateBriefing**: エグゼクティブ・ブリーフィング生成用
    *   **generateProposal**: 次世代インタラクション（提案）用
    *   **generateDailySummary**: 記憶生成用（日次サマリー）
