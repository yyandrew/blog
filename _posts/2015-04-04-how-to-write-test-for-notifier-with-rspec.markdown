---
layout: post
title: "使用rspec+factory_girl测试Rails邮件(ActionMailer)"
date: 2015-04-04 11:38
comments: true
categories: "Rails"
---

邮件测试其实只用对是不是可以发送，邮件标题，发件人，收件人，邮件的内容的关键部分编写测试用例就可以了。
下面对Rails的邮件进行测试.

假设我们有一个mailer用来发邮件：

app/mailers/notifier.rb
```ruby
class Notifier < ActionMailer::Base
  default from: "noreply@example.com"

  def exchange_notifier_mail(rate)
    @rate = rate
    mail(to: "andrew@example.com", subject: "please exchange")
  end
end
```

邮件的内容如下：

app/views/notifier/exchange_notifier_mail.html.haml
```ruby
%div please exchange
%div= "the rate is #{@rate}"
```

使用`User`的**deliver_exchange_notifier**方法来发这封邮件：

app/models/user.rb
```ruby
class User < ActiveRecord::Base
  def deliver_exchange_notifier!
    Notifier.exchange_notifier_mail("6.20").deliver
  end
end
```

在编写测试代码前我们要在**config/environment/test.rb**中添加如下代码：
```ruby
config.action_mailer.delivery_method = :test
```
这一行代码是标记邮件不会被发送出去，它只会被存储在**ActionMailer::Base.deliveries**数组里。接下来我们就可以写测试用例了。

结合**FactoryGirl**创建测试数据：

spec/factories/user.rb
```ruby
FactoryGirl.define do
  factory :user do
    name "Andrew Yang"
    email "andrew@example.com"
  end
end
```

spec/models/user_spec.rb
```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  let(:user) {FactoryGirl.create :user}

  describe 'deliver_password_reset instruction' do
    it 'should send new email' do
      expect {user.deliver_exchange_notifier!}.to change {ActionMailer::Base.deliveries.count}.to(1)
    end
  end
end
```
运行**rspec spec/models/user_spec.rb**可以跑测试用例了。如果输出
```
.

Finished in 0.41145 seconds (files took 9.26 seconds to load)
1 example, 0 failures
```
说明测试通过了。你的代码是可以正常工作的。

下面继续添加notifier测试用例。主要是测试邮件主题，发件人，收件人。添加如下代码：

app/spec/mailers/notifier_spec.rb
```ruby
require "rails_helper"

RSpec.describe Notifier, type: :mailer do
  let(:user) {mock_model User, email: "andrew@example.com"} 
  let(:rate) {"6.20"}

  describe 'exchange notifier mail' do
    it "renders the subject" do
      mail = Notifier.exchange_notifier_mail(rate)
      expect(mail.subject).to eq("please exchange")
    end
    it "send to andrew@example.com" do
      mail = Notifier.exchange_notifier_mail(rate)
      expect(mail.to).to eq(["andrew@example.com"])
    end
    it "send from noreply@example.com" do
      mail = Notifier.exchange_notifier_mail(rate)
      expect(mail.from).to eq(["noreply@example.com"]) 
    end
    it "content should include rate value" do
      mail = Notifier.exchange_notifier_mail(rate)
      expect(mail.body.encoded).to match("6.20" )
    end
  end
end
```
运行**rspec spec/mailers/notifier_spec.rb**。如果输出
```
....

Finished in 0.69902 seconds (files took 8.15 seconds to load)
4 examples, 0 failures
```
那么恭喜你你的代码可以使用没有问题。
