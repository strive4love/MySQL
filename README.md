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
锁
事务
备份与恢复
性能调优
SQL
InnoDB存储引擎源代码
