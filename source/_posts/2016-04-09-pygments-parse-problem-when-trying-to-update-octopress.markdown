---
layout: post
title: "升级octopress造成的pygments解析错误问题"
date: 2016-04-09 18:37:59 +0800
comments: true
categories: "Blog"
---
今天看了看octpress的版本实在是太低了，用的jeklly的版本还是`0.1`，于是开始着手升级。按照[官方的文档](http://octopress.org/docs/updating/)升级还算是比较顺利。

结果在用`rake generate`命令重新生成静态文件的时候报错了。错误信息如下：

``` bash
~$ rake generate

...

jekyll 2.5.3 | Error:  Pygments can't parse unknown language: bash</p>.

```

通过调查发现这个错误是由于octpress的自带的`source/plugins/pygments_code.rb`插件抛出的。代码如下所示：

``` ruby
...

def self.pygments(code, lang)
  if defined?(PYGMENTS_CACHE_DIR)
    path = File.join(PYGMENTS_CACHE_DIR, "#{lang}-#{Digest::MD5.hexdigest(code)}.html")
    if File.exist?(path)
      highlighted_code = File.read(path)
    else
      begin
        highlighted_code = Pygments.highlight(code, :lexer => lang, :formatter => 'html', :options => {:encoding => 'utf-8', :startinline => true})
      rescue MentosError
        raise "Pygments can't parse unknown language: #{lang}."
      end
      File.open(path, 'w') {|f| f.print(highlighted_code) }
    end
  else
    highlighted_code = Pygments.highlight(code, :lexer => lang, :formatter => 'html', :options => {:encoding => 'utf-8', :startinline => true})
  end
  highlighted_code
end
...
```

我们可以修改一下上述代码查看一下是哪个代码块的生成有问题。

```ruby
...

def self.pygments(code, lang)
  puts "***********"
  puts "\npath: #{path}"
  puts "***********"
  puts "\ncode: #{code}"
  puts "***********"
  puts "\nlang: #{lang}"
...
```

### 解决方法
1.在```和[language]中间添加空格
2.``` [language]代码块必须和其它的文本用一个空白行隔开
