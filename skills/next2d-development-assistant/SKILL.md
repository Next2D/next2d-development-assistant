---
name: next2d-development-assistant
description: >
  Next2D Player/Framework 開発支援。MVVM+CleanArch+AtomicDesign、WebGL/WebGPU API活用。

  Use when: DisplayObject API、MVVM(View/VM/UseCase/Repository)、routing/config/stage、AtomicDesign、AnimationTool、マルチプラットフォームビルド、ButtonAtom連打防止、stopIndexタイプライター、イベント定数

  Trigger keywords: Next2D, next2d, @next2d/player, @next2d/framework, gotoView, routing.json, stage.json, ButtonAtom, 連打防止, stopIndex, タイプライター, テキストアニメーション, イベント, PointerEvent, KeyboardEvent, addEventListener, イベント定数
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
コード例は `references/player-specs.md` を参照。

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

Detailed specifications are available in the `references/` directory. Read the relevant file based on the user's needs:

- **[player-specs.md](references/player-specs.md)** - Next2D Player API reference (DisplayObject, MovieClip, Sprite, Shape, TextField, Video, Sound, Tween, Events, Filters). Read when implementing rendering, animation, graphics, or interaction logic.
- **[framework-specs.md](references/framework-specs.md)** - Next2D Framework reference (MVVM architecture, routing, config, View/ViewModel lifecycle, Animation Tool integration). Read when working on application architecture, screen transitions, or configuration.
- **[develop-specs.md](references/develop-specs.md)** - Development template specs (project structure, CLI commands, interfaces, Model layer, UI layer with Atomic Design, View/ViewModel patterns). Read when creating new components, setting up projects, or following coding patterns.

