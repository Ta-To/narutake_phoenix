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


**Phoenix**

[elixir-web-push-encryption](https://github.com/danhper/elixir-web-push-encryption)

```
mix web_push.gen.keypair
```

- README.md では `gcm_api_key` が登場していてVAPIDのようではないが、ソースをみるとオプション扱い
- 鍵ペア作成から送信まではこのライブラリでいけそう
- そのほかの関連ライブラリメモ
  - [pigeon](https://github.com/codedge-llc/pigeon)


## Notification (通知) 許可

- 通知が必要になったタイミングで初めて許可を求めること(UX)


## 資料など

- [DRYな備忘録 webブラウザにPush通知送るサーバとjsのサンプル](https://otiai10.hatenablog.com/entry/2017/06/19/200715)
- [Qiita Firebase Cloud Messaging(FCM)を利用したWebPush通知の実装について](https://qiita.com/megadreams14/items/2f4221c3cc2a39663d7d)
- [Qiita Web Pushのサーバ認証VAPIDを試してみる](https://qiita.com/tomoyukilabs/items/9346eb44b5a48b294762)


## 徒然メモ

---
