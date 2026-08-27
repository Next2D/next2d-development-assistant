---
name: next2d-development-assistant
description: >
  Next2D Player/Framework 開発支援。MVVM+CleanArch+AtomicDesign、WebGL/WebGPU API活用。

  Use when: DisplayObject API、MVVM(View/VM/UseCase/Repository)、routing/config/stage設定、AtomicDesign、AnimationTool、マルチプラットフォームビルド、ButtonAtom連打防止、stopIndexタイプライター、namespaceクラス判定、Sprite中心点・アニメーション基点、Shapeキャッシュ・cacheAsBitmap最適化

  Trigger keywords: Next2D, next2d, @next2d/player, @next2d/framework, gotoView, routing.json, stage.json, config.json, scaleMode, align, ButtonAtom, 連打防止, stopIndex, タイプライター, テキストアニメーション, イベント, PointerEvent, KeyboardEvent, GamepadEvent, addEventListener, イベント定数, ゲームパッド, gamepad, コントローラー, ボタン入力, スティック, 軸入力, namespace, constructor.name, クラス判定, instanceof, minify, beginBitmapFill, ビットマップ塗りつぶし, 画像タイル, Shapeキャッシュ, graphicsキャッシュ, パスキャッシュ, cacheAsBitmap, 描画負荷, GPU負荷, 描画最適化, Sprite中心点, 0,0中心, 回転基点, スケール基点, アニメーション基点
---

# Next2D Development Assistant

## Architecture

```
View Layer (view/, ui/)
  └─ depends on ─→ Interface Layer (interface/)
                     ↑
Application Layer (model/application/)
  ├─ depends on ─→ Interface Layer
  ├─ depends on ─→ Domain Layer (model/domain/)
  └─ calls ──────→ Infrastructure Layer (model/infrastructure/)
```

**Design Patterns:** MVVM + Clean Architecture + Atomic Design
**Language:** TypeScript (any 禁止, Interface は I プレフィックス)
**Build Tool:** Vite / **Testing:** Vitest / **Package Manager:** npm

## Initial Setup

新規プロジェクトを作成する場合は以下のコマンドを実行する。**TypeScript テンプレートを推奨する：**

```bash
# 推奨: TypeScript
npx create-next2d-app {{PROJECT-NAME}} --template @next2d/framework-typescript-template

# JavaScript (非推奨)
npx create-next2d-app {{PROJECT-NAME}} --template @next2d/framework-javascript-template
```

## Core Workflow

MCP ツールが利用可能なら（next2d-development-mcp）、雛形生成は `create_view` / `add_route` / `create_usecase` / `create_repository` / `create_ui_component` 等を優先し、変更後は `validate_architecture` で検証する。

### 1. 新しい画面を追加する

1. `src/config/routing.json` にルートを追加
2. `npm run generate` で View/ViewModel の雛形を自動生成
3. ViewModel に UseCase を追加
4. View に UI コンポーネント (Atomic Design) を配置
5. イベントは必ず ViewModel に委譲

### 2. API データを取得する画面

1. `interface/` にレスポンス型を定義 (`I` プレフィックス)
2. `model/infrastructure/repository/` に Repository を作成 (try-catch 必須, config からエンドポイント取得)
3. `model/application/{screen}/usecase/` に UseCase を作成 (`execute` メソッド統一)
4. `routing.json` の `requests` で自動取得、または ViewModel から UseCase 経由で取得

### 3. Animation Tool アセットを使う

1. Animation Tool でシンボルを作成 → `.n2d` ファイルを `file/` に配置
2. `ui/content/` に MovieClipContent 継承クラスを作成 (`namespace` でシンボル名を指定)
3. `routing.json` で `type: "content"` として読み込み

### 4. TextField タイプライターアニメーション（stopIndex）

`stopIndex` で表示文字数を制御し、`Tween` でアニメーション。RPGゲームの台詞ウィンドウ演出に使用する。
`stopIndex` のデフォルトは `-1`（全文字表示）。`0` にすると文字が非表示になる。
コード例は `references/player-text-field.md` の「RPGゲーム風台詞アニメーション（stopIndex）」を参照。

### 5. 設定ファイルを変更する（src/config/）

1. **stage.json**: ホワイトリストのみ設定可能（Key Rules 参照）。**ホワイトリスト外のキー（`scaleMode`・`align` 等）は追加しない**
2. **config.json**: 環境依存設定は `local`/`dev`/`stg`/`prd` 環境ブロックへ、全環境共通は `all` へ
3. **カスタム表示挙動**（scaleMode / align 等）: `config.json` にユーザ定義設定として書いて**コード側で実装**する（フレームワークは自動処理しない）
4. 変更後は MCP `validate_architecture`（stage.json の無効キーを検出）で検証

## View/ViewModel Lifecycle

```
ViewModel 生成 → ViewModel.initialize() → View 生成 (VM注入) → View.initialize() → View.onEnter() → (操作) → View.onExit()
```

**重要:** ViewModel の `initialize()` は View の `initialize()` より前に呼ばれる。View 初期化時にはデータ準備済み。

## Key Rules

- **View:** 表示構造のみ。ビジネスロジック禁止。イベントは ViewModel に委譲
- **ViewModel:** UseCase を保持。インターフェースに依存 (具象クラス禁止)
- **UseCase:** 1 アクション = 1 UseCase。エントリーポイントは `execute`。単一責任
- **Repository:** try-catch 必須。エンドポイントは `config` から取得。`any` 禁止
- **UI Component:** 単一責任。データは ViewModel から引数で受け取る
- **Interface:** `I` プレフィックス。必要最小限のプロパティのみ
- **stage.json:** 設定できるのは `width` / `height` / `fps` / `options`（`fullScreen` / `tagId` / `bgColor`）のみ。`scaleMode`・`align`・`quality`・`wmode` 等の Flash 由来オプションは無効（Next2D は Flash Player の派生でなく、ホワイトリスト外のキーはサイレントに無視される）。画面いっぱい表示は `options.fullScreen: true`。詳細は `references/framework-specs.md` の「stage.json」を参照
- **クラス判定:** `constructor.name` は minify でクラス名が変わるため禁止。`displayObject.namespace`（インスタンス）または `Stage.namespace`（static）の文字列比較で判定する。詳細は `references/player-display-object.md` の「クラス判定（namespace）」を参照
- **動作検証:** 画面遷移や UI 挙動の変更後は、`npx playwright` によるE2E動作確認を推奨（例: `npx playwright test`）
- **CSP設定:** `default-src 'self' data: blob:` / `worker-src 'self' blob: data:` / `style-src 'self' 'unsafe-inline'` が必須。`frame-ancestors 'none'` は追加禁止

## Display Object Hierarchy

Next2D の表示オブジェクトの継承関係。型キャストや子オブジェクト管理で誤りやすいため注意：

```
DisplayObject (基底クラス)
├── InteractiveObject
│   ├── DisplayObjectContainer
│   │   └── Sprite
│   │       └── MovieClip    ← addChild() 可能、タイムラインアニメーション
│   └── TextField            ← addChild() 不可、テキスト表示/入力
├── Shape                    ← addChild() 不可、軽量ベクター描画専用
└── Video                    ← addChild() 不可、動画再生専用
```

**重要な型制約:**
- `Shape` は `DisplayObjectContainer` を継承しない → `addChild()` 不可、子オブジェクト管理不可
- `Shape` と `Sprite` は直接キャスト不可（`Conversion of type 'Shape' to type 'Sprite'` エラー）→ `as unknown as Sprite` の二段階アサーションが必要
- `hitArea` プロパティの型は `Sprite | null` → `Shape` を `hitArea` に渡す場合は型アサーション必須

## Build Commands

| Command | Platform | Output |
|---------|----------|--------|
| `npm run build:web -- --env prd` | Web | `dist/web/prd/` |
| `npm run build:steam:windows -- --env prd` | Windows | `dist/steam/windows/` |
| `npm run build:steam:macos -- --env prd` | macOS | `dist/steam/macos/` |
| `npm run build:ios -- --env prd` | iOS | Xcode project |
| `npm run build:android -- --env prd` | Android | Android Studio project |

Environment options: `--env local|dev|stg|prd`

## References

詳細仕様は `references/` ディレクトリにある。
**Load only the single file needed for the current task.**（タスクに必要なファイルだけ1つ読む）
MCP を利用する場合は `next2d://specs`（索引）を読み、URI を1つ選んで読む。

- **クイック**: 実装パターン・落とし穴 → まず「クイックリファレンス」
- **完全版**: 詳細なAPI確認が必要なときのみ「クラス別フルAPI」

### Player - クイックリファレンス（パターン・落とし穴）

- **[player-overview.md](references/player-overview.md)** - Playerの仕組み・アーキテクチャを理解したいとき
- **[player-display-objects.md](references/player-display-objects.md)** - クラス選び、型制約、キャッシュ・namespaceのポイント
- **[player-events.md](references/player-events.md)** - イベントリスナー、入力処理（マウス / キー / ゲームパッド）
- **[player-media-text.md](references/player-media-text.md)** - テキスト・音声・動画のポイント、stopIndexタイプライター
- **[player-tween.md](references/player-tween.md)** - Tween / Job / Easing アニメーション
- **[player-filters.md](references/player-filters.md)** - フィルター効果（グロー / 影 / ぼかしなど）

### Player - クラス別フルAPI

- **[player-display-object.md](references/player-display-object.md)** - DisplayObject
- **[player-sprite.md](references/player-sprite.md)** - Sprite
- **[player-movie-clip.md](references/player-movie-clip.md)** - MovieClip
- **[player-shape.md](references/player-shape.md)** - Shape
- **[player-text-field.md](references/player-text-field.md)** - TextField
- **[player-video.md](references/player-video.md)** - Video
- **[player-sound.md](references/player-sound.md)** - Sound

### Framework / 開発仕様

- **[framework-specs.md](references/framework-specs.md)** - MVVMアーキテクチャ、routing / config、ライフサイクル、Animation Tool
- **[develop-specs.md](references/develop-specs.md)** - プロジェクト構造、CLIコマンド、Interface / Model / UIパターン

