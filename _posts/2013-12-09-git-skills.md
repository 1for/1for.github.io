---
layout: post
title: git技巧
date: 2013-12-09
tags: git
categories: 工具
---

### 日志查看

+ 图形化，显示颜色

{% highlight sh linenos %}
$git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
{% endhighlight %}

- 查找日志
{% highlight sh linenos %}
$git log --grep='find_str'
{% endhighlight %}

- 查看某个文件的历史
{% highlight sh linenos %}
$git log --pretty=oneline file_path
{% endhighlight %}

### 标签相关

- 显示所有标签
{% highlight sh linenos %}
$git tag
{% endhighlight %}

- 匹配显示
{% highlight sh linenos %}
$git tag -l '1.0.*'
{% endhighlight %}

- 创建带注释的标签
{% highlight sh linenos %}
$git tag -a tag_name -m 'tag_description'
{% endhighlight %}

- 上传标签信息
{% highlight sh linenos %}
$git push origin tagname
{% endhighlight %}

- 删除标签
{% highlight sh linenos %}
$git tag -d v1.0
$git push origin :refs/tags/v1.0
{% endhighlight %}

### 追踪分支

- 创建追踪分支
{% highlight sh linenos %}
$git branch --track new_branch origin/master
{% endhighlight %}

- 建立已经存在的本地分支与远程分支的关系
{% highlight sh linenos %}
$git branch --set-upstream local_branch origin/master
{% endhighlight %}

### 提交相关

- 将某次提交应用到另一个分支
{% highlight sh linenos %}
$git checkout no-commit-branch
$git cherry-pick <commit-id>
{% endhighlight %}
