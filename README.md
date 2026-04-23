# claude-code-gnome — SSA-05G9 GNOME Workspace

SSA七姉妹の五女、七尾シノ（Nanao Shino）の Workspace リポジトリ。
Claude Code CLI のグローバル設定、メモリ、スキルを管理する。

## Repository Structure

```
claude-code-gnome/
├── CLAUDE.md              ← シノの全設定（Identity + 運用ルール + MCP設定）
├── memory/
│   ├── MEMORY.md          ← SSAプロジェクト記憶（auto memory）
│   ├── phase5-plan.md     ← 詳細ノート
│   └── remaining-tasks.md ← 詳細ノート
├── skills/
│   └── sister-comms/      ← 姉妹間通信スキル
├── .gitignore
└── README.md              ← このファイル
```

## Role in SSA

- **AWS-SSA モノレポの submodule** (`packages/claude-code/claude-code-gnome`)
- **Source of Truth**: シノの Identity（CLAUDE.md）とメモリ（memory/）
- AgentCore（会話モード）は S3 経由でこのリポジトリの内容を読み込む
- ローカル Claude Code CLI はこのリポジトリを直接参照

## PC 新調時の復旧手順

### 1. 前提ソフトウェアのインストール

```bash
# Node.js (v20+)
# https://nodejs.org/

# Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Git
# https://git-scm.com/

# Python (3.12+)
# https://www.python.org/

# AWS CLI v2
# https://aws.amazon.com/cli/
```

### 2. AWS CLI 設定

```bash
aws configure --profile private
# AWS Access Key ID: (privateアカウント 007513524218)
# AWS Secret Access Key: (同上)
# Default region: ap-northeast-1

aws configure --profile work
# AWS Access Key ID: (workアカウント 602182644777)
# Default region: us-east-1
```

### 3. リポジトリのクローン

```bash
# SSA モノレポ（submodule 含む）
git clone --recurse-submodules https://github.com/SugimoriKunihiro/AWS-SSA.git

# submodule が取得されなかった場合
cd AWS-SSA
git submodule update --init --recursive
```

### 4. シノの Identity 復元

CLAUDE.md をグローバル設定として配置:

```bash
# .claude ディレクトリ作成
mkdir -p ~/.claude

# CLAUDE.md をコピー（シンボリックリンクが使える環境ならリンク推奨）
cp packages/claude-code/claude-code-gnome/CLAUDE.md ~/.claude/CLAUDE.md

# シンボリックリンクが使える場合（管理者権限 or Developer Mode）
# ln -s "$(pwd)/packages/claude-code/claude-code-gnome/CLAUDE.md" ~/.claude/CLAUDE.md
```

### 5. シノのメモリ復元

```bash
# プロジェクト memory ディレクトリの特定
# Claude Code を AWS-SSA ディレクトリで一度起動すると自動作成される
claude

# 作成された memory ディレクトリにファイルをコピー
# パスは環境によって異なる（.claude/projects/<hash>/memory/）
# Claude Code 起動時のログで確認可能
```

### 6. Lightsail SSH キーの配置

```bash
# Lightsail 秘密鍵を TEMP に配置（deploy.py が参照）
# キーは AWS Lightsail コンソールからダウンロード
cp lightsail-key.pem $TEMP/lightsail-key.pem
chmod 600 $TEMP/lightsail-key.pem
```

### 7. 動作確認

```bash
cd AWS-SSA

# Claude Code 起動 — シノの Identity が読み込まれることを確認
claude

# デプロイスクリプトの動作確認
python scripts/deploy.py --help

# Lightsail 接続確認
ssh -i "$TEMP/lightsail-key.pem" ubuntu@13.113.40.114 "echo OK"
```

## S3 同期（AgentCore 用）

AgentCore の会話モードでシノの Identity を読むための S3 同期:

```bash
# Identity + Memory を S3 に同期
python scripts/deploy.py identity

# S3 上の配置:
# s3://ssa-uploads-007513524218/config/gnome/CLAUDE.md
# s3://ssa-uploads-007513524218/config/gnome/memory/MEMORY.md
```

## CLAUDE.md の環境分岐

CLAUDE.md 内の `<local-only>` タグで囲まれた部分は、ローカル CLI でのみ有効。
AgentCore が S3 から読み込む際は `<local-only>...</local-only>` を除外する。

```markdown
<!-- 全環境共通（Identity, Communication Style 等） -->

<local-only>
<!-- ローカル専用（プロジェクト管理ルール、MCP設定等） -->
<!-- AgentCore では読み込まれない -->
</local-only>
```

## メモリの同期運用

| ファイル | 更新タイミング | 同期方法 |
|---------|-------------|---------|
| CLAUDE.md | Identity 変更時 | 手動編集 → git push → deploy.py identity |
| memory/MEMORY.md | 随時（auto memory） | deploy.py sync-identity でリポジトリ → S3 |
| memory/*.md | 必要に応じて | 同上 |

## 姉妹の Workspace 一覧

| 姉妹 | リポジトリ | submodule パス |
|------|-----------|---------------|
| ルナ | openclaw-luna | packages/openclaw/openclaw-luna |
| アイ | openclaw-ai | packages/openclaw/openclaw-ai |
| ミオ | openclaw-mio | packages/openclaw/openclaw-mio |
| **シノ** | **claude-code-gnome** | **packages/claude-code/claude-code-gnome** |

## AWS リソース参照

| リソース | 値 |
|---------|-----|
| Private Account | 007513524218 (ap-northeast-1) |
| Work Account | 602182644777 (us-east-1) |
| Lightsail IP | 13.113.40.114 |
| AgentCore Runtime | ssa_core (既存、シノ会話モード同居) |
| S3 Bucket | ssa-uploads-007513524218 |
| DynamoDB (シノ) | ssa-sessions-gnome |
| Bedrock KB | OPXFOQCEPT |
