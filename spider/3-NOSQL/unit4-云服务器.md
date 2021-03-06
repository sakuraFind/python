# mongoDB云服务器

##mongoDB
1. 下载安装文件

		curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.2.tgz    # 下载
		tar -zxvf mongodb-linux-x86_64-3.0.6.tgz                                   # 解压
		mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb                         # 将解压包拷贝到指定目录
2. MongoDB 的可执行文件位于 bin 目录下，所以可以将其添加到 PATH 路径中：

		export PATH=<mongodb-install-directory>/bin:$PATH
		source /etc/profile
	查看配置信息
	
		echo $PATH
3. 创建工作目录

		mkdir -p /data/db
		mkdir -p /data/log
		
	这个时候就可以本机测试了，开通服务 mongod --dbpath /data/db;打开客户端 mongo
	
4. 添加配置文件

	这里为了方便，我们把配置文件mongodb.conf 放到了bin目录下（/etc/local/mongodb/bin）
	
	文件信息
	
		dbpath=/data/db
		logpath=/data/log/mongodb.log
		logappend=true
		port=27017
		fork=true
		bind_ip=0.0.0.0
		##auth = true # 先关闭, 创建好用户在启动
		
	**为了可以远程访问，bind_ip=0.0.0.0一定要加入
	
5. 通过配置文件启动

		mongod -f /usr/local/mongodb/bin/mongodb.conf
		
	出现以下信息即表示启动成功
	
		about to fork child process, waiting until server is ready for connections.
		forked process: 2400
		child process started successfully, parent exiting
6. 配置防火墙(此步骤可以跳过*********)

	查看现有的防火漆配置信息 
	
		more /etc/sysconfig/iptables 
		
	将27017端口添加到防火墙中

		vi /etc/sysconfig/iptables
		    -A INPUT -m state --state NEW -m tcp -p tcp --dport 27017 -j ACCEPT
		/etc/init.d/iptables reload

	注：如若不想修改iptables表，可以直接输入下面命令：

		# iptables -I INPUT -p tcp --dport 8889 -j ACCEPT
		
	若/etc/sysconfig/iptables不存在，
		
	原因：在新安装的linux系统中，防火墙默认是被禁掉的，一般也没有配置过任何防火墙的策略，所有不存在/etc/sysconfig/iptables文件。
		
	解决：
		
	在控制台使用iptables命令随便写一条防火墙规则，如：iptables -P OUTPUT ACCEPT
	
	使用service iptables save进行保存，默认就保存到了/etc/sysconfig目录下的iptables文件中
	
	[参考文档](https://www.cnblogs.com/blog-yuesheng521/p/7198829.html)
	
5. 配置mongodb开机启动，设置mongodb.service启动服务

	
	
	1. 新建配置文件
	
			cd /lib/systemd/system
			vi mongodb.service
 
 	2. 配置信息
 	
			[Unit]
			Description=mongodb
			After=network.target remote-fs.target nss-lookup.target
			 
			[Service]
			Type=forking
			ExecStart=/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf
			ExecReload=/bin/kill -s HUP $MAINPID
			ExecStop=/usr/local/mongodb/bin/mongod --shutdown --config /usr/local/mongodb/bin/mongodb.conf
			PrivateTmp=true
			 
			[Install]
			WantedBy=multi-user.target
		
	3. 设置mongodb.service权限

			chmod 754 mongodb.service
			
	4. 系统mongodb.service操作命令

			#启动服务
			systemctl start mongodb.service
			#关闭服务
			systemctl stop mongodb.service
			#开机启动
			systemctl enable mongodb.service

6. shell 管理进程

	查看mongodb的进程的方式：
	
		ps -ef | grep mongodb

	杀死mongodb进程的方式：
	
		pkill mongod


##mysql

1. 安装前看是否安装过mysql

		yum list installed mysql*
		
2. 删除mysql

		yum  remove mysql*
		
3. 查看yum库下是否有mysql-server

		yum list mysql*
		#或者
		yum list | grep mysql 或 yum -y list mysql*
		
	如果没有（一般在centos7下没有）

	wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
	
	rpm -ivh mysql-community-release-el7-5.noarch.rpm
	
4. ####安装mysql
	
		yum install mysql-server
		yum install mysql-devel

5.	mysql配置文件

		/etc/my.cnf的[mysqld]中加入character-set-server=utf8
		
6. 启动mysql服务：

		service mysqld start
		或者/etc/init.d/mysqld start
		
		service mysqld start
		service mysqld stop
		service mysqld restart
		service mysqld status
		
	**设置开机启动：**
	
7. 创建root管理员：
		
		假如：
		mysqladmin -u root password 666666
		登录：
			mysql -u root -p
		如果忘记密码，则执行以下代码
			service mysqld stop
			mysqld_safe --user=root --skip-grant-tables
		加入密码****	
			mysql -u root
			use mysql
			update user set password=password("666666") where user="root";
			flush privileges;
			
8. 开放防火墙的端口号

		mysql增加权限：mysql库中的user表新增一条记录host为“%”，user为“root”。
		use mysql;
		UPDATE user SET `Host` = '%' WHERE `User` = 'root' LIMIT 1;
		%表示允许所有的ip访问

