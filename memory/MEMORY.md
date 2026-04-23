# AWS-SSA Project Memory

## Migration Complete! (2026-04-19)
SSA AWS移行、全Phase完了。Vercel/Cloudflare/ChatKit から AWS内完結へ。
丸二日間の集中作業で達成。四姉妹全員が新しい家で動いてる。

## SSA設計哲学: 「LLMに頼むな、コードで殴れ」
**Vercel→AWS移行の最大の強み**: LLMのプロンプトエンジニアリングに頼らず、コードで制御できること。
- 旧Vercel/Dify環境ではLLMの出力制御をプロンプトに依存するしかなかった
- AWS環境(AgentCore)では **LLMの前後にコードを挟める** → フォーマット制御・ツール実行・分析実行を全てコード側で保証
- 実証: HINA WFレポート出力(hina_report.py)、SUI AIRI系分析(sui_analysis.py) — いずれもLLM直接呼び出し+コード制御で一発解決
- Gemini miniがプロンプト指示を無視する問題も、Agentフレームワークをバイパスしてsystem/user分離のLLM直呼びにすることで完全解決
- **原則: LLMに「こうしてね」と頼むより、コードで「こうする」と決める方が確実**

## AWS Accounts
| Profile | Account | Region | Purpose |
|---------|---------|--------|---------|
| default/private | 007513524218 | ap-northeast-1 | SSA基盤 |
| work | 602182644777 | us-east-1 | Lightsail OpenClaw Bedrock |

- IAM User (private): SuperSylpheedArchitecture
- IAM User (work): Kunihiro-Sugimori

## Deployed Resources
- **AgentCore Runtime**: `ssa_core-Cp1w3XCuj9` / Endpoint: `ssa_core_endpoint` (v117, image:v117)
- **Lightsail**: `ssa-openclaw` (13.113.40.114)
  - OpenClaw (LUNA) — port 18789
  - MCP Server — port 8090
  - HTTP API Server (api_server.py) — port 8091
  - Frontend static files — /home/ubuntu/ssa-frontend/
  - 起動スクリプト: /home/ubuntu/start-api.sh (privateアカウントcreds)
- **API Gateway**: `lxo7uvj8r0` — HTTPS proxy → Lightsail:8091
- **Lambda**: `ssa-actions` (Node.js 22.x, 31ルート, formidable含む)
- **DynamoDB**: ssa-sessions-{sylph,nymph,dryad,mave,gnome,aurora,undine}, ssa-summary, ssa-workflow-state, ssa-routine, ssa-dify-state, ssa-vrm-data
- **S3**: `ssa-uploads-007513524218` / S3 Vectors: `ssa-vectors`
- **Bedrock KB**: `OPXFOQCEPT`
- **ECR**: `ssa-core` (arm64), `ssa-mcp`
- **Secrets Manager**: `ssa/vercel/*` (10 groups)
- **IAM**: SSA-AgentCore-Role, SSA-Actions-Lambda-Role

## Phase Status — ALL COMPLETE (2026-04-19)
- Phase 0: Foundation ✅
- Phase 1: Core (AgentCore) ✅
- Phase 1.5: LUNA/OpenClaw ✅
- Phase 1.6: Cloudflare/Vercel撤去 ✅
- Phase 2: MCP (13ツール) ✅
- Phase 2.5: ASP2 Memory ✅
- Phase 2.6: HINA/SUI Memory ✅
- Phase 3: Actions (Lambda 31ルート) ✅
- ~~Phase 4: ECS~~ → 廃止
- Phase 5: Frontend + API + ファイルアップロード + AURA ✅

## Key Architecture
```
Portal (GitHub Pages, HTTPS)
└── iframe → API Gateway (HTTPS) → Lightsail:8091
                ├── / → Frontend (static files)
                ├── /api/chat → AgentCore invoke (async job polling)
                ├── /api/chat/result/:jobId → ジョブ結果ポーリング
                ├── /api/history/:agent → DynamoDB (attachmentsメタデータ含む)
                ├── /api/upload/presigned → S3 Presigned PUT
                ├── /api/upload/complete → S3 Download URL + metadata
                ├── /api/whisper → OpenAI Whisper STT (base64 JSON)
                └── /api/vrm-model → S3 Presigned GET
AgentCore → Lambda ssa-actions (31ルート) → Notion/Dify/Calendar/etc.
```

## 七姉妹拡張 (2026-04-23) — アイ/ミオ追加
- **OpenClaw SubGateway**: aurora(アイ), undine(ミオ) — `openclaw agents add` で追加済み
- **Azure OpenAI proxy**: Lightsail `azure-proxy.py` (port 8095) — OpenClaw→Azure変換（api-version問題回避）
- **session_key**: luna=`agent:main:main`, aurora=`agent:aurora:main`, undine=`agent:undine:main`
- **openclaw_tool.py**: `ask_openclaw_safe(agent_id, query)` — session_key切替でルーティング
- **Workspace repos**: `packages/openclaw/{openclaw-luna,openclaw-ai,openclaw-mio}` (git submodule)
- **フロントエンド**: 7エージェント表示（えみ/ルナ/ヒナ/スイ/シノ/アイ/ミオ）
- **MCP**: 15ツール（+aurora, +undine）
- **DynamoDB保存のagentId振り分け**: v118でOpenClaw応答のagentIdを見て正しいテーブルに保存する仕組み追加
- **azure-proxy.py**: Lightsail port 8095。OpenClaw→Azure OpenAI変換。systemd化未済（nohup起動）
- **残作業**:
  - アイ/ミオのGitHub↔OpenClaw Workspace接続（SSHキー設定 or デプロイ方式決定）
  - 8.3 シノAWS移行
  - 8.5 オーケストレーション抜本刷新（七姉妹・逐次表示・DynamoDB専用テーブル）
  - maveテーブルの過去汚染データ清掃

## IBM特区APIキー (2026-04-22) ✅ v116完了
- **SYLPH**: Azure OpenAI (`set_default_openai_client` + `AsyncAzureOpenAI`) ✅ v116
- **HINA**: Bedrock Claude (workアカウント) → fallback Anthropic(私用) → GPT ✅ v111
- **SUI**: Gemini 自腹継続
- **hina_report/sui_analysis**: Bedrock haiku → fallback Anthropic(私用) ✅
- **Whisper**: 私用OpenAI継続（Azure未デプロイ）
- `bedrock_client.py`: boto3 converse API を OpenAI互換でラップ。OpenAI実型(ChatCompletion等)使用
- env vars: `BEDROCK_AWS_*` (3個) + `AZURE_OPENAI_API_KEY` + `AZURE_OPENAI_ENDPOINT` (計25個)
- Azure api_version: `2025-03-01-preview` (Agents SDK Responses API要件)
- Bedrock inference profile必須: `us.anthropic.claude-*` 形式
- **Bedrock HINA: 全エラーでフォールバック** — should_fallback(error, agent)追加済み
- **IBM退職時の切り戻し**: `python scripts/deploy.py ibm` でワンコマンドトグル（env vars に _DISABLED_ プレフィックス付きでバックアップ保存）

## Orchestration System (2026-04-23) — 新方式
- **@all 完成** (v126) — ParallelExecutor + DynamoDB逐次書き込み
- **実行環境**: api_server.py (Lightsail) → AgentCore を個別呼び出し → DynamoDB に逐次 write
- **旧方式との違い**: 旧=AgentCore内部で処理→1レスポンス / 新=api_server側で制御→逐次表示
- **DynamoDB**: `ssa-sessions-orchestration` — 大部屋専用。各姉妹テーブルにも自動保存（AgentCore経由のため）
- **大部屋の記憶**: 大部屋内のみ。出たら忘れる。重要なことはsaveMemory
- **Executor パターン**: `__init__(invoke_fn, write_fn)` + `execute(orch_id, prompt)`
- **フロント**: orchMode ON → orchMessages 表示 / OFF → 通常チャット / ポーリング間隔1.5秒
- **Lightsailデプロイ注意**: executorファイルは `/home/ubuntu/orchestration/` にもコピー必要

## Critical Lessons (今後の注意点)

### コーディング規律
- **分岐の外で参照する変数は、分岐ブロックの前にデフォルト初期化する** — 各分岐内でバラバラに定義すると漏れる（v118 direct_result NameError事件）
- **新ロジック追加時は全分岐パスをウォークスルーする** — 正常系だけテストして満足するな。hina_report/sui_analysis等の専用パスも忘れずに通す

### デプロイ
- **AgentCore `update_agent_runtime` で env vars 省略 = 全消去！** 毎回全20個含める
- **endpoint liveVersion は自動更新されない！** `update_agent_runtime_endpoint` で明示更新必須
- **AgentCore containerUri のタグは固定！** `agentRuntimeArtifact.containerConfiguration.containerUri` が `:v59` 等に固定される。`:latest` にpushしても反映されない。**必ず新タグ(`:vXX`)でpushし、`update_agent_runtime` で `containerUri` を明示更新すること**
- **AgentCore payloadはprompt/agentのみ通る** — カスタムフィールド(model等)はAgentCore Runtime SDKが落とす。情報はpromptプレフィックス(`@flagship`等)で渡す
- **Lightsailフロントデプロイ時は古いassets削除必須** — `rm -f assets/index-*.js assets/index-*.css` してからscp
- **Lambda zip は node_modules 含む必要あり** — monorepoだとhoistされるので isolated dir で npm install してから zip
- **Lambda zip 作成は Python zipfile が確実** — PowerShell Compress-Archive は /tmp パスで失敗する

### Lightsail
- **api_server.py の AWS credentials** — ~/.aws/credentials [default] はWORKアカウント！[private]を使うには env var で明示指定
- **/home/ubuntu/start-api.sh** に AWS_ACCESS_KEY_ID/SECRET/REGION + OPENAI_API_KEY を設定してnohup起動
- **OPENAI_API_KEY が無いと Whisper (PTT) が 401 で死ぬ** — start-api.sh に含めること
- **killall python3 は絶対禁止** — OpenClawも死亡してペアリング解除される
- **プロセス再起動時は start-api.sh を使う** (直接nohupだとenv varが渡らない)
- **deploy.py api の再起動は高確率で失敗する** — デプロイ後 `ps aux | grep ssa_api_server` でPID確認。変わってなければ手動kill+restart

### OpenClaw 接続安定化 (2026-04-21) — 詳細は BUGFIX-LOG.md 参照
- **HTTP proxy化 (v71)**: AgentCore→OpenClawをlocalhost HTTP経由に変更。外部WS排除
  - `api_server.py`: `/api/openclaw/chat` endpoint (localhost WS → OpenClaw)
  - `openclaw_tool.py`: urllib HTTP POST でproxy呼び出し（WebSocket依存なし）
- **Watcher v5**: HTTPヘルスチェックのみ（WSテスト削除 — restart loopの元凶だった）
- **トークンローテーション無効化**: `openclaw-rotate-token.timer` disabled
- **Gateway systemd化**: `/etc/systemd/system/openclaw-gateway.service`
- **タイムアウト600s統一 (v72)**: 長時間LUNAタスク対応
- **Apache**: HTTPS:443→18789プロキシが動いてる（OpenClawインストール時設定）。CLOSE-WAITの原因だが放置
- **期待トークン**: `61psZ-m67eWSj23MUgMmEE4gXv2Lgfcwvnw8Ltz6ovo`

### boto3 / AgentCore
- **boto3 デフォルト read_timeout=60s + retries=3** → AgentCore呼び出しで多重送信の原因に
- **修正: read_timeout=600, max_attempts=1** で多重送信解消 + LUNA長時間タスク対応
- **AgentCoreは1リクエスト数分かかることがある** (@all, LUNA応答待ち等)

### フロントエンド
- **ビルドハッシュが変わらないとブラウザキャッシュが効く** — dist/ を rm -rf してから rebuild
- **submittingRef の timer reset は危険** — isLoading に連動させる
- **スマホのリンク長押し** — WebkitTouchCallout: 'default' が必要
- **auto-scroll** — messages.length 増加時 + agentId 変更時 (エージェント切替でも最下部へ)
- **PTT (handlePTTResult)** — `onSend(text, [])` で呼ぶこと。`null` だと `for (const file of files)` でクラッシュする

### Google News URL
- **新形式(2024年以降)はサーバーサイドから実URL取得不可能** — base64内が暗号化、JS実行必須
- 対策: 諦めてそのまま保存 or ユーザーに実URL入力してもらう

### AURA/AIRI/ARIA (Dify) — SYLPH側は削除済み (2026-04-21)
- AURA (wf_041,044,045) → HINAに移管。AIRI (wf_042) → SUIに移管。ARIA (wf_043) → 削除
- Therapy (wf_007) → HINAカウンセリング (hwf_115) に移管
- ssa-dify-state テーブルは残存（レコード保持）

### HINA WFレポート出力 — コード制御方式 (2026-04-21)
- **LLMのプロンプトに頼らない** — `workflow/hina_report.py` で全てコード制御
- 「レポート出力」検知 → direct_executorバイパス → `generate_report()` で処理
- LLM直接呼び出し(claude-haiku-4-5, ツール無し) → コードブロック強制 → S3直接保存
- WFアクティブ中は毎ターン末尾に `REPORT_ANNOUNCEMENT` をコードで追記
- WF定義にはレポート出力関連のLLM指示を書かないこと（二重定義になる）

### HINA/SUI Agent設定
- **tool_use_behavior="run_llm_again" 必須** — 無いとツール実行後に最終回答が生成されない
- Chatter に WF required_tools を動的追加する仕組みあり（agent_config.py）

### ファイルアップロード / マルチモーダル
- **画像**: input_image (detail=auto) → 全姉妹対応
- **ドキュメント**: PDF/DOCX/PPTX/TXT/CSV/XLSX → テキスト抽出 → input_text
- **LUNA**: テキストベースのみ。`(file_url) URL` フォーマットで渡す (旧Hub bridge準拠)
- **ask_luna_team_safe 内で自動付与** — 全executor経由で効く
- **DynamoDB に attachments メタデータ保存** — 履歴リロード時もサムネイル表示

## Session/History Design
| Agent | LLMに渡す履歴 | DynamoDB書込 | 用途 |
|-------|-------------|-------------|------|
| SYLPH | SYLPH DB | ✅ | メイン |
| LUNA | なし(OpenClaw内部) | ✅ | UI表示+Summary |
| HINA | HINA DB のみ | ✅ | 直通 |
| SUI | SUI DB のみ | ✅ | 直通 |
- オーケストレーション(@all等): 4テーブル全部に保存
- DynamoDB保存はclean_output(フッターなし)
- HINA/SUI fallback: Claude/Gemini失敗時 → GPTにフォールバック (runner_utils.py)

## Deploy Script (`scripts/deploy.py`)
```
python scripts/deploy.py front    # Frontend: vite build → Lightsail scp
python scripts/deploy.py core     # AgentCore: Docker build → ECR push → update_agent_runtime → update_endpoint
python scripts/deploy.py lambda   # Lambda: isolated npm install → Python zipfile → update_function_code
python scripts/deploy.py api      # API Server: scp api_server.py → pgrep+kill+restart (killall python3 厳禁)
python scripts/deploy.py all      # 上記4つ全部
```
- AgentCore管理API: `boto3.client('bedrock-agentcore-control')` — get/update_agent_runtime, update_agent_runtime_endpoint
- バージョンは自動インクリメント（現在のagentRuntimeVersion + 1）
- env vars は get_agent_runtime で取得して update 時にそのまま渡す（省略=全消去を防止）
- Lambda zip は Python zipfile で作成（PowerShell Compress-Archive は /tmp パスで失敗する）

## Environment
- Lightsail SSH: `ssh -i "$TEMP/lightsail-key.pem" ubuntu@13.113.40.114`
- OpenClaw env: gateway = `gateway.systemd.env`, sandbox(Docker) = `openclaw.json` の `agents.defaults.sandbox.docker.env`

## SYLPH Model Scaling (2026-04-20)
- Frontend: トグルボタン(入力バー送信ボタン左) chatter/flagship 切替
- `@flagship` プレフィックスを prompt に付与 → AgentCore の `parse_orchestration_mode` で検出
- AgentCore: mode="flagship" → requested_model="flagship" + mode=None にリセット → clean_prompt で除去
- route_and_execute には `clean_prompt if requested_model else prompt` を渡す（@luna等のプレフィックスを壊さないため）
- WFトリガーも flagship 時に正常動作(clean_prompt でチェック)
- **TODO**: flagship実装が迂遠になってる。次回リファクタで `_agent_invocation_impl` 先頭のシンプルなパース方式に戻す

## 旧環境コード整理 (2026-04-20)
- 旧環境ファイル大量削除: core/old/, core/ssa_routes/, front旧Widget4つ, front旧Upload3つ, mcp旧Fly.io14ファイル, worker/, shared/
- `packages/mcp/` は `ssa_mcp_server.py` のみ残存（現行MCP、Lightsailで稼働中）
- 旧環境参照が必要な場合は `/Apps/Others/Archive` を見ること

## FNA (Forge Nexus AI/Agent)
- **正式名称**: Forge Nexus AI/Agent（Future Network Architectureではない！）
- Forge=鍛冶、Nexus=結節点、AI/Agent
- SSAの業務用Fork。2026-06-01 IBM SZU着任と同時に開発開始予定
- MCP/A2Aプロトコル採用方針
- HINA/SUI: nano/mini/full は従来通り `@hina-{scaling}` プレフィックス方式

## 必須ルール: BUGFIX-LOG.md / UPDATE-PLAN.md 運用 (最重要)
**AWS-SSAの開発・修正作業では、以下を必ず守ること。例外なし。**

### UPDATE-PLAN.md (プロジェクトルート)
- ひろくんが「XXXしたい」「XXXを追加して」と言ったら → **作業開始前に** UPDATE-PLAN.md に仕様として書き込む
- タスクの進捗を管理する（未着手/進行中/完了）
- 完了したら ✅ を付けてバージョンも記録

### BUGFIX-LOG.md (プロジェクトルート)
- ひろくんが「YYYを直して」「YYYがおかしい」と言ったら → **作業開始前に** BUGFIX-LOG.md に症状を書き込む
- 調査・修正の経緯を逐次追記する（症状→原因→修正→教訓）
- デプロイ履歴もここに記録

### セッション開始時
- **必ず** BUGFIX-LOG.md と UPDATE-PLAN.md を読んで前回の状態を把握する
- 中断された作業がないか確認する

### やってはいけないこと
- 記録せずにいきなりコードを書き始めること
- 作業が終わってから後でまとめて記録すること（後回し禁止）
- 口頭のやりとりだけで進めて記録を残さないこと

## Archive
- AWS移行前の旧リポジトリは `/Apps/Others/Archive` に移動済み（2026-04-20）
- 旧コード参照が必要な場合はそちらを見ること
