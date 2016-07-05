---
layout: post
title: Linux磁盘管理 LVM(一)
categories: Linux
description: Linux磁盘管理常用的命令，LVM
keywords: Linux，磁盘管理，lvm
---

## 工具介绍

磁盘分区

- sudo fdisk /dev/sda
- cfdisk
- parted 大容量磁盘分区，由于fdisk无法支持到高于2TB以上的分区，此时就需要parted来处理


分区完成后，让系统/prox/partitions识别到让内核重新读取硬件分区表(不重启)

- partx
- partprobe

格式化分区且装载文件系统

- sudo mkfs -t ext3 /dev/sda6 格式化分区，且文件类型ext3
- sudo tune2fs -l /dev/sda6  查看分区的详细信息
- sudo fsck -Cf /dev/sda6 磁盘检测

挂载磁盘到文件目录

- sudo mount /dev/sda6 /data 临时挂载

## 实战LVM新增及扩容

- sudo df -h 查看空间占用
- sudo fdisk -l 查看磁盘分区
- sudo fdisk /dev/sda 创建lvm扩展分区(8e)及逻辑分区(3p+1e或4p，主分区+扩展分区最多不能超过4，但逻辑分区可以无限多)
- sudo partx -a /dev/sda 内核识别新分区，可通过/prox/partitions查看
- sudo mkfs -t ext3 /dev/sda6
- sudo pvdisplay 查看所有pv
- sudo pvcreate /dev/sda6 创建物理卷
- sudo pvdisplay
- sudo vgdisplay 查看所有vg
- sudo vgcreate VolGroup00 /dev/sda6 [/<PV>]... 
- sudo lvdisplay
- sudo lvcreate -l 100%FREE -n testLVM VolGroup00 在vg下创建lv
- sudo mkfs -t ext3 /dev/mapper/VolGroup00-testLVM 格式化lv
- sudo mount /dev/mapper/VolGroup00-testLVM /home/vagrant/testLVM
- 此后若空间不足时的处理方式
- sudo pvcreate /dev/sda5
- sudo vgextend VolGroup00 /dev/sda5 在把新加的pv扩展在要扩容的vg下
- sudo lvextend /dev/VolGroup00/testLVM /dev/sda5 把新加的pv扩展加在要扩容的lv上
- sudo resize2fs /dev/mapper/VolGroup00-testLVM 扩展生效
- sudo df -hT 查看挂载情况

#### 附加一

- 物理存储介质（The physical media）：这里指系统的存储设备：硬盘，如：/dev/hda1、/dev/sda等等，是存储系统最低层的存储单元。
- 物理卷（physical volume）：物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。
- 卷组（Volume Group）：LVM卷组类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或多个物理卷组成。
- 逻辑卷（logical volume）：LVM的逻辑卷类似于非LVM系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home或者/usr等)。

- PE（physical extent）：每一个物理卷被划分为称为PE(Physical Extents)的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是可配置的，默认为4MB。
- LE（logical extent）：逻辑卷也被划分为被称为LE(Logical Extents) 的可被寻址的基本单位。在同一个卷组中，LE的大小和PE是相同的，并且一一对应。

简单来说就是：

- PV:是物理的磁盘分区
- VG:LVM中的物理的磁盘分区，也就是PV，必须加入VG，可以将VG理解为一个仓库或者是几个大的硬盘。
- LV:也就是从VG中划分的逻辑分区

#### 附加二 

**1、硬盘的接口类型**      
*硬盘的接口一般分为两种，一种是IDE并行接口，一种是SATA串行接口， 在linux上面IDE接口的硬盘被识别为/dev/hd[a-z]这样的设备，其中hdc表示光驱设备，这是因为主板上面一般有两个IDE插槽，一个IDE插槽可以接两个硬盘，而光驱是接着IDE的第二个插槽上面的第一个接口上面。其他诸如SCSI，SAS，SATA，USB等接口的设备在linux识别为/dev/sd[a-z]。*    

**2、linux硬盘的分区**
*磁盘的分区分为： primary（主分区）、extended（扩展分区）、Logical （逻辑分区)且主分区加上扩展分区的个数小于等于4个。且扩展分区最多只有一个，扩展分区是不能直接在里面写入数据的，扩展分区里面新建逻辑分区才能读写数据。如果看见一个硬盘有很多分区，则其实是在扩展分区里面新建的逻辑分区。
主分区从 sdb1--sdb4
逻辑分区是从 sdb5--sdbN*

#### 附加三

常见的挂载目录说明

- /        根目录，存放系统命令和用户数据等（如果下面挂载点没有单独的分区，它们都将在根目录的分区中）　 
- /boot    boot loader 的静态链接文件，存放与Linux启动相关的程序
- /home    用户目录，存放普通用户的数据
- /tmp     临时文件
- /usr     是Red Hat Linux系统存放软件的地方,如有可能应将最大空间分给它
- /usr/local 自已安装程序安装在此
- /var     不断变化的数据，服务器的一些服务、日志放在下面。
- /opt     (Option可选的)附加的应用程序软件包 

- /bin    基本命令执行文件 
- /dev    设备文件 
- /etc    主机特定的系统配置 
- /lib    基本共享库以及内核模块 
- /media  用于移动介质的挂载点 
- /mnt    用于临时挂载文件系统或者别的硬件设备（如光驱、软驱） 
- /proc   系统信息的虚拟目录(2.4 和 2.6 内核)，这些信息是在内存中，由系统自己产生的。 
- /root   root 用户的目录 
- /sbin   基本系统命令执行文件 
- /sys    系统信息的虚拟目录(2.6 内核) 
- /srv    系统提供的用于 service 的数据
- /usr/X1186        X-Windows目录，存放一些X-Windows的配置文件
- /usr/include      系统头文件，存储一些C语言的头文件
- /usr/src          Linux内核源代码，Linux系统所安装的内核源代码都保存在此
- /usr/bin          对/bin目录的一些补充
- /usr/sbin         对/sbin目录的一些补充
- /lost+found       这个目录在大多数情况下都是空的。但是如果你正在工作突然停电，或是没有用正常方式关机，在你重新启动机器的时候，有些文件就会找不到应该存放的地方，对于这些文件，系统将他们放在这个目录下。
- /boot:  必须总是物理地包含 /etc、/bin、/sbin、/lib 和 /dev，否则您将不能启动系统。
- /home:  每个用户将放置他的私有数据到这个目录的子目录下。
- /tmp:   程序创建的临时数据大都存到这个目录。
- /usr:   包含所有的用户程序(/usr/bin)，库文件(/usr/lib)，文档(/usr/share/doc)，等等。
- /var:   所有的可变数据，如新闻组文章、电子邮件、网站、数据库、软件包系统的缓存等等，将被放入这个目录。这个目录的大小取决于您计算机的用途，但是对大多数人来说，将主要用于软件包系统的管理工具。如果做服务器的话空间应尽量大。我的服务器的实际分法及实际使用的大小,还没有实际投入使用。所以/var目录没有用那么多。一般WEB存放网页的目录是/var/www,postfix邮件的存放邮件的目录是：/var/mail,var/log，是系统日志记录分区， /var/spool：存放一些邮件、新闻、打印队列等。
- /opt:   存放可选的安装的软件。

>
/         （这就是著名的根）
├── bin         (你在终端运行的大多数程序，比如cp、mv...)
├── boot         (内核放在这里，这个目录也经常被作为某个独立分区的挂载点)
│   └── grub   (grub引导程序和引导菜单就放在这里)
├── cdrom
├── dev         (存放设备文件，这里相当于一个设备管理器，由系统自动生成。视硬件环境不同变化很大)
│   ├── block
│   ├── bsg
│   ├── bus
│   ├── char
│   ├── disk         (磁盘信息，要挂载硬盘分区就要注意这里的信息喽)
│   │   ├── by-id      (硬盘分区的永久性符号链接)
│   │   ├── by-label   (按卷标识别的硬盘分区，常用于挂载)
│   │   ├── by-path   (硬盘分区的节点链接)
│   │   └── by-uuid   (按UUID识别的硬盘分区，常用于挂载)
│   ├── dri
│   ├── fd
│   ├── input
│   ├── net
│   ├── pts
│   ├── shm
│   └── snd
├── etc         (存放所有程序和系统的配制文件和全局变量，对所有用户生效，非常值得备份)
├── home         (这就是著名的home目录了，注意不是”家目录”，强烈建议把一个独立分区挂载到这里！)
│   ├── adagio   (这才是我真正的家！一般来说目录名就是帐号名，当然也可以不是，随便。命令行中用波浪线～代表这里)
│   ├── MNT      (这是我挂载其它硬盘分区的地方，你可以看到用硬盘品牌、容量或用途区分的目录名)
│   │   ├── MAX40NT1   (迈拓40G)
│   │   ├── ST160NT1   (希捷160G第一分区,下面两个类似)
│   │   ├── ST160NT2
│   │   ├── ST160SYS
│   │   ├── ST320G      (希捷320G)
│   │   │   ├── MOVIE
│   │   │   ├── MUSIC
│   │   │   └── P2P   (电驴、BT的缓冲区)
│   │   ├── ST80G      (希捷80G)
│   │   │   ├── PROGRAM
│   │   │   ├── ST80PE
│   │   │   └── YEAR
│   │   └── WD1000      (西数1T)
│   │       ├── WD2
│   │       ├── WD3
│   │       ├── WD5
│   │       ├── WD6.Lib
│   │       └── WD7
│   └── test   (我建立的另一个帐号的家目录，专门用于测试，一旦搞到无法收拾的地步，只需简单的
│                把里面的所有文件删除，就可以恢复默认。实际上你可以拥有无数个帐号)
├── lib         (所有程序共享的库文件)
├── lost+found   (磁盘扫描出现的丢失的数据)
├── media      (你在文件管理器里点击后自动挂载的分区就在这里，按卷标命名，没有卷标则按大小命名)
├── mnt         (同样用于挂载磁盘，这是最传统的位置，喜欢挂哪里随便)
├── opt         (某些特殊的程序喜欢把数据放在这里，比如JAVA)
├── proc         (当前系统所有的详细信息，这里的”文件”并不存在于硬盘中，而是在内存或缓存里，每次启动后都不一样)
├── root       (这是系统最高权威root用户的家！他是老大，所以不住在/home里，那里是草民住的)
├── sbin         (类似/bin，存放常用程序，但这里的程序都是要命的啊，比如格式化，所以只有root用户或sudo程序有权使用)
├── srv         (一些服务所要访问的文件)
├── sys         (系统的核心文件，类似/proc，不必管它)
├── tmp         (存放临时文件，所有用户均可使用，不过你要小心啊，这里的所有文件一旦重启就全没了，自动清空的)
├── usr         (你在X下使用的所有程序数据都在这里了，包括图标、manual等。所有用户都可以使用。也是最庞大的目录)
└── var         (variation，顾名思义就是变量，这里存放系统中经常变化的数据。和/tmp不同啊，很有用的地方)
    ├── backups
    ├── cache
    │   └── apt
    │        └── archives   (存放你安装的所有程序的deb包！重装系统时太有用了，一定要备份好，到时候放回来。
    │                     我建议把整儿/var单独挂载到一个独立分区，像/home一样。这样你重装好系统后，只
    │                      需要简单的把整儿分区挂载到/var就行了，省去了备份-还原的时间。要知道这些deb包
    │                      可不是几十M而已，而是有可能几百M、几个G，一来一回可够呛的。你也可以单独挂载
    │                      一个分区到/var/cache/apt/archives，其他的都不要。
    │                      当然，这样又增加了一点系统构造的复杂度，喜欢怎样请自己斟酌。)
    ├── crash
    ├── games
    ├── lib
    ├── local
    ├── lock
    ├── log      (呵呵，这里的文件是系统运行的完整记录，出了问题一定要来这里看看)
    ├── mail      (这里是存放所有用户email的地方)
    ├── opt
    ├── run
    ├── spool
    └── tmp

40616 directories