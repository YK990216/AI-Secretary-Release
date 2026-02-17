# Debug Report: Issue #11 Calendar Auto-Refresh Failure

## 1. 現状の実装 (Current Implementation)

### A. 認証フロー (Frontend: `App.tsx`)
1.  **Google Login**: `signInWithPopup(auth, provider)` を使用。
2.  **Refresh Token Extraction**: ログイン成功時、`result._tokenResponse?.refreshToken` からリフレッシュトークンを取得しようと試みる。
    *   *注: `_tokenResponse` はFirebaseの非公開プロパティであり、バージョンアップや環境によって `undefined` になる可能性がある。*
3.  **Secure Storage**: 取得できた場合、`window.ipcRenderer.saveSecureToken(token)` を呼び出す。

### B. トークン保存 (Backend: `electron/main.ts`)
1.  **Encryption**: Electronの `safeStorage` API (Windows DPAPI等) を使用して文字列を暗号化。
2.  **File Save**: `%APPDATA%/.../secure_token.bin` にバイナリとして保存。

### C. 自動更新フロー (Feature v0.6.3)
1.  **API Call**: `fetchUpcomingEvents` (ChatService/BriefingSection) が Google Calendar API を叩く。
2.  **401 Error Handling**: ステータス `401 (Unauthorized)` を検知した場合:
    *   `window.ipcRenderer.loadSecureToken()` を呼び出し、保存されたリフレッシュトークンを復号化して取得。
    *   `refreshGoogleToken` (firebase.ts) を呼び出し、新しい Access Token を取得。
    *   新しいトークンでAPIリクエストを再試行。

## 2. 発生している問題 (The Problem)

### 現象
*   ユーザーのスクリーンショットにて、AIが **「アクセス権限が切れているため今後のスケジュールを確認できません」** と回答。
*   これは `ChatService.ts` 内で自動更新（再試行）が失敗し、最終的に401エラーとして処理された場合に返されるメッセージと一致する。

### 原因の仮説 (Hypothesis)
以下のいずれかの理由で、**リフレッシュトークンが正しく保存されていない**、または**利用できない**状態にあると考えられる。

1.  **Refresh Tokenの取得失敗**:
    *   `signInWithPopup` の戻り値 `_tokenResponse.refreshToken` が `undefined` だった場合、アプリは「保存失敗」をユーザーに通知せず、警告ログ (`console.warn`) を出すのみである（ユーザーは気づかない）。
    *   Google OAuthは、`access_type: 'offline'` かつ `prompt: 'consent'` を指定しない限り、**初回ログイン時しかリフレッシュトークンを返さない**仕様がある。

2.  **IPC/Storageのエラー**:
    *   `safeStorage` が復号化に失敗した（Windows OSのアップデートやパスワード変更等でKeyが飛んだ可能性）。

## 3. Deep Research への依頼事項

以下の点について調査・解決策を求めます。

1.  **Firebase JS SDK (v9/v10) における正式な Refresh Token 取得方法**:
    *   非公開API `_tokenResponse` に依存しない、堅牢な取得方法は存在するか？
2.  **Google OAuth "Force Refresh Token"**:
    *   既存ユーザーが再ログインした際にも確実にRefresh Tokenを取得するためのパラメータ設定（`prompt: 'consent'` の挙動など）。
3.  **Electron `safeStorage` の永続性**:
    *   Windows Update等で復号化不能になるケースへの対策。

## 4. 参考コード (Current Code Snippets)

### App.tsx (Login Logic)
```typescript
const result = await signInWithPopup(auth, provider);
// ...
// ⚠️ Undocumented property usage
// @ts-ignore
const refreshToken = result._tokenResponse?.refreshToken;
if (refreshToken) {
    await window.ipcRenderer.saveSecureToken(refreshToken);
} else {
    // ⚠️ Silent failure (Log only)
    console.warn("No refresh token received. Auto-login may not work.");
}
```

### ChatService.ts (Retry Logic)
```typescript
if (response.status === 401) {
    const refreshToken = await window.ipcRenderer.loadSecureToken();
    if (refreshToken) {
        // Refresh & Retry...
    }
}
// If retry fails or no token found:
if (response.status === 401) return "（保護された情報: ...許可が切れました...）";
```
