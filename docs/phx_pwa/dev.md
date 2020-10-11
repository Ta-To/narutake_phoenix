Phoenix で Progressive Web Application (PWA) 実装に関する記録

※ 具体的なファイル内容は [リポジトリ](https://github.com/Ta-To/phx_pwa) を参照

## PWA化の利点

- プッシュ通知の実現
- ホームへのインストールの実現
- オフライン状態での機能提供の実現
  - LiveView を使う場合にはうま味は減るが、限定的な用途での機能提供等であれば十分可能
- キャッシュを生かしたアプリケーション


## (1) ~ ホームへのインストール の実現

なにをともあれ service-worker.js を用意する必要がある

### register-service-worker パッケージインストール

なにこれ: ServiceWorkerへの登録と各イベント処理の利便性向上のため導入

```
cd assets
npm install register-service-worker
```

### service-workder.js の作成とその読み込み

- `assets/static/service-worker.js` を作成する
  - `service-worker.js` が置かれているフォルダがスコープを意味するので注意すること
  - `/js/service-worker.js` 等としてはいけない
- app.js 等で上記ファイルをインポートするように処理を記載する
  - register-service-worker を利用して `regiser(service-worker.jsのパス)`
- ブラウザのデバッグツールから「アプリケーション > ServiceWorkers」などで確認する

### service-worker.js へのアクセス許可

デフォルトでは静的コンテンツとして扱われないため、endpoint.ex に追加する

``` elixir:endpoint.ex
  plug Plug.Static,
    at: "/",
    from: :phx_pwa,
    gzip: false,
    only: ~w(css fonts images js favicon.ico robots.txt service-worker.js)
```

### manifest ファイルの作成とその読み込み

- `assets/static/manifest.webmanifest` を作成する
- headタグ内 `rel="manifest"` として読み込む

### manifest 拡張子へのアクセス許可

デフォルトでは .manifest は静的コンテンツとして扱われないため、endpoint.ex に追加する

``` elixir:endpoint.ex
  plug Plug.Static,
    at: "/",
    from: :phx_pwa,
    gzip: false,
    only: ~w(css fonts images js favicon.ico robots.txt service-worker.js manifest.webmanifest)
```

- ブラウザのデバッグツールから「アプリケーション > マニフェスト」などで確認する
- in private な状態ではアプリケーションの登録はできない様子

### 「端末にインストール」ボタンの設置

- ブラウザのメニューから操作して追加するのは一般的には敷居が高い
- また、デフォルト=いきなりポップアップが表示される、と意味が分からない人を驚かす。意味が分かっている人もいらだつ(?)
- 実装は[こちら](https://developer.mozilla.org/ja/docs/Web/Progressive_web_apps/Add_to_home_screen)


## (2) ~ プッシュ通知の実現

- プッシュサーバとの通信時の暗号方式などのプロトコルがあるため、pigeonライブラリを使用する
- Firebase は使わない方針で試す (VAPID)

// とりあえずFCMのみ対応


## (3) ~ バックグラウンド処理対応



## 徒然メモ

---

Q. beforeinstallprompt イベントが発火しない

同じ場所で load イベントは発火した
|> ServiceWorkerに最低限でも実装が必要とのこと

`self.addEventListener("fetch", function(event) {});`

参照記事

- https://stackoverflow.com/questions/44886248/add-to-home-screen-web-app-install-banner-not-showing-up-in-my-web-app-and-show
- https://amymd.hatenablog.com/entry/2017/10/12/001612
- https://qiita.com/ProjectEuropa/items/467ef24d0708b76f685d

- また `service-worker.js` がルートフォルダ(/)においてあることを確認する

---
