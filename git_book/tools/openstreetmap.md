---
description: OpenStreetMap是一个可供自由编辑的世界地图，Docker 私有化部署。
---

# 🐯 OpenStreetMap

官网 [https://www.openstreetmap.org/](https://www.openstreetmap.org/)

openstreetmap 包括websit 和db(postgres), 数据全部保存在DB中。Docker安装部署过程制作两个镜像。

### service 镜像 和 db 镜像dockerfile如下

{% embed url="https://github.com/zhaolibingit/docker_file/tree/master/osm" %}

{% tabs %}
{% tab title="Dockerfile-db" %}
```
FROM postgres:12

ADD docker_postgres.sh docker-entrypoint-initdb.d/docker_postgres.sh

# compiling ruby from source as official debian has ruby 2.3 and we need => 2.4
RUN apt-get update
RUN apt-get install -y git vim wget curl build-essential libreadline-dev libssl-dev libcurl4-openssl-dev postgresql-server-dev-all zlib1g-dev libxml2-dev
RUN curl -L https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.4.tar.gz | tar zx
RUN cd ruby-2.6.4 && ./configure && make && make install
RUN cd ..  && rm -r ruby-2.6.4
RUN apt-get clean && rm -rf /var/lib/apt/lists/*


# Setup app location
RUN mkdir -p /app
WORKDIR /app

RUN cd /tmp/  && git clone --depth=1 -b docker https://github.com/zhaolibingit/openstreetmap-website.git

# add database functions directory
RUN mkdir -p /app/db/functions/
RUN cp -r /tmp/openstreetmap-website/db/functions/ /app/db/functions/
RUN cp -r /tmp/openstreetmap-website/Gemfile* /app/
RUN bundle install

RUN apt-get update

# change ownership to postgres as while running docker_postgres.sh postgres will need write access
RUN chown -R postgres /app/db

RUN rm -rf /tmp/*
```
{% endtab %}

{% tab title="Dockerfile-srv" %}
```
FROM ruby:2.5-slim

# install packages
# fixes dpkg man page softlink error while installing postgresql-client [source: https://stackoverflow.com/a/52655008/5350059]
RUN mkdir -p /usr/share/man/man1 && mkdir -p /usr/share/man/man7
RUN apt-get update && apt-get install curl wget vim git -y
RUN apt-get clean && rm -rf /var/lib/apt/lists/*


#npm is not available in Debian repo so following official instruction [source: https://github.com/nodesource/distributions/blob/master/README.md#debinstall]
RUN curl -sL https://deb.nodesource.com/setup_10.x -o nodesource_setup.sh && bash nodesource_setup.sh && rm nodesource_setup.sh
RUN apt-get install -y --no-install-recommends ruby-dev libarchive-dev libmagickwand-dev libxml2-dev libxslt1-dev build-essential libpq-dev libsasl2-dev imagemagick libffi-dev locales postgresql-client nodejs osmosis

RUN apt-get install -y wget
RUN apt-get remove postgresql-client -y

RUN sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc |  apt-key add -
RUN apt-get update
RUN apt-get install postgresql-client-12 -y
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN npm install yarn phantomjs-prebuilt -g --unsafe-perm


# Setup app location
RUN mkdir -p /app
WORKDIR /app

# Install gems
RUN cd /tmp/  && git clone --depth=1 -b docker https://github.com/zhaolibingit/openstreetmap-website.git
RUN cp -r /tmp/openstreetmap-website/* /app/
RUN touch /app/config/settings.local.yml
RUN cp /app/config/example.storage.yml /app/config/storage.yml
RUN cp /app/config/example.database.yml /app/config/database.yml
RUN rm -rf /tmp/*

RUN bundle install
# Setup local
RUN sed -i -e 's/# en_GB.UTF-8 UTF-8/en_GB.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="en_GB.UTF-8"'>/etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_GB.UTF-8

ENV LANG en_GB.UTF-8
```
{% endtab %}

{% tab title="docker_postgres.sh" %}
```
#!/bin/bash
set -e
psql -v ON_ERROR_STOP=1 -U "$POSTGRES_USER" -c "CREATE EXTENSION btree_gist" openstreetmap
make -C db/functions libpgosm.so
psql -v ON_ERROR_STOP=1 -U "$POSTGRES_USER" -c "CREATE FUNCTION maptile_for_point(int8, int8, int4) RETURNS int4 AS '/app/db/functions/libpgosm', 'maptile_for_point' LANGUAGE C STRICT" openstreetmap
psql -v ON_ERROR_STOP=1 -U "$POSTGRES_USER" -c "CREATE FUNCTION tile_for_point(int4, int4) RETURNS int8 AS '/app/db/functions/libpgosm', 'tile_for_point' LANGUAGE C STRICT" openstreetmap
psql -v ON_ERROR_STOP=1 -U "$POSTGRES_USER" -c "CREATE FUNCTION xid_to_int4(xid) RETURNS int4 AS '/app/db/functions/libpgosm', 'xid_to_int4' LANGUAGE C STRICT" openstreetmap

```
{% endtab %}
{% endtabs %}

### 已经在阿里云镜像服务生成镜像，可直接使用如下

```
docker pull registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/osm:service
docker pull registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/osm:db_postgres
```

docker-compose.yaml 如下

```
version: '2'
services:
  web:
    image: registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/osm:service
    ports:
      - "3000:3000"
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    depends_on:
      - db
    environment:
      DATABASE_URL: postgres://postgres@db:5432
  db:
    image: registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/osm:db_postgres
    ports:
      - "5432:5432"
    volumes:
      - ./pg_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: openstreetmap
      POSTGRES_HOST_AUTH_METHOD: trust
```

`docker-compose up -d 启动即可`

### 初始化数据库

```
docker-compose -f docker-compose.yml exec web bundle exec rake db:migrate
```

### nodejs 使用yarn 来管理包

```
docker-compose -f docker-compose.yaml exec web bundle exec rake yarn:install
```

### 注册用户 http://127.0.0.1:3000

![](<../../.gitbook/assets/image (7).png>)

### 修改数据库激活用户

![](<../../.gitbook/assets/image (13).png>)

### 创建auth 认证，iD编辑器就可以用了

![](<../../.gitbook/assets/image (10).png>)

![](<../../.gitbook/assets/image (12).png>)

注册完成后显示

![](<../../.gitbook/assets/image (8).png>)

### 更改iD 认证配置，开启编辑功能

```
#  docker-compose -f docker-compose.yaml exec web bash
root@a0dda688c030:/app# cat  /app/config/settings.local.yml
# Default editor
default_editor: "id"
# OAuth consumer key for Potlatch 2
potlatch2_key: "TGKEVoqTPoJWW7jWApgP9Uo98amtDNJHJF8t7zzR"
# OAuth consumer key for the web site
#oauth_key: ""
# OAuth consumer key for iD
id_key: "TGKEVoqTPoJWW7jWApgP9Uo98amtDNJHJF8t7zzR"

# The maximum area you're allowed to request, in square degrees
max_request_area: 5
# Number of GPS trace/trackpoints returned per-page
tracepoints_per_page: 10000
# Maximum number of nodes that will be returned by the api in a map request
max_number_of_nodes: 100000
# Maximum number of nodes that can be in a way (checked on save)
max_number_of_way_nodes: 10000
# The maximum area you're allowed to request notes from, in square degrees
max_note_request_area: 50
```

![](<../../.gitbook/assets/image (6).png>)

更改完成后iD 编辑器可用，如下

![](<../../.gitbook/assets/image (9).png>)

### 数据导入

```
docker-compose -f docker/docker-compose.yml exec web osmosis --read-pbf /path/to/file.osm.pbf --write-apidb host="db" database="openstreetmap" user="postgres"  validateSchemaVersion="no"
```
