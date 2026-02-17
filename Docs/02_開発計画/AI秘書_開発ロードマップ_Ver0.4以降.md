# AI秘書開発ロードマップ (Ver 0.4.0以降)

**最終更新日**: 2026/01/30
**現状**: Ver 0.3.0 (Smart Sensing Update) 完了

本ドキュメントでは、AI秘書プロジェクトの今後の開発計画（Phase 4以降）を定義する。
Ver 0.3.0 で実現した「Windows上の感覚（Sensing）」と「記憶（Deep Memory）」を基盤とし、次は**「場所を選ばない利用（Cloud Native）」**と**「自然な対話（Voice）」**を目指す。

---

# AI秘書開発ロードマップ (Ver 0.4.0以降) & 次世代プラットフォーム仕様書

**最終更新日**: 2026/01/31
**現状**: Ver 0.3.0 (Smart Sensing Update) 完了

本ドキュメントは、Google Antigravityプラットフォームを活用した「次世代AI秘書（エグゼクティブ・エージェント）」の基本設計および開発ロードマップである。
ユーザーの「意思（Intent）」を最優先し、心理的摩擦を最小化する「謙虚な介入ロジック」を実装の中核に据える。

---

## 1. プロジェクトビジョン (Agent-First Approach)
*   **Vibe Coding**: 詳細なコード記述よりも「実現したい振る舞い」の定義に注力し、実装はAntigravityに自律させる。
*   **Artifact Transparency**: エージェントの思考過程（Task Lists, Recordings）を可視化し、ブラックボックス化を防ぐ。
*   **Humility (謙虚さ)**: AIは「絶対的な正解」ではなく「セカンドオピニオン」として振る舞い、最終決定権を常にユーザーに残す。

## 2. コア・コンセプト

### 2.1 業務の切り分け
| 領域 | AIエージェント (得意) | 人間 (聖域) |
| :--- | :--- | :--- |
| **スケジュール** | 空き時間抽出、調整、リマインド | 政治的優先度判断、断りの機微 |
| **情報処理** | 議事録作成、要約、検索 | 戦略的示唆、リスク検知 |
| **対話** | 定型ドラフト、日程打診 | 感情を伴う交渉、根回し |

### 2.2 [Removed] 意思の同期
*(Morning Huddle機能は Ver 0.4.0 計画から除外されました)*

### 2.3 謙虚な介入: Observe, Ask, Intervene (OAI)
ユーザーが作業から逸脱した際の行動指針。
1.  **Observe (観察)**: 「仕事（Productive）」と定義されない行動（SNS等）を一定時間検知。
2.  **Ask (質問)**: 「集中を邪魔して申し訳ありません。現在の私の把握していない緊急事項はありますか？」と無知を認める態度で尋ねる。
3.  **Intervene (介入)**: 「緊急」なら即座に謝罪して撤退。「実はサボっていた」なら、リスク（目標達成率の低下）を数値で示唆する。

### 2.4 集中度 (Focus Level)
*   **スコアリング**: 直近5分間のKPM（打鍵数）とアクティブウィンドウから算出。
*   **介入抑制**: 集中度が高い、または会議中の場合は通知を自動延期する。

---

## 3. アーキテクチャとセキュリティ

### 3.1 長期記憶システム (Three-Tier Memory)
*   **L0 (Working Memory)**: 直近の対話・コンテキスト。
*   **L1 (Short-Term)**: 数日間のログ、未完了タスク。
*   **L2 (Long-Term)**: BigQuery Vector Search を用いたサーバーレス検索。数年分の文脈を保持。

### 3.2 セキュリティ
*   **Row-Level Security (RLS)**: アカウント間データの完全隔離。
*   **Zero Data Retention**: 学習へのデータ利用禁止。
*   **KMS/CMEK**: ユーザー固有鍵による暗号化。

---

## 4. 📅 開発ロードマップ (Phase 4 - 6)

これまでの技術的マイルストーンに、新たな「意図駆動」要件を統合する。

| Version | Phase | テーマ | 主な実装機能 |
| :--- | :--- | :--- | :--- |
| **v0.4.0** | **Phase 4** | **L0 First & Infra (セキュア基盤)** | ・Cloud-Native移行 (Firestore/BigQuery)<br>・Multi-Device識別<br>・自動アップデート (electron-updater) |
| **v0.4.5** | **Phase 8.5** | **Security Hardening (Vertex AI)** | ・Vertex AI Migration (Zero Data Retention)<br>・Cloud Functions BFF (Auth/IAM)<br>・Document Security (RLS) |
| **v0.5.0** | **Phase 5** | **OAI Engine (謙虚な介入)** | ・集中度検知 (pynput活用)<br>・OAIフロー実装 (Observe-Ask-Intervene)<br>・アイゼンハワー・マトリクス可視化 |
| **v0.6.0** | **Phase 6** | **Executive Persona (音声・人格)** | ・エグゼクティブ秘書プロンプト統合<br>・音声対話 (Hey Secretary)<br>・リアルタイム議事録 |
| **v0.7.0** | **Phase 7** | **Context Intelligence (状況適応)** | ・Next Best Action (タスク提案)<br>・インテリジェンス・インジェクション<br>・シャドーワーク自動化提案 |
| **v0.8.0** | **Phase 9** | **Dynamic Memory (記憶管理)** | ・Dual-Layer Rule System (恒久/一時ルール)<br>・Chat Reminder Integration<br>・Auto-Cleanup Logic |

### 詳細: Phase 4 (Next Step)
*   **基盤構築**: Terraformを用いたFirestore + BigQuery RLS環境の構築。
*   **自動アップデート**: `electron-updater` の導入。

### 詳細: Phase 8.5 (Immediate Priority: Security)
*   **Vertex AI Migration**: 情報漏洩リスクを排除するため、Google AI Studio (API Key) から Vertex AI (Cloud Functions + IAM) へ移行する。
*   **Backend For Frontend (BFF)**: Electronから直接APIキーを使わず、Firebase Functions経由でAIを呼び出すセキュアな構成にする。

### 詳細: Phase 5 (OAI Engine & Humble Intervention)
*   **OAI実装**: 「逸脱」を定義し、謙虚な通知ロジック (Observe-Ask-Intervene) をコーディング。
*   **スマホ連携**: 離席時の通知をモバイルに飛ばす（Omni-Channel）。
*   **アイゼンハワー・マトリクス**: 提案を4象限で色分けし、優先度を直感的に可視化する。

### 詳細: Phase 7 (Context-Aware Intelligence) - *New*
*   **Next Best Action**: 会議終了時などに「次に行うべき最適タスク」を提案。
*   **Intelligence Injection**: 作業中のファイルに関連する情報をサイドパネルに自動提示。
*   **Shadow Work Automation**: 反復作業（コピペ等）を検知し、自動化スクリプトを提案。

### 詳細: Phase 9 (Dynamic Memory & Rules) - *New*
*   **Dual-Layer System**: ルールを「恒久 (Core)」と「一時 (Ephemeral)」に分離し、今日だけの指示（例：17時退社）と長期的な好み（例：敬語禁止）を両立させる。
*   **Chat Reminders**: "14時にリマインドして" といった指示を一時的なルールとして登録し、時間を過ぎたら自動消去する。


## 5. コスト試算 (2026予測)
*   **ライト運用**: 月額 5,000円〜 (Gemini 3 Flash)
*   **高度運用**: 月額 30,000円〜 (Gemini 3 Pro + 全ログ解析)

---

## 6. パーソナル・ステアリング：SECRETARY.md による「秘書の教育」

本システムは、ユーザーが明示的なルールを記述することでAIを即座に教育できる**「SECRETARY.md (個人秘書指針)」**機能を実装する。
これはAntigravityの `.agent/rules/` アーキテクチャを活用した高度な挙動制御機能である。

### 6.1 SECRETARY.md の構成と役割
ユーザーは自然言語で「絶対ルール」を記述し、AIに常時遵守させる。

| カテゴリー | 記述例 | AIの挙動への影響 |
| :--- | :--- | :--- |
| **定型行動** | 「金曜16時以降は、翌週の準備のため新規の打ち合わせを原則入れない」 | カレンダー調整時に自動ブロック / 代替案提示 |
| **人物重要度** | 「CFOの田中さんと、家族からの通知は常に最優先で表示せよ」 | 集中度スコア $S_{focus}$ 無視で即時通知 |
| **禁止事項** | 「私のプライベートなブラウジング履歴（趣味）を、仕事のメール要約に含めるな」 | ログ解析時のフィルタリング強制 |

### 6.2 実装・運用フロー (Antigravity活用)
1.  **ルールの永続読み込み**: ユーザーのFirestore内 `PersonalPolicy` コレクションを、System Instructionの最上位に注入。
2.  **ルールのリアルタイム更新**: ユーザーの発言（「月曜朝も会議OK」）から、AIが自律的に `SECRETARY.md` を書き換え、承認を求める。
3.  **「管理 tax」ゼロ**: 矛盾検知時にAI側から設定更新を提案し、メンテナンスの手間を最小化する。

### 6.3 顧客価値
AIを「使いにくい汎用ツール」から、**「自分の癖を完全に把握した、替えの効かないパートナー」**へと昇華させる核心機能。
これは「沈黙の質」の向上と並び、サブスクリプション継続の最大の動機付けとなる。


