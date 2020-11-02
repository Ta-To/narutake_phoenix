Phoenix で Progressive Web Application (PWA) 実装に関する記録

※ 具体的なファイル内容は [リポジトリ](https://github.com/Ta-To/phx_pwa) を参照

## PWA化の利点

- プッシュ通知の実現
- ホームへのインストールの実現
- オフライン状態での機能提供の実現
  - LiveView を使う場合にはうま味は減るが、限定的な用途での機能提供等であれば十分可能
- キャッシュを生かしたアプリケーション


## (1) ~ ホームへのインストール の実現

まず service-worker.js を用意する必要がある

### register-service-worker パッケージインストール

なにこれ: ServiceWorkerへの登録と各イベント処理の利便性向上のために導入

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
- 実装参考例は[こちら](https://developer.mozilla.org/ja/docs/Web/Progressive_web_apps/Add_to_home_screen)


## (2) ~ プッシュ通知の実現

- Firebase のサービスは介さない方針で試す (VAPID)
- まとまった資料は[こちら](https://ta-to.github.io/narutake_phoenix/#/phx_pwa/push)

### 通知に対する許可を得る

- 通知は、ユーザに許可してもらう必要がある

### ブラウザから購読情報(配信先)をサーバに知らせる

- 配信先として登録するために必要な情報をつくる
  - endpoint: サーバが配信に使用するURL。サーバはここ(プッシュサーバ)に対して情報を送る
  - keys: クライアントの識別情報一式
- ブラウザはこれらの情報をServiceWorkerを介して取得する
- その際にはどのサービスからの配信を受け取りたいかを明確にする必要がある。そのため、サービス側の公開鍵が必要になる
  - 後述の通り、配信を実際に受け取るのはServiceWorkerのため
  - 公開鍵は先に仕込んでおくか、APIで取得したりする

### サーバからの配信を受け取る＆プッシュ通知する

- 配信されると ServiceWorker のイベントとして発火する
- ServiceWorkerに取得時の処理を記述する必要がある


## (3) ~ バックグラウンド処理対応

サンプルとしてポモドーロタイマーを構成する

- 25分後に一回通知する
- 5分後に再度通知する

### 配信元の情報を保持するプロセス(Agent)をつくる

- 配信時刻になった際にプッシュ処理を行う
- 配信後にプロセスを破棄する

### 配信処理をトリガーするためのプロセス(Agent)をつくる

- 定期的に、配信時刻になったプロセスがないかどうかを確認する


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

Q. クライアント側でサーバに定義済みの環境変数を参照するには？

必要な環境変数をクライアントコード上で使用する
// PWA は無関係

- EnvironmentPlugin を使用する
- 使用する環境変数を明示する（とともにデフォルトを設定する）
- 明示した環境変数が定義済みであればサーバの環境変数が使用される

``` js:webpack.config.js 抜粋
const webpack = require('webpack');

    plugins: [
      new MiniCssExtractPlugin({ filename: '../css/app.css' }),
      new CopyWebpackPlugin([{ from: 'static/', to: '../' }]),
      new webpack.EnvironmentPlugin({
        NODE_ENV: 'development',
        PWA_PUBLiC_KEY: 'PUBLIC_KEY',
      }),
    ]
```

- [EnvironmentPluginで環境変数をコード内に渡す](https://nju33.com/webpack/EnvironmentPlugin%20%E3%81%A7%E7%92%B0%E5%A2%83%E5%A4%89%E6%95%B0%E3%82%92%E3%82%B3%E3%83%BC%E3%83%89%E5%86%85%E3%81%AB%E6%B8%A1%E3%81%99)

---

Q. async 使用箇所で `regeneratorRuntime is not defined` エラー

// PWA は無関係

``` json: assets/.babelrc
{
  "presets": [
    [
      "@babel/preset-env", {
        "targets": {
          "node": "current"
        }
      }
    ]
  ]
}
```

---

Q. Javascriptで現在時刻を取得する

`new Date(Date.now())`

- 時刻操作は `time.setMinutes(time.getMinutes()+ 25)` 等とできる

---
