# Next2D Player - メディア・テキスト（クイックリファレンス）

テキスト・音声・動画のAPI選びと主要ポイント。完全なAPIはクラス別フルAPIファイルを参照。

## どのAPIを使うか

| 要件 | API | フルAPI |
|------|-----|---------|
| テキスト表示 / 入力 | `TextField` / `TextFormat` | [player-text-field.md](references/player-text-field.md) |
| タイプライター演出 | `TextField.stopIndex` + `Tween` | player-text-field.md の「RPGゲーム風台詞アニメーション」 |
| BGM / 効果音 | `Sound` / `SoundMixer` | [player-sound.md](references/player-sound.md) |
| 動画再生 | `Video` / `VideoEvent` | [player-video.md](references/player-video.md) |

## 主要ポイント

- **stopIndex タイプライター**: デフォルト `-1`（全文字表示）、`0` で全非表示。`Tween.add(textField, { stopIndex: 0 }, { stopIndex: text.length }, 遅延, 時間)` で animate
- **効果音を同時に複数回鳴らす**: `sound.clone()` してから再生
- **BGM ループ**: `loopCount = 9999`（実質無限ループ。0 でループなし）
- **Video**: `VideoEvent.PLAY` / `PLAYING` / `PAUSE` / `SEEK` で再生状態を管理。addChild 不可
