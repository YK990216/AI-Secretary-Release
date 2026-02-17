# AI秘書アプリケーション仕様書 (v0.2.0)

## 1. 概要
本ドキュメントは、**AI秘書アプリケーション Windows MVP (v0.2.0)** の実装仕様を定義するものである。
v0.2.0では、Windows PC上の作業ログ収集、クラウド同期、および Office 365 / Google Calendar との連携機能を実装し、スタンドアローンアプリ（EXE）として動作することをゴールとしている。

## 2. システム構成
### 2.1. アプリケーションアーキテクチャ
*   **Platform**: Windows Desktop Application (Electron + React + TypeScript)
*   **Build System**: Vite (Frontend), Electron-Builder (Packaging)
*   **Authentication**:
    *   **Google**: Firebase Authentication (Google Sign-In)
    *   **Microsoft**: MSAL React (`@azure/msal-react`) for Graph API
*   **Database**: Firebase Firestore
*   **AI Engine**: Google Gemini API (2.0 Flash / Latest)

### 2.2. 動作環境
*   **OS**: Windows 10/11 (PowerShell使用)
*   **Network**: インターネット接続必須（Firebase, Graph API, Gemini API利用のため）
*   **Dependency**: アプリ単独で動作（Unpacked EXE + DLLs）

## 3. 機能仕様

### 3.1. ユーザー認証 (Authentication)
*   **必須ログイン**: アプリ利用開始時に Google アカウントによるログインを強制する。
*   **データ隔離**: 認証されたユーザーID (UID) に基づき、Firestore上のデータアクセス権を制御する。
*   **Office 365連携**:
    *   任意で Microsoft アカウントを連携可能。
    *   連携時、Outlook カレンダーの読み取り権限 (`Calendars.Read`) を取得する。
    *   アクセストークンは `localStorage` に保存され、有効期限切れ時は再ログインを促す。

### 3.2. アクティビティログ収集 (Log Monitor)
*   **監視対象**: アクティブなウィンドウの「プロセス名」と「ウィンドウタイトル」。
*   **収集ロジック**:
    *   PowerShellスクリプト (`electron/logMonitor.ts`) がバックグラウンドで監視。
    *   ウィンドウが切り替わったタイミングで、そこまでの「滞在時間 (Duration)」を計算して送信。
    *   アプリ側 (`App.tsx`) でこれを受信し、画面表示およびクラウドへ送信する。
*   **プライバシー**: ウィンドウタイトルが空の場合は除外。

### 3.3. クラウド同期 (Cloud Sync)
*   **送信トリガー**: ログ収集（ウィンドウ切り替え）と同時に Firestore へ送信。
*   **データ構造**:
    ```json
    users/{uid}/activity_logs/{docId}
    {
      "ProcessName": "chrome",
      "Title": "GitHub - AI-Secretary ...",
      "Timestamp": "2026-01-29T10:00:00...",
      "duration": 120, // 秒
      "createdAt": ServerTimestamp
    }
    ```

### 3.4. ダッシュボード機能
*   **ログ一覧表示**: 直近の作業ログをリアルタイムでリスト表示。
*   **Focus Widget**:
    *   直近20件のログ内容（アプリ名、タイトル）から、「集中（Productive）」「散漫（Distracted）」を判定。
    *   スコア (0-100) を算出し、状態（Flow / Neutral / Distracted）を表示。
*   **Schedule Widget**:
    *   **Google Calendar**: 直近の予定を取得して表示。
    *   **Outlook Calendar**: 連携済みの場合、Microsoft Graph API 経由で予定を取得し、Googleの予定とマージして時系列順に表示。
*   **Briefing Section**:
    *   Gemini API を使用し、蓄積されたログから「現在の作業状況」と「次のアクション」をAIが生成して提示。

## 4. 配布・デプロイ
### 4.1. ビルド形式
*   **形式**: Unpacked Executable (フォルダ形式)
*   **構成**: `win-unpacked` フォルダ内に `AI Secretary.exe` と依存DLLが含まれる。
*   **ローカルサーバー**: 
    *   `file://` プロトコルによる認証エラー (Firebase Auth) を回避するため、アプリ内部で `http://localhost:5173` サーバーを起動し、自身をホスティングする。

### 4.2. 配布方法
*   GitHub Releases を使用。
*   `win-unpacked` フォルダ全体をZip圧縮して配布する。

## 5. 既知の制限事項 (Limitations)
*   **オフライン動作**: 不可（認証およびログ送信に失敗する）。
*   **バックグラウンド動作**: アプリを完全に終了（×ボタンで閉じる）するとログ収集も停止する。最小化状態で維持する必要がある。
*   **インストーラー**: v0.2.0 時点では未提供（Unpacked形式のみ）。
*   **デバイス区別**: 複数PCで同一アカウントを使用した場合、ログはサーバー上で混在する（区別フラグ未実装）。

## 6. 今後のロードマップ (v0.3.0+)
*   **インストーラー対応**: 署名問題の解決と各種インストーラー作成。
*   **デバイス識別**: ログに `Hostname` または `DeviceId` を付与し、デバイスごとのフィルタリングを可能にする。
*   **セキュリティ強化**: ビルド時のAPIキー隠蔽（環境変数の動的注入など）。
*   **自動更新**: Electron-Updater によるアプリ自動更新機能。
