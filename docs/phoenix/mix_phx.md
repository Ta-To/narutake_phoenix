アプリケーション作成等に使用する mix phx.* についての整理です

## Mix.Tasks

- [公式(hexdocs)](https://hexdocs.pm/phoenix/Mix.Tasks.Phx.New.html)

## 徒然メモ

---

Q. アプリケーション作成から確認方法

https://hexdocs.pm/phoenix/up_and_running.html

```
mix phx.new --live --no-dashboard my_app
cd my_app
mix ecto.create
mix phx.server
```

---

Q. 現フォルダ直下に作成したい

`--app` でアプリ名を指定する

- `mix phx.new --live --no-dashboard --app my_app ./`

---
