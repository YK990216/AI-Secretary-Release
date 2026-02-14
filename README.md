# AI Secretary (AI秘書)

**「文脈」と「感覚」を持つ、エグゼクティブのための自律型AIアシスタント。**

AI Secretary は、単なるチャットボットではありません。
あなたのPC上の活動履歴（ログ）と、未来の予定（カレンダー）、そしてリアルタイムの集中状態（センサー）を統合し、「今、あなたが何をすべきか」を先回りしてサポートする Windows アプリケーションです。

![Version](https://img.shields.io/badge/version-0.6.3-blue.svg)
![Platform](https://img.shields.io/badge/platform-Windows-0078D6.svg)
![Stack](https://img.shields.io/badge/stack-Electron%20%7C%20React%20%7C%20Python%20%7C%20Firebase%20%7C%20Gemini-4285F4.svg)

---

## 🚀 Key Features (v0.6.3)

### 1. 🧠 Deep Memory Briefing (文脈統合ブリーフィング)
*   **Deep Memory**: 過去数日間の作業ログを記憶し、現在のコンテキストを理解。
*   **Calendar Awareness**: Google/Outlookカレンダーから直近の予定を取得。
*   **Impact**: これらを統合し、「A社との定例会議です。前回の議事録（ログ）によると、〇〇の件を確認する必要があります」といった高度なアドバイスを自動生成します。
*   **Auto-Generation**: 毎晩22:00にその日の記憶を自動生成し、翌日のブリーフィングに備えます (v0.6.3+)。

### 2. ⚡ Smart Tensai Sensing (集中センシング)
PCがあなたの「集中ゾーン」を理解します。
*   **Multi-Modal Sensor**: キーボード入力密度(KPM)とマウス移動距離をリアルタイム監視（Python Sidecar）。
*   **Privacy-First**: 入力内容（キーロガー）は**一切保存せず**、統計値のみを用いて集中スコア(0-100)を算出します。

### 3. 🛠️ Robust System Integration (システム連携の強化)
*   **Auto-Start**: Windows起動時に自動的に秘書が立ち上がり、常に見守ります (v0.6.3+)。
*   **Resilient Connectivity**: カレンダーAPI等の認証切れを自動検知し、バックグラウンドで再接続します (v0.6.3+)。
*   **Persistent Settings**: 画面分割位置やショートカット設定は、アプリ終了後も確実に保存されます。

---

## 📦 Installation

1. リリースページから `AI-Secretary-Setup-0.6.3.exe` をダウンロードしてください。
2. インストーラーを実行すると、自動的にセットアップされます。
3. 詳細は [インストールマニュアル](manual/installation_manual.md) を参照してください。

---

## 🛠️ Technology Stack

*   **Frontend**: Electron, React, TypeScript, Vite
*   **Sensing**: Python 3.12 (pynput), PyInstaller (Standalone EXE)
*   **AI**: Google Gemini 2.0 Flash (via Vertex AI / Studio)
*   **Cloud**: Firebase Authentication, Firestore (Realtime Sync)
*   **Calendar**: Google Calendar API, Microsoft Graph API

---

## 🔒 Security & Privacy

*   **Zero Data Retention**: AIモデルへのデータ送信時、学習利用を禁止する設定で運用しています。
*   **Local Processing**: センサーデータの解析はすべてローカルPC上で行われます。
*   **Secure Storage**: ログデータはユーザーIDごとに分離されたクラウドデータベース(Firestore)に暗号化して保存されます。

詳細は [セキュリティ仕様書](AI秘書_セキュリティ仕様書.md) を参照してください。

---

## 📂 Project Structure

*   `windows-mvp/`: アプリケーション本体のソースコード
    *   `src/`: React フロントエンド
    *   `electron/`: Electron メインプロセス & ログ収集(PowerShell)
    *   `sensing/`: Python センサーモジュール
    *   `scripts/`: インストール用スクリプト
*   `manual/`: 各種マニュアル・ドキュメント

---

© 2026 AI Secretary Project
