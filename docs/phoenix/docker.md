開発環境をdockerで用意する際のメモです

## Dockerfile

- Elixir (base image)
- NODE
- Phoenix
- Gigalixir (Gigalixirにデプロイする際に使用するCLI)

```dockerfile: phoenix/Dockerfile
FROM elixir:1.10

ENV NODE_VERSION 12.x
ENV NPM_VERSION 6.14.6
ENV PHOENIX_VERSION 1.5.4

# NODE/NPM
RUN curl -sL https://deb.nodesource.com/setup_${NODE_VERSION} | bash \
  && apt-get install -y nodejs
RUN npm install npm@${NPM_VERSION} -g

# Phoenix
RUN mix local.hex --force && \
  mix archive.install --force hex phx_new ${PHOENIX_VERSION} && \
  mix local.rebar --force

# gigalixir
RUN apt install -y python3 python3-pip git-core curl
RUN pip3 install gigalixir

RUN apt install -y inotify-tools

WORKDIR /srv
```


## docker-compose

- アプリケーションとデータベースのコンテナで構成
- 設定は `.env` ファイルを参照
- データベースは `../postgresql/phoenix` をマウントするなどして永続化

```yaml: docker-compose.yml
version: "3.7"

services:
  app:
    build: ./phoenix
    container_name: phoenix
    shm_size: 512m
    command: tail -f /dev/null
    env_file:
      - ./.env
    environment:
      - MIX_ENV=dev
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "100"
    ports:
      - "4000:4000"
    volumes:
      - ./:/srv/

  db:
    image: postgres
    container_name: phoenix_db
    shm_size: 512m
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_HOST=db
    volumes:
      - ../postgresql/phoenix:/var/lib/postgresql/data
```
