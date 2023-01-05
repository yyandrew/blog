---
layout: post
title: 优化Rails的remote_ip方法
date: 2023-01-05 12:42 +0800
---
1. 在新建一个`lib/middleware/client_ip.rb`，内容如下

``` ruby
# frozen_string_literal: true

class ClientIp
  def initialize(app)
    @app = app
  end

  def call(env)
    remote_ip = ActionDispatch::RemoteIp.new(env)
    client_ip = ActionDispatch::RemoteIp::GetIp.new(env, remote_ip)

    forwarded_ips = client_ip.send(:ips_from, 'HTTP_X_FORWARDED_FOR')
    real_ips = client_ip.send(:ips_from, 'HTTP_X_REAL_IP')
    client_ips = client_ip.send(:ips_from, 'HTTP_CLIENT_IP')
    remote_addr = client_ip.send(:ips_from, 'REMOTE_ADDR').last

    ips = [forwarded_ips, real_ips, client_ips, remote_addr].flatten.compact
    puts "ips!!! #{ips}"
    env['action_dispatch.client_ip'] = client_ip.send(:filter_proxies, ips).first || remote_addr
    @app.call(env)
  end
end
```

{:start="2"}
2. 修改`config/application.rb`文件，将上面的middleware引入到项目中

``` ruby
require_relative '../lib/middleware/client_ip'
module YouApp
  class Application < Rails::Application
    config.middleware.use ClientIp
	end
end
```

{:start="3"}
3. 将下面方法添加到`app/controllers/application_controller.rb`共享给所有的控制器，在其它的控制器调用`client_ip`方法就可以获取的ip地址。

``` ruby
  def client_ip
    env["action_dispatch.client_ip"]
  end
```
