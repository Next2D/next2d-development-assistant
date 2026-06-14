# Next2D Player - イベントシステム

Next2D Playerは、W3C DOMイベントモデルと同様の3フェーズイベントフロー機構を採用しています。

---

# EventDispatcher

すべてのイベント発行可能なオブジェクトの基底クラスです。

## addEventListener(type, listener, useCapture, priority)

イベントリスナーを登録します。

```typescript
const { PointerEvent } = next2d.events;

displayObject.addEventListener(PointerEvent.POINTER_DOWN, (event) => {
    console.log("ポインターが押されました");
});

// キャプチャフェーズで受け取る
displayObject.addEventListener(PointerEvent.POINTER_DOWN, handler, true);

// 優先度を指定
displayObject.addEventListener(PointerEvent.POINTER_DOWN, handler, false, 10);
```

## removeEventListener(type, listener, useCapture)

```typescript
displayObject.removeEventListener(PointerEvent.POINTER_DOWN, handler);
```

## removeAllEventListener(type, useCapture)

```typescript
displayObject.removeAllEventListener(PointerEvent.POINTER_DOWN);
```

## hasEventListener(type) / willTrigger(type)

```typescript
if (displayObject.hasEventListener(PointerEvent.POINTER_DOWN)) { /* ... */ }
if (displayObject.willTrigger(PointerEvent.POINTER_DOWN)) { /* ... */ }
```

## dispatchEvent(event)

```typescript
const { Event } = next2d.events;
const event = new Event("customEvent");
displayObject.dispatchEvent(event);
```

---

# Event クラス

## プロパティ

| プロパティ | 型 | 説明 |
|-----------|------|------|
| `type` | String | イベントタイプ |
| `target` | Object | イベント発行元 |
| `currentTarget` | Object | 現在のリスナー登録先 |
| `eventPhase` | Number | イベントフェーズ |
| `bubbles` | Boolean | バブリングするか |

## メソッド

| メソッド | 説明 |
|----------|------|
| `stopPropagation()` | 伝播を停止 |
| `stopImmediatePropagation()` | 伝播を即座に停止 |

---

# イベント定数の使用

イベントを `addEventListener` で登録する際は、文字列リテラルではなく必ず定数を使用する。

```typescript
// NG: 文字列リテラル
sprite.addEventListener("pointerup", handler);

// OK: 定数を使用
sprite.addEventListener(PointerEvent.POINTER_UP, handler);
```

**理由:** 定数を使用することで、タイプミスの防止、IDEによる補完・型チェック、リファクタリング時の安全性が得られる。

## 全イベント定数一覧

| クラス | 定数 | 文字列値 | 説明 |
|--------|------|----------|------|
| `PointerEvent` | `POINTER_DOWN` | `"pointerdown"` | ポインター押下 |
| `PointerEvent` | `POINTER_UP` | `"pointerup"` | ポインター解放 |
| `PointerEvent` | `POINTER_MOVE` | `"pointermove"` | ポインター移動 |
| `PointerEvent` | `POINTER_OVER` | `"pointerover"` | ポインター進入 |
| `PointerEvent` | `POINTER_OUT` | `"pointerout"` | ポインター退出 |
| `PointerEvent` | `POINTER_LEAVE` | `"pointerleave"` | ポインター離脱 |
| `PointerEvent` | `POINTER_CANCEL` | `"pointercancel"` | ポインターキャンセル |
| `PointerEvent` | `DOUBLE_CLICK` | `"dblclick"` | ダブルクリック |
| `KeyboardEvent` | `KEY_DOWN` | `"keydown"` | キー押下 |
| `KeyboardEvent` | `KEY_UP` | `"keyup"` | キー解放 |
| `FocusEvent` | `FOCUS_IN` | `"focusin"` | フォーカス取得 |
| `FocusEvent` | `FOCUS_OUT` | `"focusout"` | フォーカス喪失 |
| `WheelEvent` | `WHEEL` | `"wheel"` | ホイール操作 |
| `VideoEvent` | `PLAY` | `"play"` | 再生リクエスト |
| `VideoEvent` | `PLAYING` | `"playing"` | 再生開始 |
| `VideoEvent` | `PAUSE` | `"pause"` | 一時停止 |
| `VideoEvent` | `SEEK` | `"seek"` | シーク操作 |
| `JobEvent` | `UPDATE` | `"update"` | プロパティ更新 |
| `JobEvent` | `STOP` | `"stop"` | ジョブ停止 |
| `Event` | `COMPLETE` | `"complete"` | 処理完了（Tween/Loader） |
| `Event` | `ENTER_FRAME` | `"enterframe"` | 毎フレーム |
| `Event` | `ADDED` | `"added"` | コンテナに追加 |
| `Event` | `ADDED_TO_STAGE` | `"addedtostage"` | Stageに追加 |
| `Event` | `REMOVED` | `"removed"` | コンテナから削除 |
| `Event` | `REMOVED_FROM_STAGE` | `"removedfromstage"` | Stageから削除 |
| `Event` | `IO_ERROR` | `"ioerror"` | IOエラー |
| `Event` | `HTTP_STATUS` | `"httpstatus"` | HTTPステータス受信 |

## 使用例

```typescript
import { PointerEvent, KeyboardEvent, Event } from "@next2d/events";

// ポインターイベント
button.addEventListener(PointerEvent.POINTER_DOWN, onPointerDown);
button.addEventListener(PointerEvent.POINTER_UP,   onPointerUp);

// キーボードイベント
stage.addEventListener(KeyboardEvent.KEY_DOWN, onKeyDown);

// タイムラインイベント
movieClip.addEventListener(Event.ENTER_FRAME, onEnterFrame);
```

## 定数がないイベント

一部のイベント（表示リスト、タイムライン、ロード関連など）は `Event` クラスの文字列リテラルを使用する。

```typescript
import { Event } from "@next2d/events";

sprite.addEventListener(Event.ADDED_TO_STAGE, onAdded);
movieClip.addEventListener(Event.ENTER_FRAME, onFrame);
```

---

# 標準イベントタイプ

## 表示リスト関連

| イベント | 説明 |
|----------|------|
| `added` | DisplayObjectContainerに追加された |
| `addedToStage` | Stageに追加された |
| `removed` | DisplayObjectContainerから削除された |
| `removedFromStage` | Stageから削除された |

## タイムライン関連

| イベント | 説明 |
|----------|------|
| `enterFrame` | 各フレームで発生 |
| `frameConstructed` | フレーム構築完了 |
| `exitFrame` | フレーム離脱時 |

## ロード関連

| イベント | 説明 |
|----------|------|
| `complete` | ロード完了 |
| `progress` | ロード進捗 |
| `ioError` | IOエラー |
| `httpStatus` | HTTPステータス受信 |

---

# ポインターイベント

PointerEventはマウス、ペン、タッチなどのポインターデバイスの操作を統一的に処理します。

| イベント | 定数 | 説明 |
|----------|------|------|
| `pointerDown` | `PointerEvent.POINTER_DOWN` | ボタンの押下開始 |
| `pointerUp` | `PointerEvent.POINTER_UP` | ボタンの解放 |
| `pointerMove` | `PointerEvent.POINTER_MOVE` | ポインター座標の変化 |
| `pointerOver` | `PointerEvent.POINTER_OVER` | ポインターがヒットテスト境界に入った |
| `pointerOut` | `PointerEvent.POINTER_OUT` | ポインターがヒットテスト境界を出た |
| `pointerLeave` | `PointerEvent.POINTER_LEAVE` | ポインターが要素領域を離れた |
| `pointerCancel` | `PointerEvent.POINTER_CANCEL` | ポインター操作がキャンセルされた |
| `doubleClick` | `PointerEvent.DOUBLE_CLICK` | ダブルクリック/タップが発生 |

```typescript
const { PointerEvent } = next2d.events;

sprite.addEventListener(PointerEvent.POINTER_DOWN, (event) => {
    console.log("ポインターダウン:", event.localX, event.localY);
});

sprite.addEventListener(PointerEvent.POINTER_MOVE, (event) => {
    console.log("ポインター移動:", event.stageX, event.stageY);
});
```

---

# キーボードイベント

| イベント | 定数 | 説明 |
|----------|------|------|
| `keyDown` | `KeyboardEvent.KEY_DOWN` | キー押下 |
| `keyUp` | `KeyboardEvent.KEY_UP` | キー解放 |

```typescript
const { KeyboardEvent } = next2d.events;

stage.addEventListener(KeyboardEvent.KEY_DOWN, (event) => {
    console.log("キーコード:", event.keyCode);
    switch (event.keyCode) {
        case 37: player.x -= 10; break;  // 左矢印
        case 39: player.x += 10; break;  // 右矢印
    }
});
```

---

# フォーカスイベント

| イベント | 定数 | 説明 |
|----------|------|------|
| `focusIn` | `FocusEvent.FOCUS_IN` | フォーカスを受け取った |
| `focusOut` | `FocusEvent.FOCUS_OUT` | フォーカスを失った |

---

# ホイールイベント

| イベント | 定数 | 説明 |
|----------|------|------|
| `wheel` | `WheelEvent.WHEEL` | マウスホイールが回転した |

---

# ビデオイベント

| イベント | 定数 | 説明 |
|----------|------|------|
| `play` | `VideoEvent.PLAY` | 再生がリクエストされた |
| `playing` | `VideoEvent.PLAYING` | 再生が開始された |
| `pause` | `VideoEvent.PAUSE` | 一時停止された |
| `seek` | `VideoEvent.SEEK` | シーク操作 |

---

# ジョブイベント

Tweenアニメーション用のイベントです。

| イベント | 定数 | 説明 |
|----------|------|------|
| `update` | `JobEvent.UPDATE` | プロパティが更新された |
| `stop` | `JobEvent.STOP` | ジョブが停止した |

---

# カスタムイベント

```typescript
const { Event } = next2d.events;

// カスタムイベントの定義
const customEvent = new Event("gameOver", true, true);

// イベントの発行
gameManager.dispatchEvent(customEvent);

// イベントのリッスン
gameManager.addEventListener("gameOver", (event) => {
    showGameOverScreen();
});
```

---

# イベントの伝播

イベントは3つのフェーズで伝播します：

1. **キャプチャフェーズ**: rootからtargetへ（eventPhase = 1）
2. **ターゲットフェーズ**: targetで処理（eventPhase = 2）
3. **バブリングフェーズ**: targetからrootへ（eventPhase = 3）

```typescript
const { PointerEvent } = next2d.events;

// キャプチャフェーズで処理
parent.addEventListener(PointerEvent.POINTER_DOWN, handler, true);

// バブリングフェーズで処理（デフォルト）
child.addEventListener(PointerEvent.POINTER_DOWN, handler, false);
```
