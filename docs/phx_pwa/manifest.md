マニフェストファイル に関するノート

## manifest.webmanifest

アプリの設定(アイコンなど)を記載したJSON形式のファイル

- `<link rel="manifest" href="manifest.webmanifest">` 等として読み込ませる
- `service-worker.js` には特に記載の必要なし (=直接は関係ない)
  - ChromeはService Workerを前提としているらしい (未確認)
- manifest.json と同義


## 資料など

- [MDN web docs ホーム画面に追加](https://developer.mozilla.org/ja/docs/Web/Progressive_web_apps/Add_to_home_screen)
- [プログレッシブ Web アプリ (Chromium) の使用を開始する](https://docs.microsoft.com/ja-jp/microsoft-edge/progressive-web-apps-chromium/get-started)


## 徒然メモ

---

Q. さくっとアイコン画像を用意するには？

- [doc.microsoft.com サンプルicon](https://docs.microsoft.com/ja-jp/microsoft-edge/progressive-web-apps-chromium/media/pwa.png)

---

Q. safari でうまくいかないとき

https://github.com/GoogleChromeLabs/pwacompat

---

## サンプル

``` json:manifest.webmanifest
{
  "lang": "ja-JP",
  "name": "My Sample PWA",
  "short_name": "SamplePWA",
  "description": "A sample PWA for practice",
  "start_url": "/",
  "background_color": "#2f3d58",
  "theme_color": "#2f3d58",
  "orientation": "any",
  "display": "standalone",
  "icons": [
    {
      "src": "/images/icon512.png",
      "sizes": "512x512"
    }
  ],
  "permissions": ["background"],
}
```
