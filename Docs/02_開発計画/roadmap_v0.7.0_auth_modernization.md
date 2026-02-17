# v0.7.x RoadMap: 認証アーキテクチャの刷新とカレンダー機能の完全化

## 1. 現状の課題と背景 (v0.6.5時点)

### 課題: Googleカレンダー連携の不安定さ
v0.6.5では、Firebase JS SDK (`signInWithPopup`) を使用していますが、Electron (SPA構成) が「公開クライアント」として扱われるため、Googleのセキュリティポリシーにより `oauthRefreshToken`（長期的なアクセス権）が取得できない問題が発生しています。

### 暫定対応 (v0.6.5)
*   **仕様**: アクセストークン（有効期限1時間）のみで動作するように修正。
*   **影響**: 1時間ごとに再ログインが必要、またはバックグラウンド更新が停止する可能性がある。

## 2. v0.7.0 の目的
**「ユーザーが意識することなく、半永続的にカレンダー連携が維持される」** 状態を実現するため、認証アーキテクチャを根本から刷新します。

## 3. 解決策: ネイティブアプリ・フロー (Native App Flow)

### アーキテクチャの変更
Webアプリ（SPA）としての認証依存を脱却し、Electronのメインプロセス（Node.js）主体で認証を行います。

| 機能 | v0.6.5 (Current) | v0.7.0 (Target) |
| :--- | :--- | :--- |
| **主体** | Renderer Process (React/SPA) | **Main Process (Node.js)** |
| **SDK** | Firebase JS SDK (`firebase/auth`) | **google-auth-library** (Node.js) |
| **認証方式** | Browser Popup (`signInWithPopup`) | **Loopback IP Flow** (RFC 8252) |
| **トークン管理** | LocalStorage / IndexedDB (Unsecure) | **Electron safeStorage** (Operating System encryption) |
| **更新処理** | JS SDK任せ (不安定) | **Main Process Cron** (確実) |

### 技術選定の理由
1.  **RFC 8252準拠**: デスクトップアプリとして標準的かつ推奨されるセキュアな方式（AWS CLIやVSCodeと同様）。
2.  **Firebaseプラン不要**: Cloud Functions（有料プラン）に依存せず、アプリ単体で完結するため、配布・利用が容易。
3.  **Electron親和性**: メインプロセスでトークンを持つことで、バックグラウンド処理（定期的な予定取得・通知）がウィンドウを閉じていても可能になる。

## 4. 実装プロセス (Step-by-Step)

### Phase 1: メインプロセス認証基盤の実装
*   [ ] `google-auth-library` の導入。
*   [ ] メインプロセスで `http://127.0.0.1` サーバーを一時的に立ち上げるロジックの実装。
*   [ ] システムブラウザを起動し、認可フローを開始するIPCハンドラの実装。

### Phase 2: トークン管理のセキュア化
*   [ ] 取得した Refresh Token を `safeStorage` (DPAPI/Keychain) で暗号化して保存するロジック。
*   [ ] アプリ起動時にトークンを復号し、アクセストークンをリフレッシュする処理。

### Phase 3: Rendererとの連携 (IPC)
*   [ ] Renderer (React) から「ログイン開始」を要求するIPC通信。
*   [ ] メインプロセスから「ログイン完了・アクセストークン」を受け取るリスナー。
*   [ ] 既存の `ChatService` が受け取ったアクセストークンを利用するように改修。

### Phase 4: Firebase Authとの二重管理の解消 (Optional)
*   Googleカレンダー用トークンとFirebase用トークンが別管理になるため、ユーザー体験（UX）として「2回ログイン」にならないよう、Firebaseへのカスタムトークンログイン（`signInWithCredential`）への連携も検討する。

## 5. ゴールと受け入れ基準
*   [ ] アプリを再起動しても、翌日になっても、カレンダー情報が自動的に更新されていること。
*   [ ] 「認証エラー」等のポップアップが出ないこと。
*   [ ] セキュリティ面で、トークンが平文で保存されていないこと。

---
**Note:** この改修は、AI秘書を「Webアプリのラッパー」から「真のデスクトップ・エージェント」へと進化させる重要なマイルストーンとなります。
