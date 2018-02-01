---
layout: post
title: "mysql入门(1)-建表和更新及其原则"
date: 2018-01-22 
description: "SQL基本操作-增删改"
tag: Mysql 
---   

> 作为一个大前端工程师, 我们不能止步于传统前端的世界, 我们要向后端发展, 向全栈发展, 因为我们的职业生涯还很长, 在这个可能长达几十年的职业发展中, 充满着挑战和意想不到的困难, 我们要自我武装, 提高自己的能力, 增长自己的见识, 让才华可以支撑梦想...

累了? 困了? 囧!  让我们开始吧...

> mysql是当今最流行的数据库之一, 也是在国内互联网企业中广泛使用的数据库, 为了和大部队保持一致, 让我们从mysql开始学习  

下面来分享一下最近的学习成果 (由于钻研数据库的时间不长, 难免会有疏漏之处, 欢迎批评指正) - mysql的更新操作, 俗称增, 删, 改

### 安装环境
 (略)

### 学习建议
* 使用 Navicat 客户端编写SQL代码 (有语法提示)
* 直接实际操作数据库, 切勿干看书 
* 数据量最好大一些, 使用脚本语言构造百万,千万, 十亿级别数据 (bt!!!)
* 准备好mysql5.7官方参考文档: https://dev.mysql.com/doc/refman/5.7/en/
* 学习完SQL语句之后, 建议学习一下 数据库系统概论(大学专业课程), 增加理论基础
* 入门之后, 看看合适的书: https://www.zhihu.com/question/28385400/answer/40607600

### 创建表
#### 示例: 创建一个 user表, 记录用户信息

```
CREATE TABLE
IF NOT EXISTS cms_user (
    id BIGINT UNSIGNED AUTO_INCREMENT,
    username VARCHAR (255) NOT NULL,
    phone_no BIGINT UNSIGNED UNIQUE DEFAULT NULL,
    sex TINYINT UNSIGNED DEFAULT NULL COMMENT "男 1, 女0",
    addr VARCHAR (255) DEFAULT "",
    is_married TINYINT UNSIGNED DEFAULT NULL,
    asset_total DECIMAL DEFAULT 0,
    gmt_create DATETIME NOT NULL,
    gmt_modified DATETIME NOT NULL,
    PRIMARY KEY (id)
) ENGINE = INNODB DEFAULT CHARSET = utf8;
```
#### ***建表原则:***
* 字段尽可能占用最少的字节的类型, 尽量使用UNSIGNED无符号数据
* 不能使用保留字,关键字
* 表名带上业务号
* 必须要有主键id/创建时间/修改时间
* 小数使用 DECIMAL类型, 避免精度丢失
* 设计字段可以适当冗余, 把经常读取的字段放入其中
* 避免使用外键以减少高并发时外键约束所带来的成本
* 如果单表数据量太大, 建议分表存储
* 根据业务需要选用存储引擎, 一般选用INNODB就行, 它支持事务
* 建议参考一些大厂的文档, 如阿里巴巴Java开发手册终极版 https://github.com/iceyangcc/web-norms


#### ***参考数据类型***
![](/images/mysql/mysql-datatype.png)
#### `日期类型`
![](/images/mysql/mysql-datetype.png)
#### `字符串类型`
![](/images/mysql/mysql-chartype.png)

### 插入数据 INSERT INTO
#### 示例: 插入单条数据

```text
INSERT INTO demo.cms_user (
    id,
    username,
    phone_no,
    sex,
    addr,
    is_married,
    asset_total,
    gmt_create,
    gmt_modified
)
VALUES
    (
        NULL,
        'blog.nodejs.tech',
        18800001111,
        1,
        '北京',
        0,
        1000000,
        '2018-01-22 12:12:12',
        '2018-01-22 12:12:12'
    );
```
#### 示例: 插入多条数据, 使用 values(), (), ()....逗号隔开, 这种方式插入多条效率更高

```text
INSERT INTO demo.cms_user (
    id,
    username,
    phone_no,
    sex,
    addr,
    is_married,
    asset_total,
    gmt_create,
    gmt_modified
)
VALUES
    (
        NULL,
        'blog.nodejs.tech',
        18800001112,
        1,
        '北京',
        0,
        1000000,
        '2018-01-22 12:12:12',
        '2018-01-22 12:12:12'
    ),
    (
        NULL,
        'blog.nodejs.tech',
        18800001113,
        1,
        '北京',
        0,
        1000000,
        '2018-01-22 12:12:12',
        '2018-01-22 12:12:12'
    );
```
***插入数据原则: 必须显示指定要插入的列, 避免数据库字段变更导致出错, 保持程序的健壮***

### 修改数据
#### 示例: 把所有 id > 100 的address置空, phone_no置为NULL
```
UPDATE cms_user SET addr = '', phone_no = NULL WHERE id > 100;
```

***更新原则: 在update时,需要根据需要 加上 where语句, 避免造成误改; 建议在更新之前先 SELECT***

### 删除表
#### 示例: 删除 demo数据库下的cms_user表,  如果删除不存在的表默认会报错, 所以可以加上 IF EXISTS
```
DROP TABLE IF EXISTS demo.cms_user;
```
### 删除表数据
#### PS: 慎用, 在删除之前确保 先 SELECT 查询一下你要删除的数据
#### 示例: 1) 删除id> 100的记录;  2)清空cms_user表, 不带条件则可以清楚所有数据

```text
DELETE FROM demo.cms_user WHERE id > 10;
DELETE FROM  demo.cms_user;
```
或者更快速的清空表所有数据
```
TRUNCATE TABLE  demo.cms_user;
```
*** 注意: TRUNCATE TABLE  是无事务且不触发 触发器的, 不建议在开发中使用, 避免造成严重事故 ***





#### 参考文献:
[阿里巴巴Java开发手册终极版](https://github.com/iceyangcc/web-norms)






不说了, 表演完毕^_^, 我去学习了;  不然, 我又要被大前端的浪潮拍到沙滩上了...


转载请注明原地址，我的新版博客：[首发于http://blog.nodejs.tech/](http://iceyangcc.github.io) 谢谢！
