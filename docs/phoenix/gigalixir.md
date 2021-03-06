サービス運用が可能なPasSであるGigalixirに関するノート

## Gigalixir

公式サイト: https://gigalixir.com/index.html

- 1 instance + 1 database
- Never sleeps
- Custom domains
- Free TLS certificates
- Zero-downtime deploys

(2020-09-22 for free より抜粋)


## デプロイまでの手順

**アプリケーション作成**

Gigalixirでアプリケーション(デプロイ先の意)を作成する

**バージョン指定ファイルの作成**

```
echo "elixir_version=1.10.4" > ./elixir_buildpack.config
echo "erlang_version=22.3.4.5" >> ./elixir_buildpack.config
echo "node_version=12.18.3" > ./phoenix_static_buildpack.config
```

**CLIのインストール**

```
apt install -y python3 python3-pip git-core curl
pip3 install gigalixir --user
echo 'export PATH=~/.local/bin:$PATH' >> ~/.bash_profile
source ~/.bash_profile
gigalixir --help
```

**データベース作成**

```
gigalixir pg:create --free -a <アプリ名>
gigalixir pg -a <アプリ名>
gigalxir config -a <アプリ名>
```

**Phoenix側の設定変更**

- host を Gigalixir のドメインに設定する
- prod.secret.exx のimportをコメントアウト

**アプリケーションのデプロイ**

```
git remote add gigalixir <用意されたリポジトリ>
git push gigalixir master
gigalixir run mix ecto.migrate
```

- <用意されたリポジトリ> : GigalixirのWebページ(Setupタブ)に記載されている


## 徒然メモ

---

Q. elixirのバージョン確認方法

`elixir -v` をみる

---

Q. Erlang/OTP のバージョン確認方法

`cat /usr/local/lib/erlang/releases/22/OTP_VERSION` 等。 `which erl` が該当パスのヒントになるかもしれない

---

Q. Gigalixir の環境変数設定方法

`gigalixir config:set MY_CONFIG=foo`

- [Environment variables and secrets](https://gigalixir.readthedocs.io/en/latest/config.html?highlight=environment#environment-variables-and-secrets)

---

Q. Gigalixir で過去に作ったアプリケーションを消したい

Web画面から

- データベースを削除する
- アプリケーションを削除する

データベース削除時に下記のようなエラーが出た

```
There are 2 other sessions using the database. Try `gigalixir ps:scale --replicas=0`.
```

下記で解消

```
gigalixir login
gigalixir ps:scale --replicas=0`
gigalixir ps:destroy -a <アプリ名> -y -d <データベースid>`
```

---