# AI秘書 エージェント・ファースト開発タスク概要

本ドキュメントは、「[AI秘書_エージェント・ファースト設計思想](../00_要件定義/AI秘書_エージェント・ファースト設計思想.md)」に基づき、実装済み機能と未実装タスクを整理したものである。

---

## 🟢 実装済み機能 (Implemented)
- **垂直型エージェント基盤**: チャット、ファイル取込、集中度検知
- **2層ルールシステム**: 恒久/一時ルール
- **外部連携**: Google/Microsoft認証
- **Mermaid PNG保存**

---

## 🟡 フェーズ1: 信頼の境界線と自律探索 (Trust Boundaries & Discovery)

「勝手に触らない」安心感を担保した上で、「自律的に探す」機能を実装する。

### Step 1: 信頼境界の構築 (Trust Boundaries)
- [ ] **許可スコープ管理 (Allowed Scopes)**
    - [ ] 初回起動時（または設定画面）で「AIが触っていいフォルダ」を指定するUIの実装。
    - [ ] `SECRETARY.md` (またはFirestore) への `ALLOWED_SCOPES` 保存ロジック。
- [ ] **ファイルシステム・バリデーション (Validation Layer)**
    - [ ] `validatePath(path)` 関数の実装: 指定パスが許可スコープ内にあるか厳密にチェック。
    - [ ] Electronメインプロセス側のファイル操作API (IPC) にバリデーションを適用し、物理的にアクセスを遮断。

### Step 2: 自律探索と実況 (Autonomous Discovery)
- [ ] **File Crawler Skill**
    - [ ] `validatePath` を通過する安全な `file-crawler` の実装。
    - [ ] トリガー実装: カレンダー予定 / チャット文脈。
- [ ] **実況UI (Narration Widget)**
    - [ ] 探索状況（「Analyzed...」「Blocked...」）のストリーミング表示。

---

## 🟡 フェーズ2: ローカル優先記憶 (Local-First Memory)
- [ ] **SQLite-vec 統合**
    - [ ] ローカルベクトルストア (`sqlite-vec`) のセットアップ
    - [ ] プライバシーフィルター実装

---

## 🟡 フェーズ3: コスト最適化 (Cost Optimization)
- [ ] **多層フィルタリング**
    - [ ] Gemini Flash-Lite トリアージ
    - [ ] LLMLingua-2 圧縮

---

## 🟡 フェーズ4: 謙虚な介入 (OAI)
- [ ] **OAIフロー**
    - [ ] 介入ロジックの調整
    - [ ] 承認フロー (Just-in-Time 権限要求含む)

---

## 🔒 ライセンス・承認制機能
- [ ] クエリ回数制限 (Quota)
- [ ] 自動フォールバック
- [ ] 組織ナレッジ連携
