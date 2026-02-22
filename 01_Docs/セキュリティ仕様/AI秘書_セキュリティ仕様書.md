# AI秘書システム セキュリティ仕様書

**最終更新日**: 2026/02/22
**バージョン**: 0.8.0 (Memory 2.0 P2 - RAG & Visualization)

本ドキュメントは、AI秘書プロジェクトにおけるセキュリティ方針、実装内容、およびデータプライバシーに関する仕様を統合したものです。
今後、セキュリティに関する変更が発生した場合は本ドキュメントを更新します。

---

## 1. 認証とID管理 (Authentication & Identity)

### 1.1 ユーザー認証
*   **認証基盤**: [Firebase Authentication](https://firebase.google.com/docs/auth) を利用。
*   **認証プロバイダ**:
    *   **Google**: メインのログイン手段（OAuth 2.0）。
    *   **Microsoft (Azure AD)**: Outlookカレンダー連携用（OAuth 2.0）。アクセストークンはローカルストレージに暗号化保存され、サーバー側には永続化されない。
*   **セッション管理**: Firebase SDKによるトークン管理。
    *   **Short-lived Token**: 1時間ごとにローテーションされるが、SDKがバックグラウンドで自動更新するため、ユーザーによる再ログイン操作は不要。
    *   **Secure Token Storage (v0.7.0)**:
        *   **Native App Flow**: システムブラウザを使用した「ループバックIPアドレス」によるOAuth 2.0認証フローを採用。
        *   **Encryption**: リフレッシュトークンは **Electron `safeStorage` (Windows DPAPI)** を用いて暗号化され、ローカルファイル (`secure_token.bin`) に保存される。
        *   **Persistence**: アプリ再起動時もトークンを復号してセッションを維持。Firebase SDKの制約（Rendererプロセスでの永続化不可）を回避し、完全なバックグラウンド運用を実現。

### 1.2 ID分離とリアルタイム同期
*   **User ID (UID)**: すべてのデータは Firebase Authentication が発行する `uid` に紐づけて管理する。
*   **データ分離**: Firestore等のデータベースにおいて、常に `/users/{uid}/` をルートパスとし、物理的・論理的に他ユーザーのデータと混在しない構造とする。
*   **探索ステータス同期 (v0.8.0)**: AIのバックエンド探索プロセス状況は `/users/{uid}/status/chat` に書き込まれ、フロントエンドへリアルタイム通知される。このデータは一時的な進捗メッセージのみを含み、個人を特定する情報は含まれない。

### 1.3 バックエンド実行権限 (Cloud Functions / Cloud Run)
*   **アプリケーション認証 (App-Level Auth)**:
    *   すべてのCloud Functions (`onCall`) 内で、必ず `request.auth` オブジェクトの存在を検証する。
    *   `request.auth` が未定義（未ログイン）の場合、即座に `HttpsError('unauthenticated')` をスローし、処理を実行しない。

---

## 2. データプライバシーと保存 (Data Privacy & Storage)

### 2.1 ログデータ (Activity Logs)
*   **収集内容**: ウィンドウタイトル、プロセス名、開始・終了時刻。
*   **プライバシー保護**:
    *   機密性の高いウィンドウタイトル（例: ブラウザのPrivateモード等）は、除外設定（Blacklist）により収集対象外とすることが可能。

### 2.2 センシングデータ (Tensai Sensing)
*   **仕様**: Pythonセンサー (`focus_sensor.py`) はキーの**打鍵数 (KPM)** とマウスの **移動距離** のみをカウントする。
*   **禁止事項**: どのキーが押されたか（入力内容）は**一切記録しない**。キーロガーとしての機能は持たない。
*   **処理場所**: 集中スコアの計算はすべて**ローカルデバイス内**で完結させる。
*   **コンテキスト制限**: 現在のAIモデルは「直近の活動ログ」のみを参照し、過去の全ログを検索する機能は持たない（プライバシーとパフォーマンスのバランスのため）。

### 2.3 データベースセキュリティ (Firestore)
*   **アクセス制御 (Security Rules)**:
    *   `request.auth.uid` がドキュメントのパスに含まれる `userId` と一致する場合のみ、読み書きを許可する (Row Level Security)。
    *   他ユーザーのデータへのアクセスはブロックされる。

### 2.4 データ保持期間 (Retention)
*   **短期記憶**: 直近の作業ログ等はコンテキスト生成のために一時保持される。
*   **日次処理**: クリーンアップ処理は現在手動だが、将来的には自動アーカイブ機能を実装予定 (Roadmap)。

---

## 3. 生成AIと機密情報 (Generative AI Security)

### 3.1 外部API利用方針
*   **APIプロバイダ**: Google Vertex AI (Gemini 2.0 Flash).
*   **認証方式**: **Application Default Credentials (ADC)**
    *   Cloud Functionsに紐付けられたサービスアカウントの権限を利用して認証する。APIキーの漏洩リスクを排除済み。
*   **Zero Data Retention**:
    *   Google Cloud のエンタープライズ契約に基づき、送信されたプロンプトやデータは、AIモデルの学習には**利用されない**。

---

## 4. アプリケーションセキュリティ (App Security)

### 4.1 配布とインストール (v0.6.3)
*   **インストーラー**: `electron-builder` (NSIS) によって生成された署名なしインストーラー (`.exe`) にて配布。
*   **SmartScreen**: デジタル署名が未実施のため、初回起動時に警告が表示される場合があるが、これは「信頼された発行元」リストにないためであり、マルウェアではない。
*   **インストール先**: ユーザー権限で書き込み可能な `%LOCALAPPDATA%` 配下にインストールされ、管理者権限を要求しない設計。

### 4.2 自動起動 (Auto-Start)
*   **レジストリ**: Windowsのスタートアップ登録 (`HKCU\Software\Microsoft\Windows\CurrentVersion\Run`) を使用し、ユーザーごとの設定として安全に管理される。

---

## 5. 変更履歴

*   **2026/02/22**: 第4版改訂 (v0.8.0 対応 - RAG最適化, 探索プロセス可視化)
*   **2026/02/19**: 第3版改訂 (v0.7.0 対応 - Native OAuth Flow, safeStorage)
*   **2026/02/15**: 第2版改訂 (v0.6.3 対応 - Gemini 2.0, Firestore Security, Auth Refresh)
*   **2026/01/29**: 初版作成 (Ver 0.3.0 リリース時点)
