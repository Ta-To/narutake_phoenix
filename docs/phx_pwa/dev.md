Phoenix で Progressive Web Application (PWA) 実装に関する記録

※ 具体的なファイル内容は [リポジトリ](TODO) を参照

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

- `assets/static/js/service-worker.js` を作成する
- app.js 等で上記ファイルをインポートするように処理を記載する
  - register-service-worker を利用して `regiser(service-worker.jsのパス)`
- ブラウザのデバッグツールから「アプリケーション > ServiceWorkers」などで確認する

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
    only: ~w(css fonts images js favicon.ico robots.txt manifest.webmanifest)
```

- ブラウザのデバッグツールから「アプリケーション > マニフェスト」などで確認する
- in private な状態ではアプリケーションの登録はできない様子
