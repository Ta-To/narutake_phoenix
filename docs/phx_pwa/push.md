プッシュ通知 に関するノート

## プッシュ通知の流れ

**クライアントサイド**

ブラウザでの通知許可
|> プッシュサーバへ通知の購読を登録
|> アプリケーションサーバへその情報を送信

**アプリケーションサーバ**

クライアントから受け取った情報
|> プッシュサーバへ通知内容を送信

**留意点**

- プッシュサーバはブラウザによる
  - ブラウザは情報を取得する方法を知っている
  - アプリケーションサーバはその情報を受け取って送信する
  - よって、プッシュサーバがどうとかは実装上で(実質)意識する必要がない
- クライアントは購読する際にプッシュサーバへアプリケーションサーバの公開鍵を送る必要がある
  - 公開鍵は、取得用のAPIか、あるいは定数で仕込んでおく必要がある
- プッシュ通知はServiceWorkerが受け取るため、その制御を実装する必要がある


## ２つの方法

1. Firebase(FCM) を使用する

Firebaseに登録してAPI_KEYをもらってうんぬん

2. VAPID を使用する

[Web Pushのサーバ認証VAPIDを試してみる](https://qiita.com/tomoyukilabs/items/9346eb44b5a48b294762)

[Voluntary Application Server Identification (VAPID) for Web Push](https://tools.ietf.org/html/rfc8292)

// なるべくサービスに閉じたいため、以下はVAPIDを想定


## 公開鍵の作成

opensslで表すと下記のような準備が必要

```
openssl ecparam -genkey -name prime256v1 -out private_key.pem
openssl ec -in private_key.pem -pubout -outform DER|tail -c 65|base64|tr -d '=' |tr '/+' '_-' > public_key.txt
openssl ec -in private_key.pem -outform DER|tail -c +8|head -c 32|base64|tr -d '=' |tr '/+' '_-' > private_key.txt
```

- Elliptic Curve Digital Signature Algorithm (ECDSA) over the P-256 curve で鍵ペア作成
- それぞれURLセーフなBase64エンコードを実施したものを使う
- 感謝: [VAPIDでWebPushを実装してみた](https://qiita.com/renoinn/items/6c88c9030130e2a83648)


**Phoenix ライブラリ**

[elixir-web-push-encryption](https://github.com/danhper/elixir-web-push-encryption)

- 鍵ペア作成から送信まではこのライブラリでいけそう
- README.md では `gcm_api_key` が登場していてVAPIDのようではないが、ソースをみるとオプション扱い
- そのほかの関連ライブラリメモ
  - [pigeon](https://github.com/codedge-llc/pigeon)

```
mix web_push.gen.keypair
```

- config.exs に設定する情報は環境変数で管理を行うこと


## 購読とNotification (通知) 許可

- 購読とは、ServiceWorkerにサーバ(配信元)情報を渡して設定すること
  - またその設定情報もサーバに渡す必要がある(=配信先を配信元に教えるための連絡)
  - // ゆえにhttps対応が必須になっているのかな
- 通知を行うにはユーザの許可が必要になる (勝手に開始することはできない)
- 許可は通知が必要になったタイミングで求めること(UX)
  - // 特に明確なタイミングがなくても「購読」ボタン等を用意して押してもらうような対応が事前
- ServiceWorker への subscribe 登録前が適当 (許可と同時にsubscribeしておくとよい様子)

**購読**

すでに購読済み => 何もしない
まだ購読していない => プッシュ通知の許可を求めて、O.K.ならば購読処理してサーバに情報を渡す

``` js
let pushSubscription = PushManager.subscribe()
if(!pushSubscription) {
  const permission = await Notification.requestPermission();
  if (permission === 'denied') {
    return alert('処理を中止します。この処理の実行には通知を許可していただく必要があります');
  } else {
    pushSubscription = await PushManager.subscribe(<配信元情報等>)
    // サーバへ設定情報を連絡
    // registerToServer(pushSbuscription)
  }
}
```

**サーバの公開鍵について**

subscribe() で指定する公開鍵は、楕円曲線 DSA P-256 の Base64 でエンコードされた DOMString または ArrayBuffer である必要がある

- [PushManager.subscribe()](https://developer.mozilla.org/ja/docs/Web/API/PushManager/subscribe)


## サーバでの配送処理

ブラウザから受け取った宛先に対して通知内容を送信する

**Phoenix ライブラリ**

[elixir-web-push-encryption](https://github.com/danhper/elixir-web-push-encryption)

- 設定するキーがアトムであることに注意する。params は文字列がキーなのでそのまま送られた情報は使えない


## ServiceWorkerの受信処理

`push`イベントを受け取り、`showNotification()`等で実際の通知処理などを行う

- 通知タイトルや内容はサーバから送られてきたままでなく、最終的な決定はServiceWorker/ブラウザ側で行う（える）

``` js:service-worker.js
self.addEventListener('push', (e) => {
  e.waitUntil(self.registration.showNotification(<title>, <options>));
})
self.addEventListener('notificationclick', (e) => {
  e.notification.close();
  e.waitUntil(clients.openWindow(e.notification.data.link_to));
})
```

## 資料など

- [Qiita Firebase Cloud Messaging(FCM)を利用したWebPush通知の実装について](https://qiita.com/megadreams14/items/2f4221c3cc2a39663d7d)
- [Qiita Web Pushのサーバ認証VAPIDを試してみる](https://qiita.com/tomoyukilabs/items/9346eb44b5a48b294762)
- [DRYな備忘録 webブラウザにPush通知送るサーバとjsのサンプル](https://otiai10.hatenablog.com/entry/2017/06/19/200715)


## 徒然メモ

---

