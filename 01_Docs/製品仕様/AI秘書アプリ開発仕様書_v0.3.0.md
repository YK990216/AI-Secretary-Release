# AI秘書アプリケーション仕様書 (v0.3.0)

## 1. 概要
本ドキュメントは、**AI秘書アプリケーション Windows MVP (v0.3.0)** の実装仕様を定義するものである。
v0.3.0では、v0.2.0の機能に加え、**「五感（Sensing）」**と**「行動介入（Gatekeeper）」**、および**「文脈統合（Briefing integration）」**を実装し、ユーザーの集中を守る機能を強化した。

## 2. システム構成
### 2.1. アプリケーションアーキテクチャ
*   **Platform**: Windows Desktop Application (Electron + React + TypeScript)
*   **Build System**: Vite (Frontend), Electron-Builder (Packaging)
*   **Sensing Engine**: Python Sidecar (`focus_sensor.exe`) - **New**
*   **Authentication**: Firebase Authentication (Google) / MSAL (Microsoft)
*   **Database**: Firebase Firestore
*   **AI Engine**: Google Gemini API (2.0 Flash)

### 2.2. 動作環境
*   **OS**: Windows 10/11
*   **Dependency**:
    *   **Python不要**: センサー用Pythonランタイムはアプリに内蔵（PyInstallerによりEXE化）。
    *   **管理者権限不要**: ユーザープロファイル環境(`AppData`)で動作。

## 3. 機能仕様 (v0.3.0 New Features)

### 3.1. 集中センシング (Smart Tensai Sensing)
ユーザーの「入力活動量」をリアルタイムで監視し、ゾーン状態を判定する。
*   **センサー**: `pynput` を使用した Python バックグラウンドプロセス。
*   **取得メトリクス**:
    *   **KPM (Keystrokes Per Minute)**: 直近1分間のキータイプ数。
    *   **Mouse Distance**: 直近1分間のマウス移動距離(px)。
    *   **Privacy**: キーの内容（何の文字を打ったか）は**一切取得しない**。
*   **集中スコア計算**:
    *   `Focus Score = (App Context Score * 0.4) + (Input Activity Score * 0.6)`
    *   アプリが「生産的（Productive）」カテゴリであり、かつ「入力が活発」な場合に最高スコアとなる。

### 3.2. 通知ゲートキーパー (Notification Gatekeeper)
集中状態に応じて、AI秘書からの通知（トースト）を制御する。
*   **Screening (取り次ぎ機能)**:
    *   **Flow状態 (Score > 80)**: 通知を即時表示せず、**キュー（保留箱）**に格納する。
    *   **通常状態**: キューに溜まった通知をまとめて放出（Digest）するか、即時表示する。
*   **UI表示**:
    *   ヘッダー部に「取り次ぎ可能」「取り次ぎ停止中（集中モード）」のステータスを表示。

### 3.3. コンテキスト統合ブリーフィング (Integrated Briefing)
過去のログと未来の予定を統合し、より高度なアドバイスを生成する。
*   **Deep Memory**: 過去3日〜7日分の作業ログを Firestore から取得。
*   **Future Context**: Google Calendar / Outlook から直近24時間の予定を取得。
*   **Prompt Engineering**: 「あなたは優秀なエグゼクティブ・セクレタリーである」という人格定義のもと、上記2つの情報を結合して「次の会議の準備」や「時間の使い方」をアドバイスする。

### 3.4. 既存機能 (v0.2.0より継承)
*   **Log Monitor**: ウィンドウタイトルとプロセス名の収集。
*   **Cloud Sync**: Firestoreへのリアルタイムログ送信。
*   **User Auth**: Googleログインによる認証とデータ分離。

## 4. 配布・デプロイ

### 4.1. パッケージング
*   **Main App**: Electron Builder による `win-unpacked` 形式。
*   **Sidecar**: `focus_sensor.exe` を `resources/` フォルダに同梱。
*   **Execution**: アプリ起動時にElectronのMain Processが自動的にセンサーEXEを `spawn` し、パイプ通信を行う。

### 4.2. インストール方法
ネットワーク制限環境に対応するため、カスタムスクリプト方式を採用。
*   **Installer**: `scripts/install.ps1` (PowerShell)
*   **動作**:
    1.  `dist/win-unpacked` を `%LOCALAPPDATA%/AISecretary/` にコピー。
    2.  デスクトップとスタートメニューにショートカットを作成。
    3.  旧バージョンのクリーンアップを自動実行。

## 5. ロードマップ (Ver 3.x)
*   **L0 First**: ログ保存のクラウドネイティブ化（ローカルディスク依存の排除）。
*   **Mobile App**: 外出先からのログ閲覧と簡易指示。
*   **Voice Interface**: 音声によるブリーフィング読み上げ。
