---
layout: post
title:  "Linux主机共享网络给Windows系统上网"
date:   2015-07-01 20:28:21 +0800
categories: linux network 折腾
---

## 需求背景

* 我的办公电脑是一台使用`Ubuntu Linux`系统的台式机。
* 我的带来了Windows系统的笔记本电脑在公司里用，但是不想用Wifi上网（太慢）。
* 我的办公卡座下就只有一个网线插口，已经被办公电脑用了。
* 这台办公电脑只有一块网卡，用网线插在了卡座唯一的插口上，用来上网。
* 问题来了，怎么才能让这台笔记本电脑通过这台Linux主机上网呢？

## 解决方案

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
