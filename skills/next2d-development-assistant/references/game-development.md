# Next2D ゲーム開発と LocalLLM 分担

テトリスやブロック崩しなど、Next2Dでリアルタイムゲームを実装・修正・検証するときに読む。

## 実装の分離

1. `model/domain/` に Next2D非依存のゲームエンジンを置く。状態取得はボール・配列要素まで防御コピーする。
2. ゲームエンジンは表示座標、時間単位（`elapsedMs` など）、乱数の注入可否を公開APIで明示する。得点、ライフ、勝敗、resetをVitestで先に固定する。
3. ViewModelはゲーム操作と更新時刻を管理し、Viewは描画と入力の購読・解除だけを担う。`Event.ENTER_FRAME` は `onEnter` で登録し `onExit` で解除する。Player は非バブリングのイベントをグローバル `stage` に dispatch するため、ゲームループは `stage.addEventListener(Event.ENTER_FRAME, listener)` を使う（View自身には登録しない）。
4. 画面は `Shape` で盤面・ゲーム要素を描き、HUDは別コンポーネントにする。ゲーム側が外部APIを必要としないなら、対象routeの`requests`を空配列にする。

## LocalLLM への委譲

- 依頼は **1ファイル・1目的** に分割する。例: 型定義、ドメイン実装、テスト、UI、検証を別依頼にする。
- 指示には対象パス、変更してよいファイル、公開API、完了条件を含める。一括の「実装→テスト→ビルド→修正」は避ける。
- 修正前に対象ファイルを読み、変更後は差分をレビューする。APIを変えたら、同じ変更単位でテストも追従させる。
- shellツールへ渡す数値引数（例: `timeout`）は文字列にしない。ツール引数エラー後は、同じ大きな依頼を繰り返さず小さく分ける。
- LocalLLMは規模にかかわらず数時間かかることがある。**経過時間やファイル更新の有無だけでは中断しない**。完了・明確な失敗・ユーザーの停止指示まで待機し、待機中は定期的に状態だけ共有する。モデルの応答待ちを「実装済み」と扱わない。
- 工程完了はファイルの存在だけで判定しない。対象API・イベント登録/解除・設定値と、型検査/Vitestの成功を確認する。テンプレートに同名ファイルがある場合は特に注意する。
- 長時間のLocalLLM工程を別プロセスで実行する場合は、`running` / `review_required` / `failed` / `complete`、PID、ログを状態ファイルへ残す。非ゼロ終了でも成果物と検証が成功していれば、Codexレビュー後に次工程へ進める。

## 検証の順序

1. 変更前に `npx tsc --noEmit --skipLibCheck` と `npm test -- --run` を実行し、既存エラーを記録する。
2. ドメインテストを単独で通してから、プロジェクト全体の型検査・Webビルドを実行する。
3. Playwrightでは canvas の有無、console/page error、入力後の状態を確認する。ユーザーの既存Chromeへ接続せず、Playwrightが起動した専用ブラウザだけを使い、必ず `browser.close()` で閉じる。Vite proxy が既定ポートを参照する場合は、その既定ポートで起動してから検証する。headless Chromiumで `WebGPU adapter not available` が出てcanvasが1×1のままなら、ゲームの失敗とは断定せずWebGPU対応ブラウザで再確認する。
4. ゲームループは初期画面と入力直後だけでなく、発射・移動後の少し遅い時点でもスクリーンショットまたはcanvas画素差を取り、描画が変化したことを確認する。
5. `validate_game_integration` が使える場合は、E2E前に盤面定数とstage寸法、グローバル`stage`での`ENTER_FRAME`登録・解除を検証する。

## テンプレートの前提確認

実装前に `src/config/Config.ts`、`src/Packages.ts`、Vite alias、`stage.json` を確認する。テンプレートの欠損による型検査失敗はゲーム変更と分離して直し、ゲームの失敗として扱わない。
