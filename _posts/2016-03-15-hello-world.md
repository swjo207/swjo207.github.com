---
layout: post
title: "hello world"
description: ""
category: 
tags: []
---
{% include JB/setup %}


{% highlight ruby %}
desc "Launch preview environment"
task :preview do
  system "jekyll --auto --server"
end # task :preview
{% endhighlight %}
