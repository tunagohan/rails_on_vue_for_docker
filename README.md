# Docker Ruby on Rails + Webpacker + MySQL

## 環境変数一覧

設定した方が良き良き

環境変数|説明|デフォルト
:--|:--|:--
MYSQL_ROOT_PASSWORD|[mysql] ルートパスワード|root
MYSQL_DATABASE|[mysql] デフォルトデータベース|test_database
MYSQL_USER|[mysql] 初期投入ユーザー|docker
MYSQL_PASSWORD|[mysql] 初期投入パスワード|docker
MYSQL_PORT|[mysql] mysqlポート|3306
RAILS_PORT|[rails] rails serverポート|3000
WEBPACK_PORT|[webpack] webpacker ポート|3035

## Step 1

### 始めから作りたい人向け

#### Gemfile

```

$ touch Gemfile Gemfile.lock

```

- `vim Gemfile`

```

# Gemfile

source 'https://rubygems.org'
gem 'rails', '~> 6.0.0'

```

#### Docker

```

$ mkdir docker/mysql
$ mkdir docker/rails

```

- `vim docker/mysql/my.cnf`

```

# my.cnf

[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[client]
default-character-set=utf8mb4

```

- `vim docker/rails/Dockerfile`

```

FROM ruby:2.6.1
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev

RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - \
  && apt-get install -y nodejs

RUN apt-get update && apt-get install -y curl apt-transport-https wget && \
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
apt-get update && apt-get install -y yarn

RUN mkdir /app
WORKDIR /app
COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock
RUN bundle install
COPY . /app

CMD ["rails", "server", "-b", "0.0.0.0"]

```

- `vim docker-compose.yml`

```

version: '3'
services:
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: test_database
      MYSQL_USER: docker
      MYSQL_PASSWORD: docker
      TZ: 'Asia/Tokyo'
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    volumes:
      - mysql_data:/var/lib/mysql
      - ./docker/mysql/my.cnf:/etc/mysql/conf.d/my.cnf
    ports:
      - "3316:3306"

  web:
    build:
      context: .
      dockerfile: ./docker/rails/Dockerfile
    environment:
      RAILS_ENV: development
      MYSQL_USER: root
      MYSQL_PASSWORD: root
      MYSQL_HOST: db
      RAILS_SERVE_STATIC_FILES: "false"
    command: bash -c "bundle exec rails s -p 3000 -b '0.0.0.0'"
    tty: true
    stdin_open: true
    depends_on:
      - db
      - webpacker
    volumes:
      - .:/app
      - bundle_data:/usr/local/bundle
    ports:
      - "3010:3000"

  webpacker:
    build:
      context: .
      dockerfile: ./docker/rails/Dockerfile
    command: bundle exec bin/webpack-dev-server
    tty: true
    stdin_open: true
    volumes:
      - .:/app
    ports:
      - "3045:3035"

volumes:
  bundle_data:
  mysql_data:

```

#### Execute


```

$ docker-compose run --rm web rails new . --force --database=mysql --webpack=vue --skip-coffee

$ sed -i ".bak" -e "s/host: localhost/host: webpacker/g" config/webpacker.yml

$ docker-compose build

$ docker-compose run web rails db:create

$ docker-compose up -d

```

## Cloneして使いたい人向け

```

$ docker-compose up -d

```

## Step 2

```

$ docker-compose run --rm web rails db:create

```

もし `error Couldn't find an integrity file` のようなエラーが出たら

```

$ docker-compose run --rm web yarn
$ docker-compose run --rm web rails db:create

```

or

```

$ bin/setup

```

## Step 3

開発へGo~
