# AI Secretary (AI秘書)

**「文脈」と「感覚」を持つ、エグゼクティブのための自律型AIアシスタント。**

AI Secretary は、単なるチャットボットではありません。
あなたのPC上の活動履歴（ログ）と、未来の予定（カレンダー）、そしてリアルタイムの集中状態（センサー）を統合し、「今、あなたが何をすべきか」を先回りしてサポートする Windows アプリケーションです。

![Version](https://img.shields.io/badge/version-0.7.1-blue.svg)
![Platform](https://img.shields.io/badge/platform-Windows-0078D6.svg)
![Stack](https://img.shields.io/badge/stack-Electron%20%7C%20React%20%7C%20Python%20%7C%20Firebase%20%7C%20Gemini-4285F4.svg)

---

## 🚀 Key Features (v0.7.1)

### 1. 🧠 Deep Memory Briefing (文脈統合ブリーフィング)
*   **Deep Memory**: 過去数日間の作業ログを記憶し、現在のコンテキストを理解。
*   **Calendar Awareness**: Google/Outlookカレンダーから直近の予定を取得。
*   **Impact**: これらを統合し、「A社との定例会議です。前回の議事録（ログ）によると、〇〇の件を確認する必要があります」といった高度なアドバイスを自動生成します。
*   **Auto-Generation (Hybrid Deterministic)**: 毎晩2:00に「不活動時間」も含めた24時間1分の隙間もない記憶を自動生成。通信エラーや停止期間があっても、稼働再開時にポインタから遡って自動補完します (v0.7.1+)。

### 2. ⚡ Smart Tensai Sensing (集中センシング)
*   **Multi-Modal Sensor**: キーボード入力密度(KPM)とマウス移動距離をリアルタイム監視（Python Sidecar）。
*   **Privacy-First**: 入力内容（キーロガー）は**一切保存せず**、統計値のみを用いて集中スコア(0-100)を算出します。

### 3. 🛠️ Robust System Integration (システム連携の強化)
*   **Auto-Start**: Windows起動時に自動的に秘書が立ち上がり、常に見守ります。
*   **Native Authentication**: システムブラウザを使用した堅牢なOAuth認証フローを採用。従来の「1時間ごとの切断」問題を解消し、長時間の安定稼働を実現しました (v0.7.0+)。
*   **Persistent Settings**: 画面分割位置やショートカット設定は、アプリ終了後も確実に保存されます。

---

## 📚 Documentation
本リポジトリには、開発者向けの技術仕様が含まれています。

*   **[リリースノート (v0.7.1)](release/RELEASE_NOTES_v0.7.1.md)**: OS通知および記憶生成の堅牢化
*   **[製品仕様書 (v0.7.1)](docs/01_製品仕様/AI秘書_製品仕様書_v0.7.1.md)**: ハイブリッド確定型ロジック
*   [セキュリティ仕様書](AI秘書_セキュリティ仕様書.md): データ保護とプライバシーポリシー

---

## 📦 Installation
1. リリースページから `AI-Secretary-Setup-0.7.1.exe` をダウンロードしてください。
2. インストーラーを実行すると、自動的にセットアップされます。
3. 詳細は [インストールマニュアル](manual/installation_manual.md) を参照してください。

---

## 🛠️ Technology Stack

*   **Frontend**: Electron, React, TypeScript, Vite
*   **Sensing**: Python 3.12 (pynput), PyInstaller (Standalone EXE)
*   **AI**: Google Gemini 2.0 Flash (via Vertex AI / Studio)
*   **Cloud**: Firebase Authentication, Firestore (Realtime Sync)

---

© 2026 AI Secretary Project
