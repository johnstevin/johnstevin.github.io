---
layout: post
title: MYSQL-修改大表DDL工具pt-online-schema-change
categories: Mysql
description: MYSQL-修改大表DDL工具pt-online-schema-change
keywords: Mysql,编码,数据库
---

官网：https://www.percona.com/downloads/percona-toolkit/

安装方法见包内INSTALL文档

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

