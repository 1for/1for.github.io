---
layout: post
title: git技巧
date: 2013-12-09
tags: git
categories: 工具
---

### 日志查看

+ 图形化，显示颜色

```$git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit```

- 查找日志

```$git log --grep='find_str'```

- 查看某个文件的历史

```$git log --pretty=oneline file_path ```

### 标签相关

- 显示所有标签

```$git tag```

- 匹配显示

```$git tag -l '1.0.*'```

- 创建带注释的标签

```$git tag -a tag_name -m 'tag_description'```

- 上传标签信息

```$git push origin tagname```

- 删除标签

```$git tag -d v1.0```

```$git push origin :refs/tags/v1.0```

### 追踪分支

- 创建追踪分支

```$git branch --track new_branch origin/master```

- 建立已经存在的本地分支与远程分支的关系

```$git branch --set-upstream local_branch origin/master```

### 提交相关

- 将某次提交应用到另一个分支

```$git checkout no-commit-branch```

```$git cherry-pick <commit-id>```
