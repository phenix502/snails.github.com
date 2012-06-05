---
layout:       post
title:        "修改Rails默认生成的Gemfile的source"
description: 
banner: 
categories: 
- 学习
---

由于大陆的“特殊情况”，rails默认生成的Gemfile的源 <https://rubygems.org>很慢甚至被重置，所以适应国情，要修改下Rails默认生成的Gemfile文件。

如何做呢？

很简单，切换到Rails的默认模板路径下，修改Gemfile文件的source:
	1.cd $rvm_path/gems/ruby-1.9.3-p194@rails32/gems/railties-3.2.5/lib/rails/generators/rails/app/templates/
	2.修改Gemfile文件，替换https://rubygems.org为`http://ruby.taobao.org`

Over~~
	