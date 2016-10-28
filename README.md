MySQL体系结构
•	数据库（database）：文件的集合，在MySQL中，数据库文件可以是frm、myd、myi、ibd结尾的文件
•	数据库实例(instance)：后台进程/线程 + 共享内存，数据库实例才是真正操作数据库文件的。MySQL数据库实例在系统上的表现就是一个进程，因为它被设计为一个单进程多线程架构的数据库，这点与SQL Server比较类似，但与Oracle多进程的架构 
•	# ./mysqld_safe & 命令可启动MySQL实例, 本质是读取配置文件，根据配置文件的参数来启动数据库实例
•	# ./mysql  - - help | grep my.cnf 命令可查看读取哪些配置文件，及读取的顺序。 配置文件爱你中datadir参数指定数据库所在路径
•	# ps –ef |grep mysqld  查看启动的MySQL实例
•	MySQL由一下几部分组成：连接池组件、管理服务和工具组件、SQL接口组件、查询分析器组件、优化器组件、缓冲（Cache）组件、插件式存储引擎、物理文件。 MySQL区别于其他数据库最重要的特点就是其插件式存储引擎。mysql> show engines \G; 语句可查看当前MySQL数据库所支持的存储引擎
•	连接MySQL的实质是进程间通信；客户端端进程和MySQL数据库实例进行通信。进程通信方式有管道、命名管道、TCP/IP套接字、Unix域名套接字。MySQL提供的连接方式从本质上看都这些进程通信方式。
o	如两个通信进程在同一台服务器上，可以使用命名管道，但需要在MySQL配置文件中启用- - enable-named-pipe选项。
o	如果客户端进程想使用共享内存方式连接MySQL，需要在MySQL服务器端的配置文件中添加- - shared-memory选项，同时客户端还必须使用-protocol = memory选项。
o	Unix套接字不是一个网络协议，所以只能在MySQL客户端和数据库实例在同一台Unix或Linux服务器上的情况下使用，需要在MySQL配置文件中指定套接字文件的路径，如-socket=/tmp/mysql.sock 当数据库实例启动后，我们可以通过下列命令来进行Unix套接字文件的查找 mysql> show variables like ‘socket’; 知道UNIX套接字文件的路径后，就可以使用该方式进行连接了，如 # mysql –uXXX –S  /tmp/mysql.sock
o	TCP/IP套接字方式是MySQL在任何平台下都提供的连接方式。客户端执行程序mysql>mysql –h192.168.0.101 –u XXX –p 发起连接请求，服务器端会先检查一张权限视图(名为user)  从而来判断是否允许客户端用户的连接。
InnoDB 存储引擎 
文件 
表
索引与算法
•	索引的基本原理：通过不断地缩小想要获得的数据的范围筛选出最终想要的结果，同时把随机的事件变成顺序的事件。
。一次磁盘IO时间 = 寻到时间（5ms以下）+ 旋转延迟（7200转的磁盘是4.17ms）+ 从磁盘读出或将数据写入磁盘的时间（一般0.几ms）= 9 ms
。局部预读性原理：当计算机访问一个地址的数据，与其相邻的数据也会很快被访问到。
。页/块（page/block）：每一次 I/O 读取的数据我们称之为一页。具体一页有多大数据跟操作系统有关，一般为4k或8k。 
。我们读取一页以内的数据，实际只发生一次I/O，这个理论对于索引的数据结构设计非常有帮助。
。索引的数据结构：我们需要一个

锁
事务
备份与恢复
性能调优
SQL
InnoDB存储引擎源代码



## 

MySQL体系结构
•	数据库（database）：是文件的集合，在MySQL中，数据库文件可以是frm、myd、myi、ibd结尾的文件。如果要对这些文件执行诸如SELECT  、INSERT、UPDATE、DELETE之类的操作，不能简单的操作文件来更改数据的内容，需要通过数据库实例来完成对数据库的操作。
•	数据库实例(instance)：才是真正操作数据库文件的。MySQL数据库实例在系统上的表现就是一个进程，因为它被设计为一个单进程多线程架构的数据库，这点与SQL Server比较类似，但与Oracle多进程的架构有所不同。因此数据库实例本质就是“后台进程/线程 + 共享内存”，
•	客户端连接MySQL的实质是进程间通信；客户端进程和MySQL数据库实例进行通信。进程通信方式有管道、命名管道、TCP/IP套接字、Unix域名套接字。MySQL提供的连接方式从本质上看都是这些进程通信方式（具体的连接细节参考ODBC/JDBC 或操作系统进程篇）。
o	如两个通信进程在同一台服务器上，可以使用命名管道，但需要在MySQL配置文件中启用- - enable-named-pipe选项。
o	如果客户端进程想使用共享内存方式连接MySQL，需要在MySQL服务器端的配置文件中添加- - shared-memory选项，同时客户端还必须使用-protocol = memory选项。
o	Unix套接字不是一个网络协议，所以只能在MySQL客户端和数据库实例在同一台Unix或Linux服务器上的情况下使用，需要在MySQL配置文件中指定套接字文件的路径，如-socket=/tmp/mysql.sock 当数据库实例启动后，我们可以通过下列命令来进行Unix套接字文件的查找 mysql> show variables like ‘socket’; 知道UNIX套接字文件的路径后，就可以使用该方式进行连接了，如 # mysql –uXXX –S  /tmp/mysql.sock
o	TCP/IP套接字方式是MySQL在任何平台下都提供的连接方式。客户端执行程序mysql>mysql –h192.168.0.101 –u XXX –p  命令发起连接请求，服务器端会先检查一张权限视图(名为user)  从而来判断是否允许客户端用户的连接。
•	启动数据库实例：类似于unix系统的启动，数据库实例的启动过程也是按顺序读取配置文件，完成全局变量初始化的过程。 # ./mysql  - - help | grep my.cnf 命令可查看实例进程读取哪些配置文件，及读取的顺序。 配置文件中配置了大量的参数，例如datadir参数指定数据库所在路径。MySQL启动后可以通过mysql> show variables like ‘xxx’ \G 命令来根据变量名称模糊查询所有变量。 # ps –ef |grep mysqld 命令可 查看启动的MySQL实例
•	MySQL的设计也是采用模块化（组件）设计，由以下组件组成：
连接池组件、
管理服务和工具组件、
SQL接口组件、
查询分析器组件、
优化器组件、
缓冲（Cache）组件、
插件式表存储引擎、
物理文件。
数据库实例（进程）作为司令官 可以派生多个线程去启动这些组件完成其所负责的特定任务，例如：数据库实例可启动存储引擎线程，让其负责执行基于表的存储功能。

InnoDB 表存储引擎 
•	存储引擎是基于表的，而不是数据库。
•	MySQL区别于其他数据库最重要的特点就是其插件式存储引擎（思考：是否会影响性能呢？）。MySQL是开源的，你可以根据MySQL预定义的存储引擎接口编写自己的存储引擎，或者修改某种存储引擎来实现自己想要的特性。MySQL官方手册第16章给出了编写自定义存储引擎的过程。每种存储引擎的实现都不相同。其中InnoDB存储引擎支持事务、支持外键、并支持类似于Oracle的非锁定读，即默认情况下读取操作不会产生锁。MySQL在Windows版本下的innoDB是默认的存储引擎，同时 InnoDB默认的被包含在所有的 MySQL二进制发布版本中。mysql> show engines \G; 命令可查看当前MySQL数据库所支持的存储引擎。
•	InnoDB存储引擎由若干个线程和3个内存缓冲区（pool）组成，
o	1个mast  thread（innoDB实例）,
Oracle的存储引擎是多进程架构，核心的后台进程有CKPT、DBWn、LGWR、ARCn、PMON、SMON， 而InnoDB存储引擎是在一个被称为master thread（innoDB实例）的线程上几乎实现了所有的功能。该线程优先级别最高。其内部由几个循环组成，
主循环（loop）,
后台循环（background loop），
刷新循环（flush loop），
暂停循环（suspend loop）。
Master Thread会根据数据库运行的状态在这些循环中进行切换。其伪代码如下
void master_thread(){
    goto loop;
      loop:　//分大概（并不精确）每秒钟的操作和每１０秒的操作（频率）两大部分。
      for(int  i = 0 ; I < 10 ; i++){　
      	thread_sleep(1) //为了控制频率为1 s,以下是每秒一次的操作
redo log buffer flush to disk //（总是）日志缓冲刷新到磁盘，即使这个事务还没有提交。　　　　　　　　　　　
　　　　　　　　　　　　　　解释了为什么再打的事务commit的时间也是很快的。
if( last_one_second_ios < 5)　
　　//如果当前一秒内I/O 次数小于５，合并插入缓冲
        do merge at most 5 insert buffer 
if( buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct)
　　//如果当前buffer pool中脏页的比例超过了配置文件中
         innodb_max_dirty_pages_pct(默认为９０，代表90%),将作磁盘同步操作，将
         100个脏页写入磁盘。
        do buffer pool flush 100  dirty page	
else if enable adaptive flush　//如果参数innodb_adaptive_flushing(自适应的刷新)开启
        do buffer pool flush desired amount dirty page　//将通过一个函数来判断需要刷新脏
　　　　　　　　　　　　　　　　　　　　　　　　页的最合适的数量
if ( no user activity)
	goto background loop　//如果当前无用户活动，切换到back ground loop
       }

       If (last_ten_second_ios < 200)
	do buffer pool flush 100 dirty page //刷新100个脏页到磁盘

       do merge at most 5  insert buffer//合并至多５个插入缓冲
        do log buffer flush to disk　//将日志缓冲刷新到磁盘
        do full purge //删除无用的undo页，对表执行update／delete 操作时，原先的行被标记
　为删除，但是因为一致性读（consistent read）的关系，需要保留这些行版
本的信息。但在full purge过程中，innoDB引擎会判断当前事务系统中已被标记为删除的行是否可以删除，比如有时候可能还有查询操作需要读取之前版本的Undo信息，如果可以，InnoDB会立即删除。　
        If ( buf_get_modified_ratio_pct > 70%)　//如果脏页在缓冲池中比例大于７０％
	do buffer pool flush 100　dirty page　//刷新１００个脏页到磁盘
else
	do buffer pool flush 10　dirty page　//刷新１０个脏页到磁盘
        do fuzzy checkpoint 　//产生一个检查点，也称模糊检查点，将最老日志序列号的脏页写
入磁盘
        goto loop
        background  loop:
        do full page　//删除无用的undo页
        do merge 20 insert buffer //合并２０个插入缓冲
        If not idle:　//如果有用户请求
        	goto loop://跳回到主循环
        else
	goto flush loop

        flush loop:
        do buffer pool flush 100 dirty page 
        If (buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct )
	go to flush loop　//不断刷新100个页直到符合条件

        goto suspend loop　
        suspend loop:
        suspend_thread()
        waiting event
        goto loop;
}


潜在问题：以上伪代码中数字200（即i/o吞吐量）都是硬编码，实际的吞吐量上可动态的配置在参数innodb_io_capacity中，默认值为２００, innodb_max_dirty_pages_pct该参数值默认为９０（”太大了”），脏页比例大于90％才同步１００个脏页到磁盘，如果你有很大的内存或你的数据库服务器的压力很大，这时刷新脏页的速度反而可能会降低。新版本将该参数默认值被改为了75
•	Merge insert buffer (插入缓冲)：　理解索引之后才能理解它
•	Full purge: 
•	两次写：当数据
•	自适应哈西索引

o	1个锁监控线程，
o	1个错误监控线程，
o	1个insert buffer thread,
o	1个log thread,
o	1个或4个（innodb Plugin版本）read thread,
o	1个或4个（innodb Plugin版本）write thread，

o	buffer pool（大小由innodb_buffer_pool_size参数指定)：
mysql> show variable like ‘innodb_buffer_pool_size’ \G ;查看buffer pool大小。
该缓冲池用来存放各种数据的缓存。当你执行SELECT操作时，数据库实例（进程）会将请求（优化后的SQL）交给存储引擎(master线程），其调用xxx线程将数据库文件按帧/页（每帧/页16K）读取到buffer pool，然后按最近最少使用（LRU）的算法来保留buffer pool中的缓存数据。buffer pool中缓存的数据页类型有：
数据页（data page）
索引页(index page)，
插入缓冲（insert buffer），
自适应哈希索引（adaptive hash index）
innoDB存储的锁信息（lock info）
和数据字典信息（data dictionary），
如果要执行INSERT/UPDATE等操作，总是首先修改buffer pool中缓存的页（发生修改后该页即为脏页），然后再按照一定的频率（由main Thread控制）将buffer pool中的脏页刷新到文件。通过命令mysql> show engine innodb status \G　来查看buffer pool具体使用情况（帧/页的总个数及使用个数） 

o	redo log buffer（大小由innodb_log_buffer_size参数指定）
mysql> show variable like ‘innodb_log_buffer_size’ \G ;
将重做日志信息先放入这个缓冲区，然后按一定频率（每秒）将其刷新到重做日志文件。如果频率是一秒，则要保证每秒产生的事务量在这个缓冲大小之内。
o	additional memory pool，也被称为堆区（heap）
（大小由innodb_additional_mem_pool_size指定）
mysql> show variable like ‘innodb_additonal_mem_pool_size’ \G ;
InnoDB master thread会以frame/page为单位申请buffer pool的空间，但是每个buffer pool中的frame buffer都有对应的buffer control block(缓冲控制对象)，而且这些对象记录了诸如LRU、锁、等待等方面的信息，而这个对象的内存需要从additional memory pool中申请。因此当你申请了很大的innoDB buffer时，这个值也应该相应增加。一旦additional memory pool空间用尽，InnoDB master thread会从buffer pool中申请额外的空间来补充











•	Mast Thread：　
•	
•	 InnoDB存储引擎的启动和关闭：本质是MySQL实例的启动过程中对InnoDB表存储引擎的处理过程。例如关闭时（即实例进程即将死亡），参数innodb_fast_shutdown影响innoDB的处理行为，
若该值为０，InnoDB存储引擎在关闭之前需要完成所有的full purge 和merge inset buffer操作
若该值为１（１是默认值），不需要完成上述full purge 和merge insert buffer操作，但buffer pool中的脏页会同步到磁盘。
若该值为２，则不完成full purge和merge insert buffer，也不将buffer pool中的脏页写回磁盘，而是将log buffer pool中的page 同步到磁盘（写入日志文件）以保证不会有任何事务会丢失。
文件 
•	参数文件：当MySQL实例启动时，会先去读一个配置参数（文本）文件，若找不到该参数文件，MySQL实例仍然可正常启动，这时所有的参数值取决于编译MySQL时指定的默认值和源代码中指定参数的默认值。但如果MySQL在默认的数据库目录下找不到mysql架构，则实例启动会失败。
o	参数查找：可以通过mysql > show variables 命令查看所有参数，或通过like 过滤参数名。从MySQL 5.1版本开始，可以通过information_schema架构下的GLOBAL_VARIABLES视图进行查找。 
o	参数类型：静态参数，在整个实例生命周期内不能进行更改。动态参数，你可以在MySQL实例运行中进行更改。可以通过SET命令对动态参数的值进行更改。
例１mysql> set  read_buffer_size = 524288
例２mysql> set  @@global.read_buffer_size = 1048576
•	错误日志文件(主机名.err)：对MySQL的启动、运行、关闭过程进行了记录（类似于Oracle的alert文件）,该文件默认名称为“$hostname.err”。可通过mysql> show variables like ‘log_error’ 命令来定位该文件。当MySQL 不能正常启动，第一个必须查看错误日志文件
•	慢查询日志文件：可以通过参数long_query_time设置一个阈（yu）值，默认值为10（秒），可精确到1微妙，同时设置参数log_slow_queries 为ON,则运行时间超过（大于）该值得所有SELECT语句都记录到慢查询日志文件中。另外如果设置了long_queries_not_using_indexes为ON,则所有运行过的没有使用索引的SQL语句也将会被记录到慢查询日志文件。文件会记录SQL语句运行的账户和IP、运行时间、锁定时间、返回行等。可以使用mysqldumpslow命令来更高效地查看慢查询日志文件，如/usr/local/mysql/bin/mysqldumpslow –s al –n 10 xxx.log 会得到锁定时间最长的10条SELECT 语句。如果将动态参数log_output指定为TABLE（默认是FILE） mysql> set global log_output= ‘TABLE’，慢查询日志记录会被放入一张名为slow_log的表中。可以改变该表的默认存储引擎（CSV引擎）以提高slow_log表在大数据量下的查询速度，更改之前需要先关闭慢查询日志开关，
mysql> set global slow_query_log=off     
mysql> alter table mysql.slow_log engine=myisam
•	查询日志（主机名.log）文件：记录所有对MySQL数据库请求的信息，不论这些请求是否得到了正确的执行，同样，查询日志的记录放入general_log表中。
•	二进制日志文件：记录对数据库执行更改的所有操作，但不包括SELECT和SHOW这类操作，因为这类操作对数据库本身并没有修改，该文件作用主要有两个：回复和复制 。默认情况下二进制日志文件并不启动，可以通过配置参数log-bin = $name 来启动，若只配置为log-bin不指定名字，则系统会采用主机名作为默认名字。二进制日志文件以序列号作为后缀名，二进制日志文件路径为数据库所在目录（mysql> show variables like ‘datadir’ 获知数据库所在目录）
以下配置文件的参数影响着二进制日志记录的信息和行为：
o	max_binlog_size
指定单个二进制日志文件的最大值，默认值为1073741824代表1GB如果超过该值，则产生新的二进制日志文件，后缀名+1，并记录到.index文件（二进制日志文件的索引文件，用来存放过往生产的二进制日志序号，不建议手工修改这个文件）
o	binlog_cache_size
指定缓存的大小，默认为32K。当使用事务的表存储引擎时，所有未提交的二进制日志会被记录到一个缓存中，等该事务提交时直接将缓存中的二进制日志写入二进制日志文件，缓存的分配是基于会话的，即当一个线程开始一个事务时，MySQL会自动分配一个大小为 binlog_cache_size的缓存，因此该值不能设置太大。当事务的记录大于设定的binlog_cache_size时，MySQL会把缓存中的日志写入一个临时文件中，因此该值又不能设得太小。可通过mysql> show global status like ‘binlog_cache%’ 命令查看binlog_cache_use（记录使用缓存写二进制日志文件的次数）和binlog-cache_disk_use（记录使用临时文件写二进制日志文件的次数）参数的状态
o	sync_binlog
默认值为0，

o	binlog-do-db
o	binlog-ingore-db
o	log-slave-update
o	binlog_format


表
索引与算法
锁
事务
备份与恢复
性能调优
SQL
InnoDB存储引擎源代码
