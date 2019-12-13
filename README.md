# sql

### 查询每天最后的一条数据sql

select *, from celebrityHistoryDouyin t1 WHERE t1.id = (SELECT max(id) FROM celebrityHistoryDouyin t2 where LEFT(FROM_UNIXTIME(t1.time/ 1000, "%Y-%m-%d %H:%i:%s"),10) = LEFT(FROM_UNIXTIME(t2.time/ 1000, "%Y-%m-%d %H:%i:%s"),10) and celebrityId = 1);

## 一，主从切换
### 1，QPS和TPS
    并发量 & CPU
    并发量：
        同一时间处理的请求的数量
        数据库同步远程计划任务 耗费磁盘I/O
        最好不要在主数据库备份 ，大型活动前取消这类计划

### 2，在大促中什么影响了数据库性能
    sql查询速度
    服务器硬件 
    网卡流量
    磁盘I/O

    超高的QPS和TPS
    风险：
        效率低下的SQL
        网站访问量增长，必然有大量的QPS和TPS，就要有效率高的SQL
        大量的并发和超高的使用率
    风险：
    大量的并发：
        数据库连接数被占满（max_connerctions默认100）
    超高的CUP使用率：
        因CPU资源耗尽而出现宕机
        连接量和并发量不一样
    
    磁盘I/O
    风险：
        *磁盘I/O性能突然下降，通常发生在热数据大于可用服务器内存的情况下
        *其他大量消耗磁盘性能的计划任务（调整计划任务：在大促之前把备份任务切换到从服务器上面进行，  
        做好磁盘维护：服务器磁盘都是有通告的）

    网卡流量
    风险：网卡I/O被占满
    如何避免无法连接数据库的情况：
        1，减少从服务器的数量，从服务器要从主服务器上复制日志
        2，进行分级缓存
        3，避免使用“select *“进行查询
        4，分离业务网络和服务器网络
    
    还有什么会影响数据库性能！什么样的表可以称之为大表
        *记录行数巨大，单表超过千万行
        *表数据文件巨大，表数据文件超过10G
    注意：
        *如果表中仅有insert，和少数select 而几乎没有update和delete操作，对业务不会有什么影响！大表对查询的影响
        *慢查询：很难在一定时间内过滤出所有的数据
        ！大表对DDL操作的影响
            建立索引需要很长时间
        风险：
            MySQL版本<5.5建立索引会锁表
            MySQL版本>5.5 虽然不会锁表但会引起主从延迟
        ！修改表结构需要长时间锁表
        风险：
            会造成长时间的主从延迟
            影响正常数据库操作
        如何处理数据库中的大表
        分库分表把一张大表分成多个小表
        难点：
            分表主键的选择
            分表后跨分区数据的查询和统计
            大表的历史数据归档减少对前端业务的影响
        难点：
            归档的时间点的选择（订单表归档一年前的数据，日志归档一周前的数据）
            如何进行归档操作
    ！大事务带来的问题
    什么是大事务
    定义：运行时间比较长，操作数据比较多的事务
    风险：
        锁定太多的数据，造成大量的阻塞和锁超时
        回滚时所需要的时间比较长
        执行时间长，容易造成主从延迟
    如何处理大事务
    1.避免一次处理太多的数据
    2.移除不必要在事务中的SELECT操作


## 二、影响性能的几个方面
    1、
    服务器硬件
    服务器系统
    数据库存储引擎的选择
    *数据库参数配置
    数据库表结构的设计和SQL语句（慢查询）

    2、CPU资源和可用内存大小
    CPU密集型选择频率高的
    并发型选择核心数多的
    MyISAM 会把索引存储到内存中数据存储到系统中
    InnoDB 会同时在内存中缓存索引和数据
    内存大于数据库文件大小即可
    数据在内存中达到一定数量写入磁盘

    3、磁盘的配置和选择
    传统机械硬盘

    4、使用RAID增加传统机械硬盘的性能
    RAID是磁盘冗余队列的简称
    RAID5
    RAID10
    
    5、使用固态存储SSD或PCle卡
    增加从服务器的I/O性能
    6、使用网络存储SAN和NAS
    SAN和NAS是两种外部文件存储设备加载到服务器上的方法
    SAN设别通过光纤连接到服务器，设备通过块接口访问服务器可将其当作硬盘使用（随机读写慢）
    NAS设备使用网络连接，通过基于文件的协议如NFS或SMB来访问
    网络存储使用的场景：数据库备份
    网络对性能的影响
    建议：
        采用高性能和高带宽的网络接口设备和交换机
        对多个网卡进行绑定，挣钱可用性和带宽
        尽可能地进行网络隔离
    7、服务器硬件对性能的影响
    CUP：
        64位的CPU一定要工作在64位的系统下
        对于并发比较高的场景CPU的数量比频率重要
        对于CPU密集型场景和复杂SQL则频率越高越好
    内存：
        选择主板所能使用的最高频率的内存
        内存的大小对性能很重要，所以尽可能的大
    I/O子系统：PCIE->SSD->Raid10>磁盘->SAN
    8、操作系统对性能的影响
        内核相关参数（/etc/sysctl.conf）
        net.core.somaxconn = 65535
        net.core.netdev_max_backlog = 65535
        net.ipv4.tcp_max_syn_backlog = 65535
        net.ipv4.tcp_fin_timeout = 10
        net.ipv4.tcp_tw_reuse = 1
        net.ipv4.tcp_tw_recycle = 1
        net.core.wmen_default = 87380
        net.core.wmen_max = 16777216
        net.core.rmen_default = 87380
        net.core.rmen_max = 16777216
        net.ipv4.tcp_keepalive_time = 120
        net.ipv4.tcp_keepalive_intvl = 30
        net.ipv4tcp_keepalive_probes = 3
        kernel.shmmax = 4294967295（注意：1、这个参数应该设置的足够大，以便能在一个共享内存段下容纳整个Innodb缓冲池的大小。2、这个值得大小对于64位Linux系统，可取的最大值为物理内存值-1byte，建议值为大于物理内存的一般，一般取值大于Innodb缓冲池的大小即可，可以去物理内存-1byte。）
        vm.swappiness = 0
        增加资源限制（/etc/security/limit.conf）
        * soft nofile 65535
        * hard nofile 65535 （加到limit.conf文件的末尾就可以了）
        磁盘调度策略（/sys/block/devname/queue/scheduler）
        cat /sys/block/sda/queue/scheduler
        noop anticipatory deadline [cfq]
        echo <schedulername> /sys/block/devname/queue/scheduler（修改磁盘调度策略命令）
    10、文件系统共对性能的影响
        Windows：FAT、NTFS   选择NTFS
        Linux：EXT3、EXT4、XFS   选择XFS
        EXT3/4系统的挂载参数（/etc/fstab）
        data = writeback | ordered |journal
        noatime,nodiratime
        /dev/sda1/etx4  noatime,nodiratime,data=writeback 1 1
    11、MySQL体系结构
        客户端  PHP，JAVA，C API，.Net以及ODBC，JDBC等
        连接管理器    MySQL服务层
        存储引擎层
    12、MySQL常用存储引擎 MyISAM 是大部分系统表和临时表所使用的存储引擎
    临时表：在排序、分组等操作中，当数量超过一定大小之后，由查询优化器建立的临时表  
    MyISAM存储引擎表由MYD和MYI组成
    特性：
        并发性与锁级别
        表损坏修复（check table tablename      repair table tablename   进行修复）
        MyISAM表支持的索引类型（全文索引、前缀索引）
        MyISAM表支持数据压缩（再不对该表进行操作的情况下（命令行：myisampack -b -f））
    限制：
        适用场景：非事务型应用         只读类应用            空间类应用
    13、MySQL常用存储引擎 Innodb
    Innodb使用表空间进行数据存储（innode_file_per_table     ON:独立表空间:tablename.ibd       OFF:系统表空间:ibdataX   （命令行：show variables like 'innodb_file_per_table'，set global innodb+file_per_table = off））
    系统表空间和独立表空间要如何选择
    比较：
        系统表空间无法简单的收缩文件大小
        独立表空间可以通过optimize table命令收缩系统文件
        系统表空间会产生I/O瓶颈
        独立表空间可以同时向多个文件刷新数据
    建议：
        对Innodn使用独立表空间
        表转移的步骤
        把原来存在于系统表空间中的表转移到独立表空间中的方法
    步骤：
        使用mysqldump导出数据库表数据
        停止MySQL服务，修改参数，并删除Innodb相关文件
        重启MySQL服务，重建Innodb系统表空间
        重新导入数据
    14、Innodb存储引擎的特性（1）
    系统表空间和独立表空间要如何选择
    Innodb 数据字典信息
    Undo 回滚段
    Innodb存储引擎的特性：
        Innodb是一种事务型存储引擎
        完全支持事务的ACID特性
        Redo Log 和 Undo Log
        Innodb支持行级锁
        行级锁可以最大程度的支持并发
        行级锁是由存储引擎层实现的
    什么是锁：
        锁的主要作用是管理共享资源的并发访问
        锁用于实现事务的隔离性
    锁的类型：共享锁（也称读锁）           独占锁（也称写锁）  
    锁的粒度：表级锁                      行级锁
    
    15、MySQL常用存储引擎Innodb（2）
    什么是阻塞：
    什么是死锁：
    Innodb状态检查：show engine innodb status
    适用场景：Innodb适用于大多数OLTP应用（支持全文索引）
    
    16、MYSQL常用存储引擎CSV
    文件系统存储特点：
        · 数据以文本方式存储在文件中  
        · .CSV文件存储表内容    
        · .CSM文件存储表的元数据如表状态和数据量       
        · .frm文件存储表结构信息
    特点：
        ·以CSV格式进行数据存储    
        ·所有列必须都是不能为NULL的    
        ·不支持索引（不适合大表，不适合在线处理）    
        ·可以对数据文件直接编辑
    
    17、MYSQL常用存储引擎Archive
    文件系统存储特点：以zlib对表数据进行压缩，磁盘I/O特少       数据存储在ARZ为后缀的文件中
    Archive存储引擎的特点：只支持insert和select操作      只允许在自增ID列上加索引
    适用场景：日志和数据采集类应用
    
    18、MySQL常用存储引擎memory
    文件存储系统特点：也称HEAP存储引擎，所以数据保存在内存中。
    功能特点：
        · 支持HASH索引和BTree索引（等值查询HASH索引会非常快，范围索引较慢       范围查找适用BTree索引）
        · 所有字段都为固定长度varchar（10）=char（10）
        · 不支持BLOG和TEXT等大字段
        · Memory存储引擎使用表级锁
        · 最大大小由max_heap_table_size参数决定
    
    19、MySQL常用存储引擎之Federated
    特点：
        · 提供了访问远程Mysql服务器上表的方法
        · 本地不存储数据，数据全部放到远程服务器上
        · 本地需要保存表结构和远程服务器的连接信息
    如何使用：
        默认禁止，启动需要再启动的时候增加federated参数（mysql://user_name[:password]@host_name[:port_num]/db_name/tbl_name） 
        具体操作：
            show engines;
            cd /usr/local/mysql
            vim /my.cnf（在sql_mode=NO_ENGINNE_SUBSTITUTION,STRICT_TRANS_TABLES下加上fedreated=1）
            mysqladmin shutdown
            ps -ef
            bin/mysqld_safe --defaults-file=./my.cnf --user=mysql &[1] 5053
            grand select,update,insert,delete on 数据库名称 to 简称@'127.0.0.1' identified by '123456';
    适用场景：偶尔的统计分析及手工查询
    
    20、如何选择存储引擎
    参考条件：事务      备份      崩溃恢复      存储引擎的特有特性（不要混合使用存储引擎）
    
    21、MySQL服务器参数介绍
    MySQL获取配置信息路径
        ·命令行参数（mysqld_safe --datadir=/data/sql_data）
        ·配置文件（/etc/my.cnf）（查询命令：mysqld -help -verbose |grep -A 1 'Default options'）
    MySQL配置参数的作用域
        ·全局参数（set global 参数名=参数值;set @@global.参数名:=参数值;）
        ·会话参数（set [session] 参数名=参数值; set@@session.参数名:=参数值];）
        set global wait _timeout=3600;set global interactive_name=3600;（一起更改才有效）
    
    22、内存配置相关参数
    内存配置相关参数：
        · 确定可以使用的内存的上限
        · 确定MySQL的每个连接使用的内存（sort_buffer_size（排序缓冲区的尺寸），join_buffer_size（连接缓冲区的尺寸），  
        read_buffer_size（读缓冲区的大小   一定要是4k的倍数），read_rnd_buffer_size（索引缓冲区的大小））为每个线程所分配的，100个用户同时访问会占用100倍的内存
        · 确定需要为操作系统保留多少内存
        · 如何为缓存池分配内存（Innodb_buffer_pool_size（定义了Innodb所使用的缓冲池的大小      总内存-（
        每个线程所需要的内存*连接数）-系统保留内存），key_buffer_size（MyISAM所使用的缓冲池的大小     select
         sum(index_length) from information_schema.tables where engine='myisam'））
    
    23、I/O相关参数配置
    I/O相关参数配置：
        Innodb I/O相关参数：
            · Innodb_log_file_size
            · Innodb_log_files_in_group
            · 事务日志总大小=Innodb_log_files_in_group*Innodb_log_file_size
            · Innodb_log_buffer_size
            · Innodb_flush_log_at_trx_commit（0：每秒进行一次log写入cache，并flush log到磁盘  
            1[默认]：在每次事务提交执行log写入cache，并flush log到磁盘      2[建议]：每次事务提交，执行log数据写入到cache，每秒执行一次flush log到磁盘）
            · Innodb_flush_method=O_DIRECT
            · Innodb_file_per_table = 1
            · Innodb_doublewrite = 1
        MyISAM I/O相关配置：delay_key_write（OFF：每次写操作后刷新键缓冲中的脏块到磁盘      ON：只对在键表时制定了delay_key_write选项的表示用延迟刷新      ALL：对所有MYISAM表都使用延迟键写入）
    
    24、安全项配置参数
    安全相关配置参数：
        · expire_logs_days 指定自动清理binlog的天数（7天）
        · max_allowed_packet 控制MySQL可以接受的包的大小（主从一致）
        · skip_name_resolve 禁用DNS查找
        · sysdate_is_now 确保sysdate()返回确定性日起
        · read_only 禁止非super权限的用户写操作（从服务器启用这个配置参数）
        · skip_slave_start 禁用Slave自动恢复
        · sql_model 设置MySQL所使用的SQL模式（strice_trans_tables，no_engine_subtitution，  
        no_zero_date，no_zero_in_date，only_full_group_by）
    
    25、其他常用参数配置
    其他常用参数配置：
        · sync_binlog 控制MySQL如何向磁盘刷新binlog（设置为1）
        · tmp_table_size和max_heap_table_size 控制内存临时表大小
        · max_connections 控制允许的最大连接数（通常情况下设置为2000）
    
    26、数据库结构设计和优化
    数据库设计对性能的影响：
        · 过分的反范式化为表建立太多的列
        · 过分的范式化造成太多的表关联（MySQL限制最多关联61个表）
        · 在OLTP环境中使用不恰当的分区表
        · 使用外键保证数据的完整性
    
    27、总结
    性能优化顺序：
        · 数据库结构设计和SQL语句
        · 数据库存储引擎的选择和参数配置
        · 系统优化
        · 硬件升级
                             
### 三、MySQL基准测试
   
    1、什么是基准测试
    定义：基准测试是一种测量和评估软件性能指标的活动用于建立某个时刻的性能基准，以便当系统发生软硬件变化时重新进行基准测试以评估变化对性能的影响
    基准测试是针对系统设置的一种压力测试
    基准测试：直接、简单、已于比较，用于评估服务器的处理能力
    压力测试：对真实的业务数据进行测试，获得真实系统所能承受的压力
    压力测试需要针对不同主题，所使用的数据和查询也是真实用到的
    基准测试可能不关心业务逻辑，所使用的查询和业务的真实性可以和业务环境没有关系
    
    2、如何进行基准测试
    基准测试的目的：
        · 建立MySQL服务器的性能基准线（确定当前MySQL服务器运行情况）
        · 模拟比当前系统更高的负载，以找出系统的扩展瓶颈（增加数据库并发，观察QPS，PTS变化，确定并发量与性能最优的关系）
        · 测试不同的硬件、软件和操作系统配置
        · 证明新的硬件设备是否配置正确
        · 对整个系统进行基准测试
    优点：
        · 能够测试整个系统的性能，包括web服务器、数据库等
        · 能反映出系统中各个组件接口间的性能问题体现真实性能状况
    缺点：
        · 测试设计复杂，消耗时间长
        · 单独对MySQL进行基准测试
    优点：测试设计简单，所需耗费时间短
    缺点：无法全面了解整个系统的性能极限
    MySQL基准测试的常见指标：
        · 单位时间的所处理的事务数（TPS）
        · 单位时间内所处理的查询数（QPS）
        · 响应时间（平均响应时间、最小响应时间、最大响应时间、各时间所占百分比）
        · 并发量：同时处理的查询请求的数量（并罚两不等于连接数）
    
    3、基准测试演示实例
    基准测试的步骤：
        · 计划和设计基准测试（对整个系统还是某一组件）
        · 使用什么样的数据
    计划和设计基准测试：准备基准测试及数据收集脚本（CPU使用率、IO、网络流量、状态与计数器信息等           
    Get_Test_info.sh ）
    
    vim Get_Test_info.sh
    #!/bin/bash
    INTERVAL=5      （每隔多长时间收集一次动态信息）
    PREFIX=/home/imooc/benchmarks/$INTERVAL-sec-status      （动态信息记录的位置）
    RUNFILE=/home/imooc/benchmarks/running      （指定运行标识）
    echo "1" > $RUNFILE      （生成运行标识文件）
    MYSQL=/usr/local/mysql/bin/mysql      （MySQL所在位置）
    $MYSQL -e "show global variables" >> mysql-variables      （记录当前系统MySQL的设置信息）
    while test -e $RUNFILE; do      （只要标识文件有的情况下就会一直循环下去）
    file=$(date +%F_%I)
    sleep=$(date +%s.%N | awk '{print 5 - ($1 % 5)}')
    sleep $sleep
    ts="$(date +"TS %s.%N %F %T")"
    loadavg="$(uptime)"      （收集系统的负载情况）
    echo "$ts $loadavg" >> $PREFIX-${file}-status
    $MYSQL -e "show global status" >> $PREFIX-${file}-status &      （收集MySQL的状态信息）
    echo "$ts $loadavg" >> $PREFIX-${file}-innodbstatus      （收集Innodb的状态信息）
    $MYSQL -e "show engine innodb status" >> $PREFIX-${file}-innodbstatus &
    echo "$ts $loadavg" >> $PREFIX-${file}-processlist
    $MYSQL -e "show full processlist\G" >> $PREFIX-${file}-processlist &      （MySQL建成的情况）
    echo $ts
    done
    echo Exiting because $RUNFILE does not exists
    运行基准测试
    保存及分析基准测试结果（analyze.sh）
    vim analyze.sh
    #!/bin/bash
    awk '
      BEGIN {
        printf "#ts date time load QPS";
        fmt=" %.2f";
      }
      /^TS/ {
      ts = substr($2,1,index($2,".")-1);
      load = NF -2;
      diff = ts - prev_ts;
      printf "\n%s %s %s %s",ts,$3,$4,substr($load,1,length($load)-1);
      prev_ts=ts;
      }
      /Queries/{
      printf fmt,($2-Queries)/diff;
      Queries=$2
      }
      ' "$@"
    基准测试中容易忽略的问题：
        · 使用生产环境数据时只是用了部分数据（推荐：使用数据库完全备份来测试）
        · 在多用户场景中，只做单用户的测试（推荐：使用多线程并发测试）
        · 在单服务器上测试分布式应用（推荐使用相同的架构进行测试）
        · 反复执行同一查询（容易缓存命中，无法反映真实查询性能）
    
    4、MySQL基准测试工具mysqlslap
    常用的基准测试工具介绍：
        · MySQL基准测试工具 mysqlslap
        · 下载及安装：MySQL服务器自带的基准测试工具，随MySQL一起安装
    特点：
        · 可以模拟服务器负载，并输出相关统计信息
        · 可以制定也可以自动生成查询语句
         常用参数说明：
            --auto-generate-sql 由系统自动生成SQL脚本进行测试
            --auto-generate-sql-add-autoincrement 在生成的表中增加自增ID
            --auto-generate-sql-load-type 指定测试中使用的查询类型（读写、删除、更新）
            --auto-generate-sql-write-number 指定初始化数据时生成的数据量
            --concurrency 指定并发线程的数量
            --engine 指定要测试表的存储引擎，可以用逗号分隔多个存储引擎
            --no-drop 指定不清理测试数据
            --iterations 指定测试运行的次数
            --number-of-queries 指定每一个线程执行的查询数量
            --debug-info 指定输出额外的内存及CPU统计信息
            --number-int-cols 指定测试表中包含的INT类型列的数量
            --number-char-cols 指定测试表中包含的varchar类型的数量
            --create-schema 制定了用于执行测试的数据库的名字
            --query 用于指定自定义SQL的脚本
            --only-print 并不运行测试脚本，而是把生成的脚本打印出来
    mysqlslap --concurrency=1,50,100,200 --iterations=3 --numbei-init-cols=5 
        -number-char-cols=5 -auto-generate-sql -auto-generate-sql-add-autoincrement 
        --enginer=mysiam,innodb -number-of-queres=10 --create-schema=sbtest
    
    5、MySQL基准测试工具sysbench
    常用的基准测试工具介绍：MySQL基准测试工具 sysbench
        安装说明：https://github.com/akopytov/sysbench/archive/0.5.zip
            unzip sysbench-0.5.zip 
            cd sysbench
            ./autogen.sh
            ./configure --with-mysql-includes=/usr/local/mysql/include/  --with-mysql-libs=/usr/local/mysql/lib
            make && make install
        常用参数：--test 用于指定所要执行的测试类型，支持以下参数（Fileio 文件系统I/O性能测试、cpu cpu性能测试、memory 内存性能测试、Oltp 测试要指定具体的lua脚本、Lua脚本位于 sysbench-0.5/sysbench/tests/db）
            --mysql-db 用于指定执行基准测试的数据库名
            --mysql-table-engine 用于指定所使用的存储引擎
            --oltp-tables-count 执行测试的表的数量
            --oltp-table-size 指定每个表中的数据行数
            --num-threads 指定测试的并发线程数量
            --max-time 指定最大的测试时间
            --report-interval 指定间隔多长时间输出一次统计信息
            --mysql-user 指定执行测试的MySQL用户
            --mysql-password 指定执行测试的MySQL用户的密码
            prepare 用于准备测试数据
            run 用户实际进行测试
            cleanup 用于清理测试数据
    
    6、sysbench基准测试演示实例
    sysbench --test=cpu --cpu-max-prime=10000 run
    sysbench --test=fileio --file-total-size=1G prepare
    sysbench --test=fileio --num-threads=8 --init-rng=on --file-total-size=1G --file-test-mode=rndrw --report-interval=1 run
    create database test;
    grant all privileges on *.* to sbtest@'localhost' identified by '123456';
    cd sysbench-0.5/sysbench/tests/db
    ls -l *lua
    sysbench --test=./oltp.lua --mysql-table-engine=innodb --oltp-table-size=10000 --mysql-db=test --mysql-user=sbtest --mysql-password=123456 --oltp-tables-count=10 --mysql-socket=/usr/local/mysql/data/mysql.sock prepare
    ./analyze.sh /home/test/benchmarks/5-sec-status-2016-03-05 11-status
    
### 四、MySQL数据库结构优化
    1、数据库结构优化介绍
    数据库结构优化的目的：减少数据冗余
    尽量避免数据维护中出现更新，添加和删除异常
    添加异常：如果表中的某个实体随着另一个实体共有
    更新异常：如果更改表中的某个实体的单独属性时，需要对多行进行更新
    删除异常：如果删除表中的某一个实体则会导致其他实体的消失
    节约数据存储空间
    提高查询效率

    2、数据库设计的步骤
        需求分析：全面了解产品设计的存储需求
        存储需求
        数据处理需求
        数据的安全性和完整性
        逻辑设计：设计数据的逻辑存储结构
        数据实体之间的逻辑关系，解决数据冗余和数据维护异常
        物理设计：根据所使用的数据库特点进行表结构设计（关系型数据库：Oralce,SQLServer,MySQL,postgresSQL      非关系型数据库：mongo,Redis,Hadoop      存储引擎：Innodb）
        维护优化：根据实际情况对索引、存储结构等进行优化
        数据库设计范式
        数据库设计的第一范式：数据库表中的所有字段都只具有单一属性
        单一属性的列是由基本的数据类型所构成的
        设计出来的表都是简单的二维表
        数据库设计的第二范式：要求一个表中只具有一个业务主键，也就是说符合第二范式的表中不能有非主键列对只对部分之间的依赖关系
        数据库设计的第三范式：指每一个非属性既不部分依赖于也不传递依赖于业务主键，也就是在第二范式的基础上消除了非主属性对主键的传递依赖

    3、需求分析及逻辑设计
        需求说明：只销售图书类商品
        需要有以下功能（用户登录、用户管理、商品展示、商品管理、供应商管理、在线销售）
        需求分析及逻辑设计
        用户登录及用户管理功能：用户必须注册并登录系统才能进行网上交易（用户名来作为用户信息的业务主键）
        同一时间一个用户只能在一个地方登陆
        用户信息（用户名（主键），密码，手机号，姓名，注册日期，在线状态，出生日期）
        商品展示及商品功能管理：商品信息（商品名称，分类名称，出版社名称，图书价格，图书描述，作者）、
        （商品信息（商品名称（主键），出版社名称，图书价格，图书描述，作者）、
        分类信息（分类名称（主键），分类描述））
        商品分类（对应关系表）（商品名称（主键），分类名称（主键））
        供应商信息（出版社名称，地址，电话，联系人，银行账号）
        在线销售功能：在线销售（订单编号（主键），下单用户名，订单日期，订单金额，订单商品分类，订单商品名，订单商品单价，订单商品数量，支付金额，物流单号）
        （订单表（订单编号，下单用户名，下单日起，支付金额，物流单号）、
        订单商品关联表（订单编号，订单商品分类，订单商品名，商品数量））
        编写SQL查寻出每一个用户的订单总金额：select 下单用户名,sum(d.商品价格*b.商品数量) from 订单表 a join 订单商品关联表 b on a.订单编号=b.订单编号 join 商品分类关联表 c on c.商品名称=b.商品名称 and c.分类名称=b.订单商品分类 join 商品信息表 d on d.商品名称=c.商品名称 group by下单用户名（关联的表越多性能也就越差）
        编写SQL查寻出下单用户和订单详情：select a.订单编号,e.手机号md,商品名称,c.商品数量,d.商品价格 from 订单表 a join 订单商品关联表 b on a.订单编号=b.订单编号 join 商品分类关联表 c on c.商品名称=b.商品名称 and c.分类名称=b.订单商品分类 join 商品信息表 d on d.商品名称=c.商品名称 join 用户信息表 e on e.用户名=a.下单用户名（完全符合范式化的设计有时并不能得到良好的SQL查询性能）

    4、需求分析及逻辑设计
        什么叫做反范式化设计：反范式化是针对范式化而言的，在前面介绍了数据库设计的范式，作为的反范式化就是为了性能和读取效率的考虑而适当的对数据库设计范式的要求进行违反，而允许存在少量的数据冗余，换句话来说反范式化就是用空间来换取时间
        图书在线销售网站数据库的反范式化改造：（商品信息（商品分类，分类名称，出版社名称，图书价格，图书描述，作者）、分类信息（分类名称，分类描述））
        （订单表（订单编号，下单用户名，手机号，下单日起，支付金额，物流单号，订单金额）、
        订单商品关联表（订单编号，订单商品分类，订单商品名，商品数量，商品单价））
        反范式化改造后的查询
        编写SQL查询出每一个用户的订单总金额：select 下单用户名,sum(订单金额) from 订单表 group by 下单用户名;
        编写SQL查询出下单用户和订单详情：select a.订单编号,a.用户名,a.手机号,商品名称,b.商品单价,b.商品数量 from 订单表 a join 订单商品关联表 b on a.订单编号=b.订单编号
        不能完全按照范式化的要求进行设计
        考虑以后如何使用表

    5、范式化设计和反范式化设计优缺点
    范式化设计的优缺点
        优点：可以尽量的减少数据冗余
            范式化的更新操作比反范式化更快
            范式化的表通常比反范式化更小
        缺点：对于查询需要对多个表进行关联
            更难进行索引优化
            反范式化设计的优缺点
        优点：可以减少表的关联
            可以更好地进行索引优化
        缺点：有数据冗余及数据维护异常
            对数据的修改需要更多的成本

    6、物理设计介绍
    物理设计：根据所选择的关系型数据库的特点对逻辑模型进行存储结构设计
    物理设计涉及的内容：定义数据库、表及字段的命名规范（数据库、表及字段的命名要遵守可读性原则，数据库、表及字段的命名要遵守表意性原则，数据库、表及字段的命名要遵守长名原则）
    选择合适的存储引擎、为表中的字段选择合适的数据类型建立数据库结构

    7、物理设计-数据类型的选择
        物理设计：为表中的字段选择合适的数据类型（当一个列可以选择多种数据类型时，应该优先考虑数字类型，其次是日期或二进制类型，最后是字符类型。对于相同级别的数据类型，应该优先选择占用空间小的数据类型）
        如何选择正确的整数类型
        
        如何选择正确的实数类型
        如何选择VARCHAR和CHAR类型
            · VARCHAR类型的存储特点（varchar用于存储变长字符串，只占用必要的存储空间、列的最大长度小于255则只占用一个额外字节用于记录字符串长度、列的最大长度大于255则要占用额外两个字节用于记录字符串长度）
            · VARCHAR长度的选择问题（使用最小的符合要求的长度）
            · varchar(5)和varchar(200)存储'MySQL'字符串性能不同
            · VARCHAR的适用场景（字符串列的最大长度比平均长度大很多，字符串列很少被更新，使用了多字节字符集存储字符串）
            · CHAR类型的存储特点（CHAR类型是定长的（50字符），字符串存储在CHAR类型的列中会删除末尾的空格，CHAR类型的最大宽度为255）
            · CHAR类型的适用场景（CHAR类型适合存储长度近似的值，CHAR类型适合存储短字符串，CHAR类型适合存储经常更新的字符串列）

    8、物理设计-如何存储日期类型
    如何存储如期数据：DATETIME类型（以YYYY-MM-DD HH:MM:SS[.fraction]格式存储日期时间 datatime = YYYY-MM-DD HH:MM:SS datetime(6) = YYYY-MM-DD HH:MM:SS.fraction 、DATETIME类型与时区无关，占用8个字节的存储空间、时间范围1000-01-01 00:00:00到9999-12-31 23:59:59）
    TIMESTAMP类型（存储了由格林尼治时间1970年1月1日到当前时间的秒数 以YYYY-MM-DD HH:MM:SS.[.fraction]的格式显示，占用4个字节、时间范围1990-01-01 到2038-01-19、timestamp类型显示依赖于所指定的时区、在行的数据修改时可以自动修改timestamp列的值）
    date类型和time类型（存储用户生日 一是把日期部分存储为字符串（至少要8个字符），二是使用int类型来存储（4个字节），三是使用datetime类型来存储（8个字节））
    date类型的优点：1.占用的字节数比使用字符串、datetime、int存储要少，使用date类型只需要3个字节
    2.使用Date类型还可以利用日期时间函数进行日期之间的计算
    date类型用于保存1000-01-01到9999-12-31之间的日期
    time类型用于存储时间数据，格式为HH:MM:SS
    存储日期时间数据的注意事项：不要使用字符串类型来存储日期时间数据（日期时间类型通常比字符串占用的存储空间小、日期时间类型在进行查找过滤时可以利用日期来进行对比、日期时间类型还有着丰富的处理函数，可以方便的对时期类型进行日期计算）
    使用Int存储日期时间不如使用Timestamp类型

    9、物理设计-总结
    innodb：为表中的每个列选择合适的类型
        如何选择表的主键（主键应该尽可能的小，主键应该是顺序增长的（增加数据的添加效率），Innodb的主键和业务主键可以不同）
        
        
        
### 五、MySQL高可用架构设计
    1、mysql复制功能介绍
        · MySQL复制功能提供分担读负载
        · 复制解决了什么问题：实现在不同服务器上的数据分布  （利用二进制日志增量进行  不需要太多的带宽  但是使用基于行的复制在进行大批量的更改时  会对带宽带来一定的压力  特别是跨IDC环境下进行复制  应该分批进行）
        · 实现数据读取的负载均衡（需要其他组件配合完成  利用DNS轮询的方式把程序的读连接到不同的备份数据库  利用LVS，haproxy这样的代理方式  非共享架构，同样的数据分布在多台服务器上）
        · 增加了数据安全性（利用备库的备份来减少主库负载  复制并不能代替备份  方便进行数据库高可用架构的部署  避免MySQL单点失败）
        · 实现数据库高可用和故障切换
        · 实现数据库在线升级
    
    2、mysql二进制日志
    MySQL服务层日志：二进制日志（记录了所有对MySQL数据库的修改事件包括增删改查事件和对表结构的修改事件）      慢查日志      通用日志
    MySQL存储引擎层日志：innodb（重做日志      回滚日志）
    二进制日志的格式：基于段的格式 binlog_format=STATEMENT（优点：日志记录量相对较小，节约磁盘及网络I/O 只对一条记录修改或者插入row格式所产生的日志量小于段产生的日志量      缺点：必须要记录上下文信息保证语句在从服务器上执行结果和在主服务器上相同 特定函数如UUID(),user()这样非确定性函数还是无法复制      可能造成MySQL复制的主备服务器数据不一致）
    show variables like 'binlog_format';
    set session binlog_format=statement;
    show binary logs; （显示二进制日志）
    flush logs;
    cd /home/mysql/sql_log
    mysqlbinlog 日志名称 （查看二进制日志文件内容）
                                   基于行的日志格式 binlog_format=ROW（Row格式可以避免MySQL复制中出现的主从不一致的问题 同一SQL语句修改了10000调数据的情况下 基于段的日志格式只会记录这个SQL语句 基于行的日志会有10000条记录分别记录每一行的数据修改      优点：使用MySQL主从复制更加安全 对与每一行数据的修改比基于断的复制高效      缺点：记录日至量较大 binlog_row_image = [FULL|MINIMAL|NOBLOB]）      误操作而修改了数据库中的数据，同时又没有备份可以恢复时，我们就可以通过分析二进制日志，对日志中的记录的数据修改操作反向处理的方式来达到恢复数据的目的
    show variables like 'binlog_row_image';
    mysqlbinlog -vv 日志名称 （直接查看二进制文件中的SQL记录）
                                   混合日志格式 binlog_format = MIXED（特点：根据SQL语句由系统决定在基于段和基于行的日志格式中进行选择 数据量的大小由所执行的SQL语句决定）
    如何选择二进制日志的格式：binlog_format=mixer      or      binlog_format=row binlog_row_image=minimal
    
    3、mysql二进制日志格式对复制的影响
    基于SQL语句的复制（SBR）二进制日志格式使用的是statement格式
    基于行的复制（RBR）二进制日志格式使用的是基于行的日志格式
    混个模式      根据实际内容在以上两者间切换
    基于SQL语句的复制（SBR）
    优点：
        · 生成的日志量少，节约网络转输I/O
        · 并不强制要求主从数据库的表完全相同
        · 相比于基于行的复制方式更为灵活
    缺点：
        · 对于非确定性事件，无法保证主从复制数据的一致性
        · 对于存储过程，触发器，自定义函数进行的修改也可能造成数据不一致
        · 相比于基于行的复制方式在从上执行时需要更多的行锁
    基于行的复制（RBR）
    优点：
        · 可以应用于任何SQL的复制包括非确定函数，存储过程等
        · 可以减少数据库锁的使用
    缺点：
        · 要求主从数据库的表结构相同，否则肯能会中断复制
        · 无法在从上单独执行触发器
    
    4、mysql复制工作方式
    主服务器将变更写入二进制日志
    从服务器读取主服务器的二进制日至变更并写入到relay_log中（基于日志点的复制 基于GTID的复制）
    在从服务器上重放relay_log中的日志（基于SQL段的日志是在从库上重新执行记录的SQL 基于行的日志则是在从库上直接应用对数据库行的修改）
    
    5、基于日志点的复制配置步骤
    在主DB服务器上建立复制账号（CREATE USER 'repl'@'IP段' identified by 'PassWord';GRANT REPLICATION SLAVE ON *.* TO 'repl'@'IP段';）
    配置主数据库服务器（bin_log（log_bin） = 目录路径/mysql-bin      server_id = 100）
    配置从数据库服务器（bin_log（log_bin） = 目录路径/mysql-bin      server_id = 101      relay_log = mysql-relay-bin      log_slave_update = on[可选]      read_only = on[可选])
                                      scp all.sql root@ip段:/root
    初始化从服务器数据（mysqldump --master-data=2 -single-transaction[ --triggers --routines --all-databases >> all.sql]      xtrabackup  --slave-info）
    启动复制连路（CHANGE MASTER TO MASTER_HOST='master_host_ip',MASTER_USER='repl',MASTER_PASSWORD='PassWord',MASTER_LOG_FILE='mysql_log_file_name',MASTER_LOG_POS=4;）
    优点：
        · 是MySQL最早支持的复制技术，Bug相对较少
        · 对SQL查询没有任何限制
        · 故障处理比较容易
    缺点：故障转移时重新获取新主服务器的日志点信息比较困难
    
    5、基于GTID的复制
    基于 GTID复制的优缺点
    什么是GTID：GTID即全局事务ID，其保证为每一个在主服务器上提交的事务在复制集群中可以生成一个唯一的ID
               GTID=source_id:transaction_id
    在主DB服务器上建立复制账号（CREATE USER 'relp'@'IP段' identified by 'PassWord'; GRANT REPLICATION SLAVE ON *.* TO 'relp'@'IP段';）
    配置主数据库服务器（bin_log（log_bin） = /usr/local/mysql/log/mysql-bin      server_id = 100      gtid_mode = on      enforce-gtid-consistency =on（强制GTID一致性）create table ... select 在事务中使用Create temporary table 建立临时表使用关联更新事务表和菲事务表都会报错      log-slave-updates = on）
    配置从数据库服务器（bin_log（log_bin） = /usr/local/mysql/log/mysql-bin      server_id = 101      relay_log = /usr/local/mysql/log/relay_log      gtid_mode = on      enforce-gtid-consistency = on      log-slave-updates = on      read_only=on[建议]      master_info_repository = TABLE      relay_log_info_repository = TABLE）
    初始化从服务器数据（mysqldump --master-data=2 -single-transaction[ -triggers --rotines --all-databases -uroot -p > all.sql]      xtrabackup --slave-info）
    启动基于GTID的复制（CHANGE MASTER TO MASTER_HOST='master_host_ip',MATER_USER='relp',MASTER_PASSOWORD='PassWord',MASTER_AUTO_POSITION=1）
    show slave status\G
    基于GTID复制的优缺点：
        · 优点（可以很方便的进行故障转移      从库不会丢失主库上的任何内容）
        · 缺点（故障处理比较复杂      对执行的SQL有一定的限制）
    选择复制模式要考虑的问题：
        · 所使用的MySQL版本
        · 复制架构及主从切换的方式
        · 所使用的高可用管理组件
        · 对应用的支持程度
    
    7、MySQL复制拓扑
    一主多从的复制拓扑
    优点：
        · 配置简单
        · 可以用多个从库分担读负载
    用途：
        · 为不同的业务使用不同的从库
        · 将一台从库放到远程IDC，用作灾备恢复
        · 分担主库的读负载
    主-主复制拓扑
    主主模式下的主-主复制的配置注意事项：
        · （产生数据冲突而造成复制链路的中断      耗费大量的时间      很容易造成数据的丢失）
        · 两个主中所操作的表最好能分开
        · 使用下面两个参数进行自增ID的生成（auto_increment_increment=2      auto_increment_offset=1|2）
    主备模式下的主-主复制的配置注意事项：
        · （只有一台主服务器对外提供服务区 一台服务器处于只读状态并且只作为热备使用 在对外提供服务的主库出现故障或是计划性的维护时  
        才会进行切 换使原来的备库成为主库，而原来的主库会成为新的备库并处于只读或是下线状态，待维护完成后重新上线）
        · 确保两台服务器上的初始数据相同
        · 确保两台服务器上已经启动binlog并且有不同的server_id
        · 在两台服务器上启用的log_slave_updates参数
        · 在初始的备库上启用read_only
    拥有备库的主-主复制拓扑
    级联复制
    
    8、MySQL复制性能优化
    影响主从延迟的因素：
        · 主库写入二进制日志的时间（控制主库的事务大小，分割大事务）
        · 二进制日志传输时间（使用MIXED日志格或设置set binlog_row_image=minimal）
        · 默认情况下从只有一个SQL线程，主上并发的修改在从上变成了串行（使用多线程复制）
    如何配置多线程复制：
        stop slave
        set global slave_parallel_type='logical_clock'
        set global slave_parallel_workers=4;
        start slave;
        show (full) processlist\G（查看复制线程）

    9、MySQL复制常见问题处理
    由于数据损坏或丢失所引起的主从复制错误：主库或从库意外宕机引起的错误（使用跳过二进制日志事件 注入空事务的方式先恢复中断的复制链路 在使用其他方法来对比主从服务器上的数据）
    主库上的二进制日志损坏（通过change master命令来重新指定）
    备库上的中继日志损坏
    在从库上进行数据修改造成的主从复制错误：在从库上进行数据修改造成的主从复制错误
    在多个从服务器上有不唯一的server_id或server_uuid（server_uuid是记录在数据目录中的auto.cnf文件中）
    max_allow_packet设置引起的主从复制错误
    MySQL复制无法解决的问题：
        · 分担主数据库的写负载
        · 自动进行故障转移和主从切换
        · 提供读写分离功能
    
    10、什么是高可用架构
    什么是高可用：高可用性H.A.（High Availability）指的是通过尽量缩短因日常维护操作（计划）和突发的系统崩溃（非计划）所导致的停  
    机时间，以提高系统和应用的可用性。
    （365*24*60）*（1-0.99999） = 5.256
    如何实现高可用：
        · 避免导致系统不可用的因素，减少系统不可用的时间（服务器磁盘空间耗尽（备份或者各种查询日志突增导致的磁盘空间被占满。MySQL  
        由于无法记录二进制日志，无法处理新的请求而产生的系统不可用的故障） 性能糟糕的SQL 表结构和索引没有优化 主从数据不一致 人为  
        的操作失误等等）
        · 建立完善的监控及报警系统
        · 对备份数据进行恢复测试
        · 正确配置数据库环境
        · 对于不需要的数据进行归档和清理
        · 增加系统冗余，保证发生系统不可用是可以尽快恢复
        · 避免有单点故障
        · 主从切换及故障转移
    如何解决MySQL单点故障
        单点故障：单点故障是指在一个系统中提供相同功能的组件只有一个，如果这个组件消失了，就会影响整个系统功能的正常使用。组成应用系统的各个组件都有可能成为单点。
        利用SUN共享存储或DRDB磁盘复制解决MySQL单点故障
        利用多写集群或NDB集群来解决MySQL单点故障（如果内存不足NDB性能就会非常差）
        利用MySQL主从复制来解决MySQL单点故障
    如何解决主服务器的单点问题
        主服务器切换后 如何通知应用新的主服务器的IP地址
        如何检查MYSQL主服务器是否可用
        如何处理从服务器和新主服务器之间的那种复制关系
    
    11、MMM架构介绍
    MMM的主要作用：监控和管理MySQL的主主复制拓扑，并在当前的主服务器失效时，进行主和主备服务器之间的主从切换和故障转移等工作。
    MMM提供了什么功能：MMM监控MySQL主从复制健康状况
        在主库出现宕机时进行故障转移并自动配置其他从对新主的复制
        提供了主，写虚拟IP，在主从服务器出现问题时可以自动迁移虚拟IP
    MMM部署所需要资源
    名称资源            数量               说明
    主DB服务器           2           用于主备模式的主主复制配置
    从DB服务器          0-N          可以配置0太或多台从服务器
    监控服务器           1           用于监控MySQL复制集群
    Ip地址             2*(n+1)       n为MySQL服务器数量
    监控用户             1           用户监控数据库状态的MySQL用户（repliacation client）
    代理用户             1           用户MMM代理的MySQL用户（super,replication client,process）
    复制用户             1           用户配置MySQL复制的MySQL用户（replication slave）
    
    12、MMM架构实例演示
    MMM部署步骤：
        · 配置主主复制及主从同步集群
        · 安装从节点所需要的支持包（Perl）
        · 安装及配置MMM工具集
        · 运行MMM监控服务
        · 测试
    建立账号
    IP      mysqldump --sigle-transaction --master-data=2  --alldatabases -uroot -p > all.sql（备份数据库）
            scp all.sql IP1:/root
            scp all.sql IP3:/root
            more all.sql（查看备份的二进制日志名字和偏移量）
            change master to master_host='IP1',master_password='repl',master_log_file='IP1的master_log_file文件名',master_log_pos='IP1的master_log_pos的值'；
            start slave;
            show slave status \G
    IP1     mysql -urrot -p < all.sql
            change master to master_host='IP',master_user='repl',master_password='123456',MASTER_LOG_FILE='IP的master_log_file',MASTER_LOG_POS=IP的master_log_pos;
            show slave status \G（查看master_log_file和master_lof_pos）
    IP3     mysql -uroot -p < all.sql
            change master to master_host='IP',master_user='repl',master_password='123456',MASTER_LOG_FILE='IP的master_log_file',MASTER_LOG_POS=IP的master_log_pos;
            show slave status \G
    
    13、MMM架构实例演示
    在MMM架构所有服务器都要安装
    cd /home
    mkdir tools
    
    cd tools
    wget http://mirrors.opencas.cn/epel/epel-release-latest-6.noarch.rpm
    wget  http://rpms.famillecollet.com/enterprise/remi-releas-6.rpm
    rpm -ivh epl-release-laster-6.noarch.rpm
    rpm -ivh remi-release-6.rpm
    vi /etc/yum.repos.d/remi.repo      enable=1
    vi /etc/yum.repos.d/epel.repo      #baseurl-> baseurl      #mirrorlist->mirrorlist
    yum search mmm
    yum install mysql-mmm-agent.noarch （三台机器都要执行）
    yum install mysql-mmm* （监控服务器执行）
    IP      mysql -uroot -p
            grant replication client on *.* to 'mmm_monitor'@'IP' identified by '123456';（创建监控用户）
            grant super.replication client.process on *.* to 'mmm_agent'@'IP' identified by '123456';（创建代理用户）
            quit
            cd /etc/mysql-mmm
            vi /mmm_common.conf      （replication_user repl      replication_password 123456      agent_user mmm_agent      agetn_password 123456      ip IP1     ip IP      ip IP3      role write ips 虚拟IP6      role read host db1,db2,db3 ips 虚拟IP7，虚拟IP9，虚拟IP10）
            scp mmm_common.conf root@IP1/etc/mysql-mmm/
            scp mmm_common.conf root@IP3/etc/mysql-mmm/
            vi mmm_agent.conf     （this db1）
    IP1     vi mmm_agent.conf      （this db2）
    IP3     vi mmm_agent.conf      （this db3）
            vi mmm_mon.conf      （ping_ips IP,IP1,IP3      monitor_user mmm_moitor     monitor_password 123456）
            /etc/init.d/mysql-mmm-agent start
    IP      /etc/init.d/mysqk-mmm-agent start
    IP1     /etc/init.d/mysql-mmm-agent start
    IP3     /etc/init.d/mysql-mmm-monitor start
            mmm_control show（查看三台服务器状态）
    IP      ip addr（查看虚拟IP是否配置成功）
            /etc/init.d/mysqld stop
    
    14、MMM架构的优缺点
    MMM工具的优点：
        · 使用Perl脚本语言完全及完全开源
        · 提供了读写VIP（虚拟IP），使服务器角色的变更对应用透明（在从服务器出现大量的主从延迟，主从链路中断时可以把这台从服务器上  
        的读的虚拟IP，漂移到集群中其他正常的服务器上）
        · MMM提供公了从服务器的延迟监控
        · MMM提供了主数据库故障转移后从服务器对新主的重新同步功能
        · 很容易对发生故障的主数据库重新上线
        · MMM提供了从服务器的延迟监控
    MMM工具的缺点：
        · 发布时间比较早不支持MySQL新的复制功能（基于gtid的复制可以保证日至不会重复在Slave服务器上被执行 对于MySQL5.6后所提供的  
        多线程复制技术也不支持）
        · 没有读负载均衡的功能
        · 在进行主从切换时，容易造成数据丢失
        · MMM监控服务有单点故障
    
    15、MHA架构介绍
    MHA完成主从切换超高效（保证数据的一致性）
    MHA提供了什么功能：
        监控主数据库服务器是否可用
        当主DB不可用时，对多个从服务器中选举出新的主数据库服务器
        提供了主从切换和故障转移功能（MHA可以与半同步复制结合）
    MHA是如何进行主从切换的：
        尝试从出现故障的主数据库保存二进制日志
        从多个备选从服务器中选举出新的备选主服务器
        在备选主服务器和其他从服务器之间同步差异二进制数据
        应用从原主DB服务器上保存的二进制日志（重复的主键等会使MHA停止进行故障转移）
        提升备选主DB服务器为新的主DB服务器
        迁移集群中的其它从DB作为新的主DB的从服务器
    MHA配置步骤：
        配置集群内所有主机的SSH免认证登陆（比如故障转移过程中保存原主服务器二进制日志，配置虚拟IP地址等）
        安装MHA-node软件包和MHA-manager软件包（yum -y install perl-Config-Tiny.noarch perl-Time-HiRes.x86_64 perl-Parall  
        el-ForkManage perl-Log-Dispatch-Perl.noarch perl-DBD-MySQL ncftp）
        建立主从复制集群（支持基于日志点的复制      基于GTID的复制）
        配置MHA管理节点
        使用masterha_check_ssh和master_check_repl对配置进行检验
        启动并测试MHA服务

    16、MHA架构实例演示
    所有服务器都要启动gtid_mode
    IP      建立复制用户
            change master to master_host='IP',master_user='repl',master_password='123456',master_position=1;
            start slave;
    IP1     change master to master_host='IP',master_user='repl',master_password='123456',master_position=1;
            start slave;
    IP3     change master to master_host='IP',master_user='repl',master_password='123456',master_position=1;
    IP      ssh-keygen
            ssh-copy-id -i /root/.ssh/id_rsa '-p 22 root@IP'（ssh root@IP）
            ssh-copy_id -i /root/.ssh/id_rsa '-p 22 root@IP1'
            ssh-copy_id -i/root/.ssh/id_rsa '-p 22 root@IP3'
    IP1     ssh-keygen
            ssh-copy-id -i /root/.ssh/id_rsa '-p 22 root@IP'
            ssh-copy-id -i /root/.ssh/id_rsa '-p 22 root@IP1'
            ssh-copy-id -i /root/.ssh/id_rsa '-p 22 root@IP3'
    IP3     ssh-keygen
            ssh-copy-id -i /root/.ssh/id_rsa '-p 22 root@IP'
            ssh-copy-id -i /root/.ssh/id_rsa '-p 22 root@IP1'
            ssh-copy-id -i /root/.ssh/id_rsa '-p 22 root@IP3'
            mha4mysql-manager-0.5600.e16.noarch.rpm
            scp mha4mysql-node-0.56-0.e16.noarch.rpm root@IP:/root
    IP      yum -y install perl-DBD-MySQL ncftp perl-DBI.x86
    IP1     yum -y install perl-DBD-MySQL ncftp perl-DBI.x86
    IP3     yum -y install perl-DBD-MySQL ncftp perl-DBI.x86  
            rpm -ivh mha4mysql-node-0.56-0.e16.noarch.rpm
            rpm -ivh mha4mysql-manage-0.56-0.e16.noarch.rpm
            mkdir -p /etc/mha （mah的配置文件目录）
            mkdir -p /home/mysql_mha （从主服务器下载二进制文件目录）
            cd /etc/mha
            vi mysql_mha.cnf
            （[server default]
            user=mha      #用于MHA管理的数据用户
            password=123456
            manage_workdir=/home/mysqk_mha
            manage_log=/home/mysql_mha/manage.log
            remote_workdir=/home/mysql_mha      #远程工作目录
            ssh_user=root
            repl_user=repl
            repl_password=123456
            ping_interval=1
            master_binlog_dir=/home/mysql/sql_log      #主服务器的binlog目录
            master_ip_failover_script=/user/bin/master_ip_faliover
            secondary_check_script=/usr/bin/masterha_secondary_check -s IP1 -s IP3 -s IP
            [server1]
            hostname=IP
            candidate_master=1      #哪台服务器可以选为备主服务器
            [server2]
            hostname=IP1
            candidate_master=1
            [server3]
            hostname=IP3
            no_master=1）
            vi /usr/bin/master_ip_failover（my $vip = 虚拟IP my #ssh_start_vip = 'sudo /sbin/ifconfig eth0:$key $vip网  
            卡接口'）
            master_check_ssh --conf=/etc/mha/mysql_mha.cnf
            master_check_repl --conf=/etc/mha/mysql_mha.cnf
            nohup masterha_manager --conf/etc/mha/mysql_mha.cnf &（启动mha）
    IP      mysql -uroot -p
            grant all privileges on *.* to mha@'IP' identified by '123456';
            mkdir /home/mysql_mha
            show variables like '%log%'      （log_bin_basename      /home/mysql/sql_log/mysql-bin）
            ifconfig eth0:1 虚拟IP/24
            /etc/init.d/mysqld stop
            ip addr
    IP1     mkdir /home/mysql_mha
            show vaiables like '%log%';      （log_bin_basename       /home/mysql/sql_log/mysql-bin）
            ip addr
    
    18、MHA架构优缺点
    MHA工具的优点：
        同样是由Perl语言开发的开源工具
        可以支持基于GTID的复制模式
        MHA在进行故障转移时更不易产生数据丢失
        同一个监控节点可以监控多个集群
    MHA工具的缺点：
        需要编写脚本或利用第三方工具来实现Vip的配置
        MHA启动后只会对主数据库进行监控
        需要基于SSH免认证配置，存在一定的安全隐患
        没有提供从服务器的读负载均衡的功能
    
    19、读写分离和负载均衡介绍
    两种读写分离方式：
        由程序实现读写分离
        由中间件来实现读写分离
    程序读写分离
    优点：由开发人员控制什么样查询在从库中执行，由此比较灵活
        由于程序直接连接数据库，所以性能损耗比较少
    缺点：增加了开发的工作量，是程序代码更加复杂
        人为控制，容易出现错误
    读写分离中间件：mysql-proxy（性能强大，稳定性不好）
        maxScale
    由中间件实现读写分离的优点：
        由中间件根据查询语法分析，自动完成读写分离
        对程序透明，对于已有程序不用做任何调整
    由中间件实现读写分离的缺点：
        由于增加了中间层，所以对查询效率有损耗
        对于延迟敏感业务无法自动在主库执行
    如何实现读负载均衡：软件（LVS      Haproxy      MaxScale）
        硬件（F5）
    
    20、MaxScale实例演示
    MaxScale的插件：
        Authentication 认证插件
        Protocal 协议插件
        Route 路由插件
        Monitor 监控插件
        Filter&Logging 日志和过滤插件
    安装：
        https://downloads.mariadb.com/enterprise/fre6-c9jr/mariadb-maxscale/1.3.0/rhel/6/x86_64/maxscale-1.3.0-1.rhel  
            .x86_64.rpm
        yum install libaio.x86_64 libaio-devel.x86_64 novacom-server.x86_64 -y
        rpm -ivh maxscale-1.3.0-1.rhel.x86_64.rpm
        create user scalemon@'%' identified by '123456';
        grant replocation slave,replication client on *.* to scalemon@'%';
        create user maxscale@'%'  iedntified by '123456';
        grant select on mysql.* to maxscale@'%';
        maxkeys       （Generating .secrets file in /var/lib/maxscale/ ...）
        maxpasswd /var/lib/maxscale/ 123456      #加密密码
        cd /etc
        vim maxscale.cnf      
            （[maxsavle]      threads=4 #线程修改      [server1]      type=server      address=IP        
             prot=3306      protocol=MySQLBackend      [server2]      type=server       address=IP1      prot=3306      
             protocol=MySQLBackend      [server3]      type=server      address=IP3      prot=3306      
             protocol=MySQLBackend      [MySQL Monitor      type=monitor      module=mysqlmon      
             servers=server1,server2,server3      user=scalemon      passwd=123456      
             monitor_interval=10000 #监控时间间隔 单位：毫秒]       [Read-Write Service]      type=service      
             router=readwritesplit      servers=server1,server2,server3      user=maxscale      passwd=123456     
             max_slave_connections=100% #最大可用从服务器的数量      max_slave_replication_lag=60 #从服务器最大延迟 
             单位：秒）
        maxscale --config=/etc/maxscale.cnf
        ps -ed | grep maxscale
        maxadmin --user=admin --password=mariadb
        list servers #查看启动了哪些服务
        show dbusers "Read-Write Service"
                
### 六、数据库索引优化
    1、Btree索引和Hash索引
        MySQL支持的索引类型：
        B-tree索引的特点
        B-tree过引以B+书的结构存储数据
        B-tree索引能够加快数据的查询速度
        B-tree索引更适合进行范围查找
    在什么情况下可以用到B树索引：全值匹配的查询（order_sn = '9876543211900'）
        匹配最左前缀的查询
        匹配列前缀查询（order_sn like '9876%'）
        匹配范围值的查询（order_sn>'98765432119900' and order_sn<'98765432119979'
        精准匹配左前列并范围匹配另外一列
        只访问索引的查询
    Btree索引的使用限制：如果不是按照索引最左列开始查找，则无法使用索引
        使用索引时不能跳过索引中的列
        Not in 和 <>操作无法使用索引
        如果查询中有某个列的范围查询，则其右边所有列都无法使用索引
    Hash索引的特点：
        Hash索引是基于Hash表实现的，只有查询条件精准匹配Hash索引中的所有列时，才能够使用到Hash索引
        对于Hash索引中的所有列，存储引擎都会为每一行计算一个Hash码，Hash索引中存储的就是Hash码。
    Hash索引的限制：
        Hash索引必须进行二次查找
        Hash索引无法用于排序
        Hash索引不支持部分索引查找也不支持范围查找
        Hash索引中Hash码的计算可能有Hash冲突
    为什么要使用索引：
        索引大大减少了存储引擎需要扫描的数据量
        索引可以帮助我们进行排序以避免使用临时表
        索引可以把随即I/O变为顺序I/O
    索引是不是越多越好：
        索引会增加写操作的成本
        太多的索引会增加查询优化器的选择时间

    2、安装演示数据库
        http://downloads.mysql.com/docs/sakila-db.tar.gz
        tar -zxvf sakila-db.tar.gz
        mysql -uroot -p < sakila-schema.sql #数据库表结构
        mysql -uroot -p < sakila-data.sql #数据库表数据

    3、索引优化策略（上）
    索引列上不能使用表达式或函数 （select ...... from product where to_days(out_date)-to_days(current_date)>=30      ->  
        select ...... from product where out_date<=date_add(current_date,interval 30 day)）
    前缀索引和索引列的选择性（CREATE INDEX index_name IN table(col_name(n));      索引的选择性是不重复的索引值和表的记录数的比值）
    联合索引：如何选择索引列的顺序（经常会使用到的列优先      选择性高的列优先      宽度小的列优先）
    覆盖索引：
        优点（可以优化缓存，减少磁盘IO操作      
        可以减少随机IO，变随即IO操作变为顺序IO操作     
        可以避免对Innodb主键索引的二次查询      
        可以避免MyISAM表进行操作调用）
    无法使用覆盖索引的情况：
        存储引擎不支持覆盖索引
        查询中使用了太多的列
        使用了双%号的like查询

    4、索引优化策略（中）
    使用索引扫描来优化排序：
        通过排序操作
        按照索引顺序扫描数据
        索引的列顺序和Order By子句的顺序完全一致
        索引中所有列的方向（升序，降序）和Order By子句完全一致
        Order by中的字段全部在关联表中的第一张表中
    explain select actor_id,last)name from actor where last_name='Joe'\G
    explain select * from rental where rental_data>'2005-01-01' order by rental_id\G
    explain select * from rental_myisam where rental_date>'2005-01-01' order by rental_id\G
    explain select * from rental where rental_date='2005-05-09' order by inventory_id,customer_id\G
    explain select * from rental_myisam rental_date='2005-05-09' order by inventory_id,customer_id\G
    explain select * from rental where rental_date='2005-05-09' order by inventory_id desc,customer_id\G
    explain select * from rental where rental_date>'2005-05-09' order by inventory_id desc,customer_id\G
    模拟Hash索引优化查询：
        alter table_film add title varcher(32);
        update film set title=(title);
        create index inde_title on film(title)
        explain select * from film where title=('EGG IGBY') and title='EGG IGBY'\G
        只能处理键值的全值匹配查找
        所使用的Hash函数决定着索引键的大小

    5、索引优化策略（下）
    利用索引优化锁：
        索引可以减少锁定的行数
        索引可以加快处理速度，同时也加快了锁的释放
    连接1：show create table actor \G
        drop index idx_actor_last_name on actor;
        explain select * from actor where last_name='WOOD'\G
        begin;
        select * from actor where last_name='WOOD' for update;
        
        rollback;
        create index dex_lastname on actor(last_name);
        begin;
        select * from actor where last_name='WOOD' for update;
        
        rollback;
    连接3：begin;
        select * from actor where last_name='willis' for update;
        
        rollback;
        select * from actor where last_name='willis' for update;
    索引的维护和优化：
        删除重复和冗余的索引（primary key(id)（主键索引）,unique key(id)（唯一索引）,idex(id)（单列索引）      index(a),index(a,b)      primary key(id),index(a,id)（联合索引））
        pt-duplicate-key-checker h=127.0.0.1
        create index idx_customerid_staffid on paymen(customer_id,staff_id);
        查找未被使用过的索引（select object_schema,object_name,index_name,b.'TABLE_ROWS' FROM performance_schema.table)io_waits_summary_by_index_usage a JOIN information_schema.table b ON a.'OBJECT_SCHEMA'=b.'TABLE_SCHEMA' AND a.'OBJECT_NAME'=b.'TABLE_NAME' WHERE index_name IS NOT NULL AND count_star = 0 ORDER BY object_schema,object_name）
        更新索引统计信息及减少索引碎片（analyze table table_name      optimize table table_name（使用不当会导致锁表））


### 七、SQL查询优化
    1、获取有性能问题SQL的三种方法
    查询优化，索引优化，库表结构优化需要齐头并进
    
    如何获取有性能问题的SQL：
        通过用户反馈获取有性能问题的SQL
        通过满查询日志获取有性能问题的SQL
        实时获取有性能问题的SQL
    
    2、慢查询日志介绍
    使用慢查询日志获取有性能问题的SQL：
        slow_query_log 启动慢查询日志
        slow_query_log_file 指定慢查日志的存储路径及文件
        long_query_time 指定记录慢查日志SQL执行时间的伐值
        log_queries_not_using_indexes 是否记录未使用索引的SQL
    常用的慢查日志分析工具（mysqldumpslow（汇总除查询条件外其他完全相同的SQL，并将分析结果按照参数中所制定的顺序输出
    mysqldumpslow -s r -t 10 slow-mysql.log      
        -s order（c,t,l,r,at,al,ar）
            （c：总次数      t：总时间      l：锁的时间      r：总数据行      at,al,ar：t,l,r平均数      
            -t top（指定取前几条作为输出结果））））

    3、慢查询日志实例
    mysqldumpslow -s r -t 10 slow-mysql.log
    使用慢查询日志获取有性能问题的SQL：
        常用的慢查日志工具（pt-query-disgest）
        pt-query-disgest \ --explain h=127.0.0.1,u=root,p=p@ssW0rd \ slow-mysql.log

    4、实时获取性能问题SQL
    如何实时获取有性能问题的SQL：
        information_schema数据库->PROCESSLIST表
        select id,'user','host',DB,command,'time',state,info FROM information_schema.PROCESSLIST WHERE TIME>=60
    
    5、SQL的解析与处理及生成执行计划
    查询速度为什么会慢：
        客户端发送SQL请求给服务器
        服务器检查是否可以在查询缓存中命中该SQL
        服务器端进行SQL解析，预处理，再由优化器生成对应的执行计划
        根据执行计划，调用存储引擎API来查询数据
        将结果返回给客户端
        对于一个读写频繁的系统使用查询缓存很可能会降低查询处理的效率，所以在这种情况下建议大家不要使用缓存
        query_cache_type 设置查询缓存是否可用（ON,OFF,DEMAND（DEMAND表示只有在查询语句中来使用SQL_CACHE和SQL_NO_CHACHE来控  
            制是否需要缓存））OFF
        query_cache_size 设置查询缓存的内存大小 0
        query_cache_limit 设置查询缓存可用存储的最大值
        （加上SQL_NO_CACHE可以提高效率）
        query_cache_wlock_invalidate 设置数据表被锁后是否返回缓存中的数据
        query_cache_min_res_unit 设置查询缓存分配的内存块最小单位
        MySQL依照这个执行计划和存储引擎进行交互
        这个过程包括了多个子过程：
            解析SQL，预处理，优化SQL执行计划
            语法解析阶段是通过关键字对MySQL语句进行解析，并生成一棵对应的“解析树”
            MySQL解析器将使用MySQL语法规则验证和解析查询
            包括检查愈发是否使用了正确的关键字
            关键字的顺序是否正确等
            预处理阶段是根据MySQL规则进一步检查解析树是否合法
            检查查询中所涉及的表和数据列是否有及名字或别名是否有歧义等
            语法检查全部通过了，查询优化器就可以生成查询计划了
        会造成MySQL生成错误的执行计划的原因：
            统计信息不准确
            执行计划中的成本估算不等同于实际的执行计划的成本
                （MySQL服务器并不知道哪些页面在内存中      哪些页面在磁盘上      哪些需要顺序读取      哪些页面需要随机读取）
            MySQL优化器所认为的最优可能与你所认为的最优不一样（基于其成本模型选择最优的执行计划）
            MySQL从不考虑其他并发的查询，这可能会影响当前查询的速度
            MySQL不会考虑不受其控制的成本（存储过程、用户自定义的函数）
        MySQL优化器可优化的类型：
            重新定义表的关联顺序（优化器会根据统计信息来决定表的关联顺序）
            将外连接转化成內连接（where条件和库表结构等）
            使用等价变换规则
            优化count()、min()和max()（select tables optimized away   优化器已经从执行计划中移除了该表，并以一个常数代替）   
            将一个表达式转化为常数表达式
            子查询优化（子查询转换为关联查询）
            提前终止查询
        explain select * from film where film_id=-1;
        对in()条件进行优化

    6、如何确定查询处理各个阶段所消耗的时间
    使用profile：
        （减少查询所消耗的时间加快查询的响应时间）
        set profile = 1;
        启动profile
        这是一个session级的配制
        执行查询
        show profile;查看每一个查询所消耗的总时间的信息
        show profile for query N;查询每一个阶段所消耗的时间
        show profile cpu for query 1;
    使用performance_schema：
        UPDATE setup_instruments SET enabled='YES',TIMED='YES' WHERE NAME LIKE 'STAGE%';
        UPDATE setup_consumers SET enabled='YES' WHERE NAME LIKE 'events%';（开启监控）
        SELECT a.thread_id,sql_text,c.event_name,(c.timer_end - c.timer_start)/1000000000 AS'DURATION(ms)'   
            FROM events_statements_history_long a JOIN threads b ON a.THREAD_ID=b.THREAD_ID JOIN 
            events_stages_history_long c ON c.THREAD_ID=b.THREAD_ID AND c.EVENT_ID BETWEEN a.EVENT_ID 
            AND a.END_EVENT_ID WHERE b.PROCESSLIST_ID=CONNECTION_ID() AND a.EVENT_NAME='statement/sql/select' 
            ORDER BY a.THREAD_ID,c.EVENT_ID

    7、特定SQL的查询优化
    大表的数据修改最好要分批处理（1000万行记录的表中删除/更新100万行记录）
    如何修改大表的结构：（对表中的列的字段类型进行修改      改变字段的宽度时还是会锁表     无法解决主从数据库延迟的问题）
    pt-online-schema-change --alter="MODIFY c VARCHAR(150) NOT NULL DEFAULT ''" --user=root --password=PassWord   
        D=imooc,t=sbtest4 --charset=utf8 --execute
    如何优化not in和<>查询：SELECT customer_id,first_name,last_name,email FROM customer WHERE customer_id NOT IN 
        (SELECT customer_id FROM payment)
    SELECT a.customer_id,a.first_name,a.last_name,a.email FROM customer a LEFT JOIN payment b ON a.customer_id=  
        b.customer_id WHERE b.customer_id IS NULL
    使用汇总表优化查询：SELECT COUNT(*) FROM product_comment WHERE product_id=999
    汇总表就是提前以要统计的数据进行汇总并记录到表中以备后续的查询使用
    CREATE TABLE product_comment_cnt(product_id INT,cnt INT);
    显示每个商品的评论数（SELECT SUM(cnt) FROM (SLELECT cnt FROM product_comment_cnt WHERE product_id = 999 UNION ALL   
        SELECT COUNT(*) FROM product_comment_cnt WHERE product_id = 999 AND timestr>DATE(NOW())) a）


### 八、数据库的分库分表
    1、数据库分库分表的几种方式
    分库分表的几种方式：
        把一个实例中的多个数据库拆分到不同的实例
        把一个库中的表分离到不同的数据库中

    2、数据库分片前的准备
    对一个库中的相关表进行水平拆分到不同实例的数据库中
    如何选择分区键：
        分区键要能尽量避免跨分片查询的发生
        分区键要能尽量使用各个分片中的数据平均
    如何存储无需分片的表：
        每个分片中存储一份相同的数据
        使用额外的节点统一存储
    如何在节点上部署分片：
        每个分片使用单一数据库，并且数据库名也相同
        将多个分片表存储在一个数据库中，并在表名上加入分片号后缀
        在一个节点中部署多个数据库，每个数据库包含一个分片
    如何分配分片中的数据：
        按分区键的Hash值取模来分配分片数据
        按分区键的范围来分配分片数据
        利用分区键和分片的映射表来分配分片数据
    如何生成全局唯一ID：
        使用auto_increment_increment和auto_increment_offset参数
        使用全局节点来生成ID
        在Redis等缓存服务器中创建全局ID

    3、数据库分片演示（上）
    oneProxy安装和配置
    wget htp:www.onexsoft.cn/software/oneproxy-rhel6-linux64-v5.8.1-ga.tar.gz
    tar -zxf oneproxy-rhel6-linux64-v5.8.1-ga.tar.gz
    node：
        create user test@'IP' identified by '123456';
        create database orders;
        grant all privileges on orders.* to test@'IP3';
        use orders
        create table order_detail_0(order_id int not null,add_time datetime not null,order_amount decimal(6,2),primary  
            key ('order_id'));
        create table order_product_0(order_id int not null,order_product_id int not null,primary key ('order_id'));
        create table category(id int null,category_name varchar(10),primary key(id));             
    node1：
        create user test@'IP1' identified by '123456';
        create database orders;
        grant all privilieges on order.* to test@'IP3';
        use orders
        create table order_detail_1(order_id int not null, add_time datetime not null,order_amount decimal(6,2),  
            primary key ('order_id'));
        create_table order_product_1(order_id int not null,order_product_id int not null,primary key  ('order_id'));
        create table category(id int not null,category_name varchar(10),primary key(id));
    node3：
        vi demo.sh（export ONEPROXY_HOME=/usr/local/oneproxy）
           cd conf
           vi proxy.conf（mysql-version=5.7.9-log    proxy-address=:3306      proxy-master-addresses.1=IP:3306@order01  
                proxy-master-addresses.2=IP1:3306@order02      proxy-user-list=数据库账号/数据库密码@数据库名称）
           cd ..
           ./demo.sh
           mysql -P4041 -uadmin -pOneProxy -h127.0.01
           passwd "123456";
           exit
           cd conf
           vi proxt.conf（proxy-part-template=conf/order_part.txt      proxy-group-policy=order01:master-only      
                proxy-group-policy=order02:master-only）
           vi order_part.txt（[
           {
             "table":"order_detail",
             "pkey":"order_id",
             "type":"int",
             "method":"hash",
             "partitions":
                   [
                         {"suffix":"_0","group":"order01"},
                         {"suffix":"_1","group":"order02"}
                    ] 
                },
            {
              "table":"order_product",
              "pkey":"order_id",
              "type":"int",
              "method":"hash",
              "partitions":
              [
                    {"suffix":"_0","group":"order01"},
                    {"suffix":"_1","group":"order02"}
              ]
            },
            {
              "table":"category",
              "pkey":"id"
              "type":"int",
              "method":"global",
              [
                    {"group":"order01"},
                    {"group":"order02"}
              ]
            }
            ]）
            kill -9 1164
            kill -9 1165
            ./demo.sh
            mysql -p4041 -h127.0.0.1 -uadmin -pOneProxy
            list backend;（查看服务器）
            list tables;
            exit;
            vi test.sh（#!/bin/bash
            order_id=1
            while:
                do
                order_id='echo $order_id+1|bc'
                sql="insert into order_detail (order_id,add_time,order_amount) values(${order_id},now(),100.00)"
                echo $sql | mysql -utest -p123456 -h127.0.0.1
                
                sql2="insert into order_product(order_id,order_product_id) values(${order_id},$),${order_id}*10)"
                echo sql2 | mysql -utest -p123456 -h127.0.0.1
                
                sql3="insert into category(id,category_name) values(${order_id},'test123')"
                echo sql3 | mysql -utest -p123456 -h127.0.0.1

                done）
                bash ./test.sh


### 九、数据库监控
    1、数据库监控介绍
    对什么进行监控：
        对数据库服务可用性进行监控
        对数据库性能进行监控（QPS和TPS      并发线程数量      如何对Innodb阻塞和死锁进行监控）
        对主从复制进行监控（主从复制链路状态的监控      主从复制延迟的监控      定期的确认主从复制的数据是否一致）
        对服务器资源的监控
            （磁盘空间      
            服务器磁盘空间大并不意味着MySQL数据库能使用的空间就足够大      
            CPU的使用情况，内存的使用情况，Swap分区的使用情况以及网络IOde情况等）

    2、数据库可用性监控
    如何确认数据库是否可用通过网络连接：
        mysqladmin -umonitor_user -p -g ping
        telnet ip db_port
        使用程序通过网络建立数据库连接
    如何确认数据库是否可读写：
        检查数据库的read_only参数是否为off
        建立监控表对表中数据进行更新
        执行简单的查询 select @@version
    如何监控数据库的连接数：
        show variables like 'max_connections';
        show blobal status like 'Threads_connected'
        Threads_connected/max_connections > 0.8

    3、数据库性能监控
    记录性能监控过程中所采集到的数据库的状态
    如何计算QPS和TPS：
        QPS=(Queries2-Queries1)/(Uptime_since_flush_status2-Uptime_since_flush_status1)
        TPS=((Com_insert2+Com_update2+Com_delete2) - (Com_insert1+Com_update1+Com_delete1))/  
            (Uptime_since_flush_status2-Uptime_since_flush_status1)
    如何监控数据库的并发请求数量：数据库的性能会随着并发处理请求的数量的增加而下降
    show global status like 'Threads_running'
    并发处理的数量通常会远小于同一时间连接到数据库的线程的数量
    如何监控Innodb的阻塞：SELECT b.trx_mysql_thread_id AS '被阻塞线程',b.trx_query AS '被阻塞SQL',  
        c.trx_mysql_thread_id AS '阻塞线程',c.trx_query AS '阻塞SQL' (UNIX_TIMESTAMP()-UNIX_TIMESTAMP(c.trx_started))  
        AS '阻塞时间' FROM information_schema.INNODB_LOCK_WAITS a JOIN information_schema.INNODB_TRX b ON   
         a.requesting_trx_id=b.trx_id JOIN information_schema.INNODB_TRX c ON a.blocking_trx_id=c.trx_id   
         WHERE(UNIX_TIMESTAMP()-UNIX_TIMESTAMP(c.trx_started)) >60
    IP：select connection_id();;
        set global innodb_local_wait_time=180;
        use test;
        select * from t;
        begin;
        select * from t for update;
    IP1：use test;
        begin;
        select * from t for update;

    4、MySQL主从复制监控
    如何监控主从复制链路的状态：show slave status
        如何监控主从复制延迟：
        show slave status（Seconds_Behind_Master）
        主上的二进制日志文件和偏移量（show master status\G）
        show slave status
        已经传输完成的主上二进制日志的名字和偏移量
    如何验证主从复制的数据是否一致：
        pt-table-checksum
        pt-table-checksum u=dba,p='PassWord' --databases mysql --replicate test.chechsums
        GRANT SELECT,PROCESS,SUOER,REPLICATION SLAVE ON *.* TO 'dba'@'ip' IDENTIFIED BY 'PassWord';