---
layout: post
title: 尝试在Docker容器里开发Rails应用
date: 2020-10-28 17:34 +0800
---

通过这篇文章，希望各位读者能够掌握如下内容：

1. 创建Dockerfile
2. 创建Docker compose配置文件
3. 使用Docker compose命令启动Rails服务

需要用到的工具：

1. [docker](https://docs.docker.com/engine/install/)
2. [docker compose](https://docs.docker.com/compose/install/)
3. [asdf-vm](https://github.com/asdf-vm/asdf)
4. [asdf-vm] ruby插件(https://github.com/asdf-vm/asdf-ruby)

最终的效果是：

1. 将ruby gem和npm package安装到一个容器，并在这个容器启动Rails服务器
2. 在另外一个容器运行postgresql数据库。
3. 通过宿主机的浏览器可以访问`http://localhost:3000`
4. 在宿主机上修改文件之后，不用重新构建容器就能够在浏览器里查看修改(不准确，应该是不需要重启Rasils服务的修改)。

#### 安装ruby

以后有时间再添加:dog:

#### 创建一个新的Rails应用

```shell
mkdir rails_with_docker
cd rails_with_docker
rails new . -d postgresql
```

#### 创建Dockerfile

在`rails_with_docker`根目录下创建一个新文件`Dockerfile`，内容如下：

```yaml
# 使用ruby 2.7.1
FROM ruby:2.7.1

# 安装依赖工具
RUN apt-get update && apt-get install -y \
  curl \
  build-essential \
  libpq-dev &&\
  curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
  curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  apt-get update && apt-get install -y nodejs yarn

# 创建一个项目目录
RUN mkdir /app
# 设置工作目录，设置好了之后就不用每次执行下面的拷贝文件命令之前都要cd到这个目录
WORKDIR /app

# 暴露3000端口
EXPOSE 3000

# 准备Gemfile文件
COPY Gemfile .
COPY Gemfile.lock .
# 安装bundler及gems
RUN gem update bundler
RUN bundle install --jobs 5

# 将package.json及yarn.lock拷贝到docker的工作目录
COPY package.json .
COPY yarn.lock .
# 安装npm packages
RUN yarn install
```

#### 创建docker-compose.yml

在`rails_with_docker`根目录下创建一个新文件`docker-compose.yml`m内容如下：

```yaml
version: '3'
services:
  # 准备postgresql服务
  db:
    image: postgres:10.5-alpine
    # 将数据库服务的5432端口映射到主机的5432端口
    ports:
      - 5432:5432
    # 设置数据库服务和主机的共享目录
    volumes:
      - ./tmp/postgres_data:/var/lib/postgresql/data
    # 设置PostgresQL数据库用户密码
    environment:
      POSTGRES_PASSWORD: password
  web:
    # 构建web容器
    build: .
    # 启动Rails服务器
    command: /bin/bash -c "rm -f /tmp/server.pid && bundle exec rails server -b 0.0.0.0 -P /tmp/server.pid"
    # 将web服务的3000端口映射到主机的3000端口,这样在主机的的浏览器可以访问http://localhost:3000
    ports:
      - 3000:3000
    # 依赖数据库服务
    depends_on:
      - db
    # 设置web服务的共享目录
    volumes:
      - .:/app
```

注意：`POSTGRES_PASSWORD`可以随意设置，这个密码会在`config/database.yml`文件里使用。

#### 修改Rails应用的数据库配置

打开`config/database.yml`，将其内容替换成

````yaml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db
  username: postgres
  # 来自docker-compose.yml文件里的`POSTGRES_PASSWORD`值
  password: password

development:
  <<: *default
  database: rails_with_docker_development
test:
  <<: *default
  database: rails_with_docker_test
production:
  <<: *default
  database: rails_with_docker_production
````
注意：配置文件里的**host**非常重要，它不是常见的localhost或者ip地址，这里它是一个docker compose服务**db**。
#### 使用docker compose构建Rails应用所需的所有服务

```shell
docker-compose build
docker-compose run web rake db:create
docker-compose run web rake db:migrate
docker-compose run web rails webpacker:install
docker-compose run web yarn install
docker-compose up
```

#### 使用浏览器测试

打开`http://localhost:3000`，不也意外你可以看到如下的页面：

![home-page-of-rails-in-docker.png](/images/home-page-of-rails-in-docker.png)

#### 如何停止服务

```
docker-compose down
```

