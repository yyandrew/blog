---
layout: post
title: 为Rails API添加文档的一次尝试
date: 2020-09-23 10:35 +0800
---

## TL;DR

使用`apipie-rails`生成json格式的文件，`rswag-ui`使用这个json文件，通过经典的swagger ui界面展示API文档。

## 具体步骤

1. 安装 Gem

   ```shell
   gem 'rswag-ui'
   gem 'apipie-rails'
   ```

2. 修改controller文件，添加一个endpoint文档说明

   ```ruby
   module Api
     module V1
       module Users
         class SessionsController < Devise::SessionsController
           api :DELETE, '/users/sign-out', 'Sign out'
           def destroy
             sign_out(:user)
             head :no_content
           end
       end
     end
   end
   ```

3. 生成swagger json

   ```shell
   rake apipie:static_swagger_json
   ```

4. 设置swagger-ui使用json

   1. 为rswag-ui生成配置文件

      ```shell
      rails g rswag:ui:install
      ```

   2. 修改`config/initializers/rswag-ui.rb`使它能够使用`apipie`生成的json文件

      ```ruby
      Rswag::Ui.configure do |c|
      	...
      	c.swagger_endpoint '/doc/apidoc/schema_swagger_form_data.json', 'API V1 Docs' 
      	...
      end
      ```

   最终的效果图

   ![swagger-with-apipie.png](/images/swagger-with-apipie.png)

## 定制SwaggerUi

如果我想禁止页面上的`Try it out!`的功能，应该怎么做呢？这时候就需要修改`swagger-ui的html源文件了`。

通过[swagger-ui](https://github.com/swagger-api/swagger-ui/tree/2.x)的官方文档可以找到需要设置一下`supportedSubmitMethods`。要在哪里设置呢？你可能已经猜到需要修改html源文件(上面已经提到)。

1. 通过[rswag](https://github.com/rswag/rswag#customizing-the-swagger-ui)命令(需要安装[rswag](https://github.com/rswag/rswag) gem)行生成html源文件

   ```shell 
   $ rails g rswag:ui:custom
   Running via Spring preloader in process 77177
         create  app/views/rswag/ui/home/index.html.erb
   ```

   

2. 将`supportedSubmitMethods`添加到`configObject`

   ```javascript 
   <script>
     window.onload = function () {
       var configObject = JSON.parse('<%= config_object.to_json %>');
       var oauthConfigObject = JSON.parse('<%= oauth_config_object.to_json %>');
   
       // Apply mandatory parameters
       ... // 省略中间不重要的代码
       configObject.supportedSubmitMethods = []; // []表示禁止所有action的Try it out
   
       ... // 省略中间不重要的代码
       const ui = SwaggerUIBundle(configObject);
   
       // Apply OAuth config
       ui.initOAuth(oauthConfigObject);
     }
   </script>
   
   ```

   从下面的截图可以看出`Try it out`按钮已经消失了。

   ![disable-try-it-out.png](/images/disable-try-it-out.png)