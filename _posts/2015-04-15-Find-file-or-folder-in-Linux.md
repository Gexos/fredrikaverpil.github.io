---
layout: post
title: Find file or folder in Linux
tags: [bash, linux]
---

Ever needed to do a simple search for an application, a file or a folder in Linux and when `whereis` doesn’t return anything useful?

{% highlight bash %}
sudo find / -name "some_folder"
{% endhighlight %}
