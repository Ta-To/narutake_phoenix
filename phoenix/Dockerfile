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

# docsify
RUN npm install docsify-cli -g

RUN apt install -y inotify-tools vim lv

WORKDIR /srv
