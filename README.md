# Claude Code Demo - Agentic Coding 学習プロジェクト

[Claude Code is a Beast](https://www.youtube.com/watch?v=...) 動画で紹介された高度な設定を、実際のアプリケーション実装を通じて体験できる学習用プロジェクトです。

## このプロジェクトの目的

**[RealWorld (Conduit)](https://realworld-docs.netlify.app/)** アプリ - Medium.com クローンのブログプラットフォーム - を実装しながら Claude Code の様々な機能を実習します：

- **Skills**: コンテキスト基盤の自動活性化指針
- **Hooks**: ツール使用前/後の自動実行スクリプト
- **Custom Agents**: 特化した作業のためのカスタムエージェント
- **Slash Commands**: 繰り返し作業のためのカスタムコマンド

---

## プロジェクト構造

```
claude-code-demo/
├── .claude/                    # Claude Code 設定
│   ├── settings.json           # 権限、hooks 設定
│   ├── skills/                 # 自動活性化スキル
│   │   ├── backend-dev-guidelines/
│   │   ├── frontend-dev-guidelines/
│   │   ├── route-tester/
│   │   ├── error-tracking/
│   │   └── skill-rules.json    # スキル活性化ルール
│   ├── hooks/                  # 自動化フックスクリプト
│   ├── agents/                 # カスタムエージェント定義
│   └── commands/               # スラッシュコマンド定義
├── docs/                       # ハイレベル設計文書
│   ├── PRD.md                  # 製品要求仕様
│   ├── TechStack.md            # 技術スタック
│   ├── Architecture.md         # システムアーキテクチャ
│   └── API-Spec.md             # API 仕様
├── frontend/                   # React 19 + TypeScript + MUI v7
├── backend/                    # Express + TypeScript + Prisma
└── CLAUDE.md                   # Claude Code メイン指針
```

### docs/ vs .claude/skills/ の関係

| フォルダ | 役割 | 対象 |
|----------|------|------|
| `docs/` | ハイレベル設計文書 | 人が読む文書 |
| `.claude/skills/` | エージェント実行指針 | Claude が使用する指針 |

`docs/` の設計内容を `.claude/skills/` のリソース文書に変換し、Claude がコーディング時に参照できるようにします。

---

## 核心概念

### 1. Skills（スキル）

**スキル**は Claude が特定の作業を実行する際に自動的にロードされるナレッジベースです。

```
.claude/skills/
├── backend-dev-guidelines/     # バックエンド開発ガイドライン
│   ├── SKILL.md                # メインスキルファイル
│   └── resources/              # 詳細リソース文書
│       ├── architecture-overview.md
│       ├── routing-and-controllers.md
│       └── ...
├── frontend-dev-guidelines/    # フロントエンド開発ガイドライン
└── skill-rules.json            # スキル活性化ルール
```

**自動活性化方式** (`skill-rules.json`):
- `pathPatterns`: 特定ファイル編集時 (`backend/**/*.ts`)
- `keywords`: プロンプトにキーワード含む時 (`"backend"`, `"API"`)
- `intentPatterns`: ユーザー意図パターンマッチング

### 2. Hooks（フック）

**フック**は特定のイベント発生時に自動的に実行されるスクリプトです。

```json
// .claude/settings.json
{
  "hooks": {
    "UserPromptSubmit": [...],  // プロンプト送信時
    "PostToolUse": [...],       // ツール使用後
    "Stop": [...]               // 作業完了時
  }
}
```

**このプロジェクトのフック**:
- `skill-activation-prompt.sh`: プロンプト分析後に関連スキル推奨
- `post-tool-use-tracker.sh`: ファイル編集追跡
- `tsc-check.sh`: TypeScript コンパイルチェック

### 3. Custom Agents（カスタムエージェント）

**カスタムエージェント**は特化した作業のために定義された AI ペルソナです。

```
.claude/agents/
├── auth-route-debugger.md      # 認証問題デバッグ
├── code-refactor-master.md     # コードリファクタリング
├── frontend-error-fixer.md     # フロントエンドエラー修正
└── ...
```

### 4. Slash Commands（スラッシュコマンド）

**スラッシュコマンド**は繰り返し作業のためのカスタムコマンドです。

```
.claude/commands/
├── dev-docs.md                 # 開発文書生成
├── dev-docs-update.md          # 開発文書更新
└── route-research-for-testing.md
```

使用法: `/dev-docs 認証システム実装計画`

---

## 始め方

### 前提条件

- Node.js 20+
- pnpm
- Claude Code CLI ([インストールガイド](https://claude.ai/code))

### インストール

```bash
# リポジトリクローン
git clone https://github.com/serithemage/claude-code-demo.git
cd claude-code-demo

# 依存関係インストール
pnpm install

# データベースマイグレーション
pnpm --filter backend prisma migrate dev

# 開発サーバー起動
pnpm dev
```

### Claude Code 実行

```bash
# プロジェクトディレクトリで
claude

# または特定作業開始
claude "バックエンドに新しい API エンドポイントを追加して"
```

---

## 学習ガイド

### レベル 1: 基本構造の理解

1. **CLAUDE.md を読む**: プロジェクト全体の指針を確認
2. **docs/ を探索**: ハイレベル設計文書を理解
3. **settings.json を確認**: 権限とフック設定を理解

### レベル 2: Skills 体験

1. `backend/` フォルダで `.ts` ファイルを編集してみる
2. 自動的に `backend-dev-guidelines` スキルが活性化されるか確認
3. `skill-rules.json` で活性化条件を分析

```bash
# 例: バックエンドファイル編集
claude "backend/src/routes/ に新しいルートを追加して"
# → backend-dev-guidelines スキル自動活性化
```

### レベル 3: Hooks 分析

1. `.claude/hooks/` フォルダのスクリプトを分析
2. `settings.json` の hooks 設定とのマッピングを理解
3. フック実行ログを観察

### レベル 4: カスタマイズ

1. 新しいスキルを追加してみる
2. `skill-rules.json` に活性化ルールを追加
3. カスタムスラッシュコマンドを作成

---

## 技術スタック

| 領域 | 技術 |
|------|------|
| **Frontend** | React 19, TypeScript, MUI v7, TanStack Query/Router, Vite |
| **Backend** | Node.js 20, Express 4, TypeScript, Prisma 5, SQLite |
| **認証** | JWT (localStorage 保存) |
| **テスト** | Vitest, Testing Library, Supertest |

---

## 主要ファイル説明

| ファイル | 説明 |
|----------|------|
| `CLAUDE.md` | Claude Code がプロジェクト作業時に参照するメイン指針 |
| `.claude/settings.json` | 権限、MCP サーバー、フック設定 |
| `.claude/skills/skill-rules.json` | スキル自動活性化ルール定義 |
| `docs/Architecture.md` | システムアーキテクチャ（Mermaid ダイアグラム含む）|

---

## 便利なコマンド

```bash
# 開発サーバー
pnpm dev                        # 全体（フロントエンド + バックエンド）
pnpm --filter frontend dev      # フロントエンドのみ
pnpm --filter backend dev       # バックエンドのみ

# テスト
pnpm test                       # 全テスト
pnpm --filter backend test      # バックエンドテストのみ

# ビルド
pnpm build                      # プロダクションビルド

# リント/フォーマット
pnpm lint
pnpm format
```

---

## 参考資料

- [Claude Code 公式ドキュメント](https://docs.anthropic.com/claude-code)
- [RealWorld スペック](https://realworld-docs.netlify.app/)
- [Skills 詳細ガイド](.claude/skills/README.md)
- [Hooks 設定ガイド](.claude/hooks/README.md)
- [Agents ガイド](.claude/agents/README.md)

---

## 言語バージョン

このプロジェクトは多言語で提供されています：

| ブランチ | 言語 | リンク |
|----------|------|--------|
| `main` | English | [View](https://github.com/serithemage/claude-code-demo/tree/main) |
| `korean` | 한국어 | [View](https://github.com/serithemage/claude-code-demo/tree/korean) |
| `japanese` | 日本語 (現在) | [View](https://github.com/serithemage/claude-code-demo/tree/japanese) |

---

## ライセンス

MIT License

---

## 貢献

Issue と PR を歓迎します。Agentic Coding 学習に役立つ改善点があればご提案ください。
