---
layout: post
title: "MySQL执行计划分析"
date: 2018-02-08
description: "SQL执行计划分析和性能优化"
tag: MySQL 
---   


> 在工作中, 每天都要和后台打交道, 有时候看到他们在群里反应的一个问题就是数据返回很慢, 而慢的原因呢, 主要就是查询数据库很慢, 经过一定的SQL查询优化之后, 性能得以大幅提升, 我对此很是好奇, 于是对SQL查询优化进行了一定的研究.

今天来介绍一下MySQL的执行计划分析, 分析SQL执行计划对我们找到慢查询的原因是至关重要的, 只有找到了慢在什么地方, 才能解决这个问题

执行计划能告诉我们什么?
* SQL如何使用索引
* 联结查询的执行顺序
* 查询扫秒的数据行数



转载请注明原地址，[本文首发于http://blog.nodejs.tech/](http://blog.nodejs.tech) 谢谢！