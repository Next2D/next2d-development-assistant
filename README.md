# Next2D AI SKILL

## 日本語

Next2D Framework および Next2D Player を用いた開発を強力にサポートするAI Skillです。
MVVM + Clean Architecture + Atomic Design パターンに従ったコード生成、WebGLレンダリングの最適化、直感的なアニメーション実装を支援します。

- **TypeScript First**: Next2D特有のクラス構造やAPIに完全対応
- **MVVM + Clean Architecture**: View/ViewModel, UseCase, Repository パターンの設計支援
- **Atomic Design**: Atom/Molecule/Organism/Page によるUIコンポーネント設計
- **Smart Debugging**: 描画エラーや論理エラーの迅速な特定と解決
- **Performance Tips**: WebGL/Canvasの負荷を抑えた実装提案
- **Multi-Platform**: Web / Steam (Windows/macOS/Linux) / iOS / Android 対応

### インストール

#### Claude Code

```bash
# 1. マーケットプレイスを追加
claude plugin marketplace add https://github.com/Next2D/skills

# 2. プラグインをインストール
claude plugin install next2d-development-assistant
```

#### Codex (OpenAI)

```bash
# 0. Codex CLI をインストール（未導入の場合）
npm i -g @openai/codex

# 1. このリポジトリ直下でスキルを登録
mkdir -p .agents/skills
ln -s ../../skills/next2d-development-assistant .agents/skills/next2d-development-assistant

# 2. Codex を起動（このディレクトリから）
codex
```

※ シンボリックリンクではなくコピーする場合は `cp -R skills/next2d .agents/skills/next2d` を使用してください。

### スペックソース

- [Next2D Player Specs](https://github.com/Next2D/player/tree/develop/specs/ja)
- [Next2D Framework Specs](https://github.com/Next2D/framework/tree/develop/specs/ja)
- [Next2D Template Specs](https://github.com/Next2D/framework-typescript-template/tree/develop/template/specs)

## English

AI Skill that powerfully supports development using the Next2D Framework and Next2D Player.
Provides code generation following MVVM + Clean Architecture + Atomic Design patterns, WebGL rendering optimization, and intuitive animation implementation support.

- **TypeScript First**: Fully compatible with Next2D's unique class structure and APIs
- **MVVM + Clean Architecture**: Design support for View/ViewModel, UseCase, Repository patterns
- **Atomic Design**: UI component design with Atom/Molecule/Organism/Page
- **Smart Debugging**: Rapidly identify and resolve rendering errors and logical errors
- **Performance Tips**: Suggest implementations that minimize WebGL/Canvas load
- **Multi-Platform**: Web / Steam (Windows/macOS/Linux) / iOS / Android support

### Installation

#### Claude Code

```bash
# 1. Add the marketplace
claude plugin marketplace add https://github.com/Next2D/skills

# 2. Install the plugin
claude plugin install next2d-development-assistant
```

#### Codex (OpenAI)

```bash
# 0. Install the Codex CLI (if needed)
npm i -g @openai/codex

# 1. Register the skill from the repo root
mkdir -p .agents/skills
ln -s ../../skills/next2d-development-assistant .agents/skills/next2d-development-assistant

# 2. Launch Codex from this directory
codex
```

If you prefer copying instead of symlinking, use `cp -R skills/next2d .agents/skills/next2d`.
