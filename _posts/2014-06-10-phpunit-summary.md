---
layout: post
title: PHPUnit安装
date: 2014-06-10
tags: php
---

pear.phpunit.de网站上声明将不再支持使用PEAR的方式安装PHPUnit，且对应的服务将于2014年底之前停止。

本文介绍另外两种安装PHPUnit的方法：

* PHAR

下载PHPUnit的PHAR文件，这一个文件包含了所有运行PHPUnit所需要的东西

    wget https://phar.phpunit.de/phpunit.phar
	chmod +x phpunit.phar
	mv phpunit.phar /usr/local/bin/phpunit
	
也可以下载后直接使用

    wget https://phar.phpunit.de/phpunit.phar
	php phpunit.phar
	
* Composer

新建一个依赖文件composer.json

    {
        "require-dev": {
        "phpunit/phpunit": "4.1.*"
        }
    }
