# AI秘書 製品仕様書 (v0.8.0)

**最終更新日**: 2026-02-22
**バージョン**: 0.8.0 (Memory 2.0 Phase 2 - RAG Optimization)

## 1. 概要
AI秘書 ("AI Secretary") は、エグゼクティブおよびプロフェッショナル向けの次世代型業務支援アシスタントです。
PC上の操作ログをローカルで解析し、ユーザーの「集中力」と「スケジュール」をリアルタイムに防御します。

v0.8.0では、**Memory 2.0 (Phase 2)** として RAG (Retrieval-Augmented Generation) システムの最適化を行いました。デフォルトのメモリ注入範囲を7日間に短縮してコストを大幅に削減しつつ、AIが自律的に過去を遡る検索機能と、その探索状況をユーザーにリアルタイムで伝える可視化機能を実装しました。

---

## 2. 機能一覧

### 2.1. ダッシュボード (メイン画面)
*   **集中力モニタリング (Focus Widget)**
    *   キーボード操作数 (KPM) とマウス操作距離から、現在の「集中スコア」をリアルタイム算出。
    *   3段階の状態判定:
        *   🌊 **FLOW**: 高い集中状態。通知が抑制されます。
        *   😐 **NEUTRAL**: 通常状態。
        *   ⚠️ **DISTRACTED**: 散漫状態（一定時間操作がない、または乱雑な操作）。
*   **スケジュール表示 (Schedule Widget)**
    *   Googleカレンダーと連携し、直近の予定を表示。
    *   **複数カレンダー対応 (v0.6.5 New)**: ユーザーが選択した複数のカレンダー（例: 「日本の祝日」「チームカレンダー」）を統合して表示。
    *   会議開始10分前に通知を送付。
    *   **安定した自動更新 (v0.7.0 New)**: ネイティブOAuthフローの実装により、長時間の稼働でも更新が停止しません。
*   **アクティビティログ (Activity Log)**
    *   現在使用中のアプリ名とウィンドウタイトルを記録・表示。
    *   記録はローカルおよびクラウド（Firestore）に同期。

### 2.2. AIチャットアシスタント (Chat Widget)
*   **統合チャットパネル (Integrated Chat Panel)**
    *   画面中央に常時表示されるメインインターフェース。
    *   **Gemini 2.5 Flash (Vertex AI)** モデルを使用。
*   **階層型RAG (v0.8.0 Enhanced)**
    *   **コスト最適化**: 初期注入メモリを直近7日間に制限。
    *   **自律的遡及検索 (Recursive Search)**: 7日間で不足がある場合、AIが自律的に過去（最大数十日分）を探索。
    *   **ハイブリッド・コンテキスト (Hybrid Strategy)**: 直近の回答において L0 (生ログ) と L1 (1時間サマリ) を併用し、精度の抜け漏れを防止。
    *   **探索状況の可視化 (Real-time Status)**: AIが検索中のプロセス（例：「15日前を精査中...」）をUI上にリアルタイム表示。
    *   **引用元表示 (Source Badges)**: 回答の根拠となった日付をメッセージ下部にバッジ表示。
*   **コンテキスト認識 (Context Awareness)**
    *   ユーザーの「直近の操作ログ」と「本日のスケジュール」をAIが把握した状態で回答。
    *   **タイムゾーン認識強化 (Global Ready)**: 内部的にUTC/JST変換ロジックを確立。将来的にユーザーごとのタイムゾーン設定に対応可能な設計に移行。
*   **ルール学習 (Rule Learning)**
    *   ユーザーの発言からルールや好みを学習し、`SECRETARY.md` (Personal Policy) に自動保存。

### 2.3. 記憶システム (Memory System)
*   **L0 短期記憶 (Short-Term)**
    *   直近の操作ログ・チャット履歴を保持。
*   **L1 中期記憶 (Daily Summary / Memory 2.0)**
    *   **完全自動生成**: 毎日深夜 02:00 (JST) に Cloud Functions が自律的にログを解析し、前日の要約を生成・保存します。
*   **L2 長期記憶・アーカイブ (Retrieval)**
    *   **オンデマンド・バッチフェッチ (v0.8.0)**: 過去のサマリを必要に応じてAIが動的に取得する仕組み（Function Calling）を構築。

### 2.4. システム機能
*   **システムトレイ常駐**
    *   ウィンドウを閉じてもバックグラウンドで動作を継続。
*   **自動アップデート**
    *   起動時に最新バージョンを確認し、自動的にダウンロード・適用。
*   **セキュアな自動ログイン (Enhanced v0.7.0)**
    *   **Native App Flow**: システムブラウザを使用した堅牢なOAuth 2.0フロー。
    *   **Refresh Token Encryption**: `safeStorage` (DPAPI) を用いてリフレッシュトークンを暗号化保存し、アプリ再起動後もセッションを持続。

---

## 3. システム要件
*   **OS**: Windows 10 / 11 (64bit)
*   **ネットワーク**: インターネット接続必須 (Google API, Firebase連携のため)
*   **アカウント**: Googleアカウント (カレンダー連携のため)

---

## 4. アーキテクチャ概要
### クライアント (Windows)
*   **Framework**: Electron + React + TypeScript
*   **Auth**: Google OAuth 2.0 (Loopback IP Flow) via `google-auth-library`
*   **Sensing**: Pythonサイドカー (`sensing/focus_sensor.py`) による入力監視。
*   **Updates**: electron-updater (GitHub Releases)

### クラウド (Backend)
*   **Platform**: Google Cloud Platform / Firebase
*   **Database**: Cloud Firestore (ログ、記憶、ユーザー設定)
*   **AI Model**: **Google Gemini 2.5 Flash** (via Vertex AI SDK)

---

## 5. 既知の制限事項 (v0.7.0)
*   **L1記憶の生成タイミング**: 毎日深夜 02:00 (JST) に自動実行されます。手動での再生成も可能です。
*   **オフライン動作**: ログ取得は継続されるが、AIチャットやカレンダー同期は不可。
