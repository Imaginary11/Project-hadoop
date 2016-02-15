##hadoop 
###day1
#####解决问题
* 海量资源存储(hdfs)
* 海量数据分析mapReduce
* 资源管理调度yarn

#####分布式文件系统
通过水平扩展计算机的计算能力 


##基本操作命令
* hadoop fs -put /root/jdk1.7.tar.gz hdfs://alex:9000/jdk1.7    上传文件
* hadoop fs -get hdfs://alex:9000/jdk1.7    下载文件
* hadoop fs -ls /  显示指定目录详细信息
* hadoop fs -lsr /  递归显示
* hadoop fs -mv /source  /target
* hadoop fs -cp / /
* hadoop fs -mkdir /soft
* hadoop fs -rm /xxx
* hadoop fs -rmr /xx
* hadoop fs -touchz /xxxx
* hadoop fs -cat /xxx


##安装方法
a1.准备Linux环境
	1.0点击VMware快捷方式，右键打开文件所在位置 -> 双击vmnetcfg.exe -> VMnet1 host-only ->修改subnet ip 设置网段：192.168.8.0 子网掩码：255.255.255.0 -> apply -> ok
		回到windows --> 打开网络和共享中心 -> 更改适配器设置 -> 右键VMnet1 -> 属性 -> 双击IPv4 -> 设置windows的IP：192.168.8.100 子网掩码：255.255.255.0 -> 点击确定
		在虚拟软件上 --My Computer -> 选中虚拟机 -> 右键 -> settings -> network adapter -> host only -> ok	
	1.1修改主机名
		vim /etc/sysconfig/network
		
		NETWORKING=yes
		HOSTNAME=itcast01    ###
 
	1.2修改IP
		两种方式：
		第一种：通过Linux图形界面进行修改（强烈推荐）
			进入Linux图形界面 -> 右键点击右上方的两个小电脑 -> 点击Edit connections -> 选中当前网络System eth0 -> 点击edit按钮 -> 选择IPv4 -> method选择为manual -> 点击add按钮 -> 添加IP：192.168.8.118 子网掩码：255.255.255.0 网关：192.168.1.1 -> apply
	
		第二种：修改配置文件方式（屌丝程序猿专用）
			vim /etc/sysconfig/network-scripts/ifcfg-eth0
			
			DEVICE="eth0"
			BOOTPROTO="static"               ###
			HWADDR="00:0C:29:3C:BF:E7"
			IPV6INIT="yes"
			NM_CONTROLLED="yes"
			ONBOOT="yes"
			TYPE="Ethernet"
			UUID="ce22eeca-ecde-4536-8cc2-ef0dc36d4a8c"
			IPADDR="192.168.8.118"           ###
			NETMASK="255.255.255.0"          ###
			GATEWAY="192.168.8.1"            ###
			
	1.3修改主机名和IP的映射关系
		vim /etc/hosts
			
		192.168.8.118	itcast01
	
	1.4关闭防火墙
		#查看防火墙状态
		service iptables status
		#关闭防火墙
		service iptables stop
		#查看防火墙开机启动状态
		chkconfig iptables --list
		#关闭防火墙开机启动
		chkconfig iptables off
	
	1.5重启Linux
		reboot

2.安装JDK
	2.1上传
	
	2.2解压jdk
		#创建文件夹
		mkdir /usr/java
		#解压
		tar -zxvf jdk-7u55-linux-i586.tar.gz -C /usr/java/
		
	2.3将java添加到环境变量中
		vim /etc/profile
		#在文件最后添加
		export JAVA_HOME=/usr/java/jdk1.7.0_55
		export PATH=$PATH:$JAVA_HOME/bin
	
		#刷新配置
		source /etc/profile
		
3.安装hadoop2.4.1
	注意：hadoop2.x的配置文件$HADOOP_HOME/etc/hadoop
	伪分布式需要修改5个配置文件
	3.1配置hadoop
	第一个：hadoop-env.sh
		vim hadoop-env.sh
		#第27行
		export JAVA_HOME=/usr/java/jdk1.7.0_65
		
	第二个：core-site.xml
		<!-- 制定HDFS的老大（NameNode）的地址 -->
		<property>
			<name>fs.defaultFS</name>
			<value>hdfs://itcast01:9000</value>
		</property>
		<!-- 指定hadoop运行时产生文件的存储目录 -->
		<property>
			<name>hadoop.tmp.dir</name>
			<value>/itcast/hadoop-2.4.1/tmp</value>
        </property>
		
	第三个：hdfs-site.xml
		<!-- 指定HDFS副本的数量 -->
		<property>
			<name>dfs.replication</name>
			<value>1</value>
        </property>
		
	第四个：mapred-site.xml (mv mapred-site.xml.template mapred-site.xml)
		mv mapred-site.xml.template mapred-site.xml
		vim mapred-site.xml
		<!-- 指定mr运行在yarn上 -->
		<property>
			<name>mapreduce.framework.name</name>
			<value>yarn</value>
        </property>
		
	第五个：yarn-site.xml
		<!-- 指定YARN的老大（ResourceManager）的地址 -->
		<property>
			<name>yarn.resourcemanager.hostname</name>
			<value>itcast01</value>
        </property>
		<!-- reducer获取数据的方式 -->
        <property>
			<name>yarn.nodemanager.aux-services</name>
			<value>mapreduce_shuffle</value>
        </property>
	
	3.2将hadoop添加到环境变量
	
	vim /etc/proflie
		export JAVA_HOME=/usr/java/jdk1.7.0_65
		export HADOOP_HOME=/itcast/hadoop-2.4.1
		export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

	source /etc/profile
	
	3.3格式化namenode（是对namenode进行初始化）
		hdfs namenode -format (hadoop namenode -format)
		
	3.4启动hadoop
		先启动HDFS
		sbin/start-dfs.sh
		
		再启动YARN
		sbin/start-yarn.sh
		
	3.5验证是否启动成功
		使用jps命令验证
		27408 NameNode
		28218 Jps
		27643 SecondaryNameNode
		28066 NodeManager
		27803 ResourceManager
		27512 DataNode
	
		http://192.168.8.118:50070 （HDFS管理界面）
		http://192.168.8.118:8088 （MR管理界面）
		
4.配置ssh免登陆
	#生成ssh免登陆密钥
	#进入到我的home目录
	cd ~/.ssh

	ssh-keygen -t rsa （四个回车）
	执行完这个命令后，会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）
	将公钥拷贝到要免登陆的机器上
	ssh-copy-id localhost
