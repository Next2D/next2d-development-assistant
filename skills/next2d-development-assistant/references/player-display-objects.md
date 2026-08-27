# Next2D Player - 表示オブジェクト（クイックリファレンス）

クラス選び・型制約・キャッシュのポイント。完全なAPIは「クラス別フルAPI」ファイルを参照。

## クラス階層

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

## どのクラスを使うか

| 要件 | クラス | 備考 |
|------|--------|------|
| コンテナ / ボタン / インタラクション | `Sprite` | addChild・イベント可能 |
| タイムラインアニメーション（Animation Tool） | `MovieClip` | Spriteのサブクラス |
| ベクター描画 / 軽量な静的グラフィックス | `Shape` | addChild 不可。cacheAsBitmap で高速化 |
| テキスト表示 / 入力 | `TextField` | addChild 不可 |
| 動画再生 | `Video` | addChild 不可 |

## 重要な型制約（よく失敗する箇所）

- `Shape` は `DisplayObjectContainer` を継承しない → `addChild()` 不可、子オブジェクト管理不可
- `Shape` と `Sprite` は直接キャスト不可 → `as unknown as Sprite` の二段階アサーションが必要
- `hitArea` の型は `Sprite | null` → `Shape` を渡す場合は型アサーション必須

## キャッシュ・クラス判定のポイント

- **cacheAsBitmap**: Matrix のスケール値（a, d）のみ設定可。スケール変更でキャッシュ再生成。Video は非対応
- **クラス判定**: `constructor.name` は minify で壊れるため**禁止**。完全一致は `namespace`（`Stage.namespace` と比較が推奨）、機能判定は `isStage` / `isSprite` / `isShape` 等のフラグ
- **Shape のパスキャッシュ**: 同じパス情報の Shape はキャッシュを再利用。`alpha` / `x` / `y` / `rotation` の変更は低負荷
- **Imageの読み込み**: 単一画像は `load()`、繰り返し描画・塗りつぶしは `beginBitmapFill`（player-shape.md 参照）

## クラス別フルAPI

| クラス | ファイル |
|--------|----------|
| DisplayObject（cacheAsBitmap・namespace・ブレンドモード） | [player-display-object.md](references/player-display-object.md) |
| Sprite（マスク・ドラッグ） | [player-sprite.md](references/player-sprite.md) |
| MovieClip（フレーム制御・フレームラベル） | [player-movie-clip.md](references/player-movie-clip.md) |
| Shape（Graphics・Image読み込み・パスキャッシュ） | [player-shape.md](references/player-shape.md) |
