# AI Secretary (AI秘書)

**「文脈」と「感覚」を持つ、エグゼクティブのための自律型AIアシスタント。**

AI Secretary は、単なるチャットボットではありません。
あなたのPC上の活動履歴（ログ）と、未来の予定（カレンダー）、そしてリアルタイムの集中状態（センサー）を統合し、「今、あなたが何をすべきか」を先回りしてサポートする Windows アプリケーションです。

![Version](https://img.shields.io/badge/version-0.8.1-blue.svg)
![Platform](https://img.shields.io/badge/platform-Windows-0078D6.svg)
![Stack](https://img.shields.io/badge/stack-Electron%20%7C%20React%20%7C%20Python%20%7C%20Firebase%20%7C%20Gemini-4285F4.svg)

---

## 🚀 Key Features (v0.8.1)

### 1. 🔍 VectorRAG (Hybrid Search) - **NEW**
*   **Knowledge Ingestion**: 「記憶して」と指示するだけで、AIが重要な情報を抽出・構造化して知識ベースへ保存。
*   **KnowledgeCard**: 保存された知識を視覚的なカードで表示。編集や有効期限管理も直感的に行えます。
*   **Source Citation**: 回答の根拠となった知識を **(出典: [知識: ...])** として明示。AIのハルシネーション（嘘）を抑制し、信頼性を高めます。

### 2. 🧠 Deep Memory Briefing (文脈統合ブリーフィング)
*   **Deep Memory**: 過去数日間の作業ログを記憶し、現在のコンテキストを理解。
*   **Calendar Awareness**: Google/Outlookカレンダーから直近の予定を取得。
*   **Auto-Generation (Hybrid Deterministic)**: 毎晩2:00に「不活動時間」も含めた24時間1分の隙間もない記憶を自動生成。通信エラーや停止期間があっても、稼働再開時にポインタから遡って自動補完します。

### 3. ⚡ Smart Tensai Sensing (集中センシング)
*   **Multi-Modal Sensor**: キーボード入力密度(KPM)とマウス移動距離をリアルタイム監視（Python Sidecar）。
*   **Privacy-First**: 入力内容（キーロガー）は**一切保存せず**、統計値のみを用いて集中スコア(0-100)を算出します。

### 4. 🛠️ Robust System Integration (システム連携)
*   **Auto-Start**: Windows起動時に自動的に秘書が立ち上がり、常に見守ります。
*   **Native Authentication**: システムブラウザを使用した堅牢なOAuth認証フローを採用。従来の「1時間ごとの切断」問題を解消し、長時間の安定稼働を実現しました。

---

## 📚 Documentation
本リポジトリには、開発者向けの技術仕様が含まれています。

*   **[リリースノート (v0.8.1)](release/RELEASE_NOTES_v0.8.1.md)**: VectorRAG Phase 2 実装
*   **[製品仕様書 (v0.8.1)](docs/01_製品仕様/AI秘書_製品仕様書_v0.8.1.md)**: ハイブリッド確定型ロジック & VectorRAG
*   [セキュリティ仕様書](AI秘書_セキュリティ仕様書.md): データ保護とプライバシーポリシー

---

## 📦 Installation
1. [リリースページ](https://github.com/YK990216/AI-Secretary-Release/tree/main/release) から `AI-Secretary-Setup-0.8.1.exe` をダウンロードしてください。
2. インストーラーを実行すると、自動的にセットアップされます。
3. 詳細は [インストールマニュアル](manual/installation_manual.md) を参照してください。

---

## 🛠️ Technology Stack

*   **Frontend**: Electron, React, TypeScript, Vite
*   **Sensing**: Python 3.12 (pynput), PyInstaller (Standalone EXE)
*   **AI**: Google Gemini 2.0 Flash (via Vertex AI / Studio)
*   **Cloud**: Firebase Authentication, Firestore (Realtime Sync & Vector Search)

---

© 2026 AI Secretary Project
