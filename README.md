# VisualViewportAPI playground

VisualViewport API を使って、オーバーレイ UI（FAB と debug パネル）を viewport に追従させる検証用プロジェクトです。

## 開発

```sh
npm install
npm run dev
```

## 現在の拡大縮小仕様（単純化後）

このプロジェクトは **`VisualViewport.scale` のみ** を拡大縮小管理に使用します。

- `devicePixelRatio` は拡大縮小補正に使用しません。
- PC ブラウザのズームでは、UI の見え方が従来どおり不変になることは保証しません。
- スマホのピンチイン/アウトでは、`scale` に対する逆スケールにより、UI の見かけサイズ・配置の安定を狙います。

## 動作確認チェックリスト

1. **PCブラウザズーム**
   - ブラウザズーム時に、FAB と debug パネルが以前の「完全不変」挙動ではなくても問題ないこと。

2. **モバイルのピンチイン/アウト**
   - ピンチ操作時に、FAB と debug パネルの見かけサイズ・配置が大きく崩れないこと。

3. **VisualViewport 非対応環境**
   - フォールバック（`scale=1`, `innerWidth/innerHeight`）で表示が破綻しないこと。

確認対象は `src/routes/+page.svelte` の以下です。

- `--vv-scale-inv` を使う transform（`.fab-slot`, `.vv-debug`）
- viewport metrics の追従（`width`, `height`, `offsetLeft`, `offsetTop`）

## ビルド

```sh
npm run build
npm run preview
```
