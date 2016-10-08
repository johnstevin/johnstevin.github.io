---
layout: post
title: MYSQL-修改大表DDL工具pt-online-schema-change
categories: Mysql
description: MYSQL-修改大表DDL工具pt-online-schema-change
keywords: Mysql,编码,数据库
---

#### MySQL DDL的问题现状

在运维mysql数据库时，我们总会对数据表进行ddl 变更，修改添加字段或者索引，对于mysql 而已，ddl 显然是一个令所有MySQL dba 诟病的一个功能，因为在MySQL中在对表进行ddl时，会锁表，当表比较小比如小于1w上时，对前端影响较小，当时遇到千万级别的表 就会影响前端应用对表的写操作。

目前InnoDB引擎是通过以下步骤来进行DDL的：

1. 按照原始表（original_table）的表结构和DDL语句，新建一个不可见的临时表（tmp_table）
2. 在原表上加write lock，阻塞所有更新操作（insert、delete、update等）
3. 执行insert into tmp_table select * from original_table
4. rename original_table和tmp_table，最后drop original_table
5. 释放 write lock。

我们可以看见在InnoDB执行DDL的时候，原表是只能读不能写的。为此 perconal 推出一个工具 pt-online-schema-change ，其特点是修改过程中不会造成读写阻塞。官网下载地址：https://www.percona.com/downloads/percona-toolkit/ ，安装方法见包内INSTALL文档。

#### pt-online-schema-change工作原理：

最大的优点：不锁表的情况下,修改表结构.该工具执行的基本流程如下:

- 判断各种参数
- 根据原表"t",创建一个名称为"_t_new"的新表
- 执行ALTER TABLE语句修改新表"_t_new"
- 创建3个触发器,名称格式为pt_osc_库名_表名_操作类型,比如
```shell
CREATE TRIGGER `pt_osc_dba_t_del` AFTER DELETE ON `dba`.`t` FOR EACH ROW DELETE IGNORE FROM `dba`.`_t_new` WHERE `dba`.`_t_new`.`id` <=> OLD.`id`
CREATE TRIGGER `pt_osc_dba_t_upd` AFTER UPDATE ON `dba`.`t` FOR EACH ROW REPLACE INTO `dba`.`_t_new` (`id`, `a`, `b`, `c1`) VALUES (NEW.`id`, NEW.`a`, NEW.`b`, NEW.`c1`)
CREATE TRIGGER `pt_osc_dba_t_ins` AFTER INSERT ON `dba`.`t` FOR EACH ROW REPLACE INTO `dba`.`_t_new` (`id`, `a`, `b`, `c1`) VALUES (NEW.`id`, NEW.`a`, NEW.`b`, NEW.`c1`)
```
- 开始复制数据,比如
```shell
INSERT LOW_PRIORITY IGNORE INTO `dba`.`_t_new` (`id`, `a`, `b`, `c1`) SELECT `id`, `a`, `b`, `c1` FROM `dba`.`t` LOCK IN SHARE MODE /*pt-online-schema-change 28014 copy table*/
```
- 复制完成后,交互原表和新表,执行RENAME命令,如 RENAME TABLE t to _t_old, _t_new to t;
- 删除老表,_t_old
- 删除触发器
- 修改完成

#### 解决报错问题：

Can't locate ExtUtils/MakeMaker.pm in @INC (@INC contains:
解决办法：
sudo yum install perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker

Can't locate Time/HiRes.pm
解决办法：
sudo yum -y install perl-Time-HiRes

Cannot connect to MySQL: Cannot connect to MySQL because the Perl DBI module is not installed or not found.
解决办法：
yum install perl-DBI

Character set 'utf8mb4' is not a compiled character set and is not specified in the '/usr/share/mysql/charsets/Index.xml' file
Character set 'utf8mb4' is not a compiled character set and is not specified in the '/usr/share/mysql/charsets/Index.xml' file
Cannot connect to MySQL: DBI connect('test;host=127.0.0.1;mysql_read_default_group=client','root',...) failed: Can't initialize character set utf8mb4 (path: /usr/share/mysql/charsets/) at /usr/local/bin/pt-online-schema-change line 2282
解决办法：
更改/usr/share/mysql/charsets/Index.xml 内容，将
<charset name="utf8">
  <family>Unicode</family>
  <description>UTF-8 Unicode</description>
  <alias>utf-8</alias>
  <collation name="utf8_general_ci"     id="33">
   <flag>primary</flag>
   <flag>compiled</flag>
  </collation>
  <collation name="utf8_bin"            id="83">
    <flag>binary</flag>
    <flag>compiled</flag>
  </collation>
</charset>
改为
<charset name="utf8mb4">
  <family>Unicode</family>
  <description>UTF-8 Unicode</description>
  <alias>utf-8</alias>
  <collation name="utf8_general_ci"     id="33">
   <flag>primary</flag>
   <flag>compiled</flag>
  </collation>
  <collation name="utf8_bin"            id="83">
    <flag>binary</flag>
    <flag>compiled</flag>
  </collation>
</charset>

#### 实验一
CREATE TABLE `online_table` (
  `id` int(11) NOT NULL,
  `name` varchar(10) DEFAULT NULL,
  `age` int(11) DEFAULT NULL
) engine = innodb default charset utf8;

alter table online_table add primary key (id),modify id int not null auto_increment;//使用pt-online-schema-change工具好像要求表必须有索引

pt-online-schema-change --user=rongzhenbang --password=oMw^Szyqusqr2cu3sm --host=120.55.189.210  --alter "ADD COLUMN content text" D=med_data,t=online_table --print --dry-run
pt-online-schema-change --user=rongzhenbang --password=oMw^Szyqusqr2cu3sm --host=120.55.189.210  --alter "ADD COLUMN content text" D=med_data,t=online_table --print --execute
pt-online-schema-change --user=rongzhenbang --password=oMw^Szyqusqr2cu3sm --host=120.55.189.210  --alter "ADD INDEX idx_eventParam1 ( eventParam1 )" D=med_data,t=log_event_test --print --execute

错误：
DBD::mysql::db selectall_arrayref failed: Access denied; you need (at least one of) the REPLICATION SLAVE privilege(s) for this operation [for Statement "SHOW SLAVE HOSTS"] at /usr/local/bin/pt-online-schema-change line 4271.

证明：
1千万数据，用时约15分钟

