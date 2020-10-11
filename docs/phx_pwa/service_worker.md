Service Worker に関するノート

## service-worker.js

Service Workerに対する制御を記載するJSファイル

- navigator.serviceWorker に読み込ませる
- service-worker.js を置いているパス以下がそのコントロール対象(スコープ)となる
- sw.js 等とも訳されている様子(同じもの)
- `https` あるいは `localhost` でしかPWAに関する機能は動作しないため注意

## workbox

TODO


## 動作確認

- [Chrome DevTools Progressive Web App のデバッグ](https://developers.google.com/web/tools/chrome-devtools/progressive-web-apps?hl=ja)


## 資料など

- [プログレッシブウェブアプリ | MDN](https://developer.mozilla.org/ja/docs/Web/Progressive_web_apps)
- [ServiceWorker Cookbook](https://serviceworke.rs/)
- [Get Started | Workbox](https://developers.google.com/web/tools/workbox/guides/get-started)


## 徒然メモ

---

Q. Service Worker とは？

ブラウザによってWebページとは別にバックグラウンドで処理されるスクリプト

- ネットワークリクエストへ介在 (プロキシ)
  - キャッシュの使用や制御
- プッシュ通知の制御
- 直接的なDOM操作不可

---

Q. Firefox のデバッグツールで ServiceWorker がないとでる

(開発途中のこと。) Edge だと表示された。特に記載のないservice-workerな状態だとそうなる？

---