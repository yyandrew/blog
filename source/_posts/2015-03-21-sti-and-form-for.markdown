---
layout: post
title: "sti and form_for"
date: 2015-03-21 09:34
comments: true
categories: "Rails"
---
**form_for and STI**
---
STI有什么会非常方便。但是有时也比如
<pre>
class User < ActiveRecord::Base
end
</pre>
<pre>
class ForntEndUser < User
end
</pre>
