title: 数据库-hadoop集群安装
date: 2019-04-12
tags: 大数据
categories: 数据库
layout: post

------

摘要：本文章介绍Hadoop集群安装的环境和步骤

原文作者：张建中

<!--more-->

# 环境准备

1. 集群结构

   > 三个结点：一个主节点master，两个从节点，内存1G以上，磁盘20G以上

   | IP地址          | 主机名 | 功能                                           |
   | --------------- | ------ | ---------------------------------------------- |
   | 192.168.139.100 | master | NameNode、Secondary NameNode、Resource Manager |
   | 192.168.139.101 | slave1 | Datanode、NodeManager                          |
   | 192.168.139.102 | slave2 | Datanode、NodeManager                          |

2. 安装项目

   > Zookeeper-3.4.6
   >   Hadoop-2.7.1
   >   HBase-1.2.1
   >   Hive-2.3.4

3. 操作系统版本

   > 安装的操作系统是CentOS 7

   ```
   [root@master ~]# cat /etc/redhat-release
   CentOS Linux release 7.6.1810 (Core)
   ```

4. IP地址和DNS

   ```
   master服务器:[root@master ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
   BOOTPROTO="static"
   IPADDR=192.168.139.100
   NETMASK=255.255.255.0
   GATEWAY=192.168.139.2
   DNS1=8.8.8.8
   DNS2=8.8.8.4
   
   slave1服务器:[root@slave1 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
   BOOTPROTO="static"
   IPADDR=192.168.139.101
   NETMASK=255.255.255.0
   GATEWAY=192.168.139.2
   DNS1=8.8.8.8
   DNS2=8.8.8.4
   
   slave2服务器:[root@slave2 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
   BOOTPROTO="static"
   IPADDR=192.168.139.102
   NETMASK=255.255.255.0
   GATEWAY=192.168.139.2
   DNS1=8.8.8.8
   DNS2=8.8.8.4
   ```

5. 修改/etc/hosts，三台服务器配置一样，以master服务器为例

   ```
   [root@master ~]# cat /etc/hosts
   127.0.0.1        localhost
   192.168.139.100  master
   192.168.139.101  slave1
   192.168.139.102  slave2
   ```

6. 关闭防火墙和selinux，三台服务器配置一样，以master服务器为例

   ```
   查看防火墙状态
   [root@master ~]# systemctl status firewalld
   Active: active (running) 开启状态
   关闭防火墙
   [root@master ~]# systemctl stop firewalld
   禁用防火墙
   [root@master ~]# systemctl disable firewalld
   
   查看selinux状态
   [root@master ~]# getenforce
   永久关闭selinux
   [root@master ~]# vi /etc/selinux/config
   SELINUX=disabled   # change to disabled
   重启系统
   [root@master ~]# reboot
   ```

7. 时间同步，三台服务器配置一样，以master服务器为例

   ```
   查看是否安装ntp
   [root@master ~]# rpm -qa |grep ntp
   如果没有安装，用yum安装
   [root@master ~]# yum install ntp
   指定时间同步
   ntpdate time.windows.com
   ```

8. 创建用户并配置sudo权限，三台服务器配置一样，以master服务器为例

   ```
   创建组
   [root@master ~]# groupadd hadoop
   创建用户并指定用户所属组
   [root@master ~]# useradd hadoop -g hadoop
   修改用户密码
   passwd hadoop
   
   [root@master ~]# visudo
   # add at the last line: user 'hadoop' can use all root privilege
   hadoop    ALL=(ALL)       ALL
   ```

9. 安装jdk，配置环境变量，三台服务器配置一样，以master服务器为例

   ```
   将jdk-8u77-linux-x64.rpm拷贝到/home/hadoop/中
   [root@master hadoop]# rpm -ivh /home/hadoop/jdk-8u77-linux-x64.rpm
   切换到hadoop用户
   [hadoop@master ~]$ vi ~/.bash_profile
   
   export JAVA_HOME=/usr/java/jdk1.8.0_77
   export PATH=$PATH:$JAVA_HOME/bin
   export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
   
   source ~/.bash_profile
   验证jdk安装成功
   [hadoop@master ~]$ java -version
   ```

10. 配置ssh无密码登录

    ```
    创建密钥对，输入ssh-keygen -t rsa，一路回车
    [hadoop@master ~]$ ssh-keygen -t rsa
    [hadoop@slave1 ~]$ ssh-keygen -t rsa
    [hadoop@slave2 ~]$ ssh-keygen -t rsa
    秘钥生成后在~/.ssh/目录下，有两个文件id_rsa(私钥)和id_rsa.pub（公钥）
    
    将master服务器的公钥复制到authorized_keys
    [hadoop@master ~]$ cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
    
    合并公钥,将三台服务器的id_rsa.pub合并到master服务器的authorized_keys中
    [hadoop@master ~]$ cat  ~/.ssh/id_rsa.pub 
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRQo7+LxndxDYyBGJZGD+0jC7LBFfGDlxWCh0aK4OvNVrs0Sf6+B/Xq3us3SJpX6YdeEgyCzfInw8L2pe1cu+XZpYuVoVevs/5bT9G3Wqtb8Ixk7nOP8EVDY5LNfDlhPp2iehxCVodpUVN09KyGPgCE8ffN4LRgVJ7SUZibtBmHn/Vdog09yZqjlYx845ZP8M8ifSMjcAQ0voSvRvWFTMvqqx5vpqDHkZhM5CA/+VOKgeNyUphV79o6SDnIHCxnLqG+m8v9U6sMRx2N4d+c59Pw5b1g112gKtF/Y86+EbQOMda8fYJMgkTnIJnKhPPOcEun/CdtinNYW1CP4pQsgCb hadoop@master
    
    [hadoop@slave1 ~]$ cat  ~/.ssh/id_rsa.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCdc3xdmHR4qY5lf0EyhD6cHRSOCOkp0H3heCeLso11hkSMVxowRZg+YCoRwTqfEWD+VkVGyws9EkRjLXXKJ8ND0l+1urlVzpM3tXMZeRfqF5YBHGlIQeAJ4lk1MnR9jev1TbITcKQLVA5GzAZx5MVQxE418UYnLYG2pLS1xIzMyFgr0CwpfpwA7owriclTIQOCa4nieUxYJDLdxmBsumBgPRp+195MCr1zrzwtb3UMEP+DLTHAw3+RBB15QoF3HEVVVhWSAN9ZjYjnit20dPwFnM2amGqpRS4p5wLxwkeyEsrdXkNsrtW+XbhtuxYo0YeusEp4spFhOTmv7lPhdfkv hadoop@slave1
    
    [hadoop@slave2 ~]$ cat  ~/.ssh/id_rsa.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDtoqqsF2s8FfffiY7sZ545anJ69kf914nENQjwJd2GkgZSUujX4DJnXDSthyLUGr22cN3iaUmATQPm6qzpus/rag+QKms8fHMkaCLG6qeS9CCJZunJ2uWdTAd25H1d9RyTReTQ9tPqHZ4dFXlXeY4QYxiKHVCkOkXWrWLqCcKilDLkU7Z5HiqfPW23pt4WDmXerCIDZbM55152pcK3UF9AwFTBFA0ofQaq7IhrVHSpEsTHYQNWQxgEGYqyUT+ep3bGtvfu6a0621DIUScGlRsptrIFTFZPtO5Enih62VY/aKrA/YhHq1slTsh/tDSs29r6ZVtPNyNF0Ruu0kmPRveR hadoop@slave2
    
    master服务器中authorized_keys的内容如下
    [hadoop@master ~]$ cat ~/.ssh/authorized_keys 
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCakZ3tpFGp1rgwpRCtdWnXH8mdaBdESwBgrMRftGi3Dj9Lh1icOnLozXmuHaZnm4DVlqkgtlZtq+KDVu2picJ1vPtM9mpCCFAwrYiLsxB+f6bETiWUjZhYJqaTCbxdQIOyHvgeFMqe/nQfSC2c+RXyd/FD6IqV4Im02khEp820pfqVhChgXibk6czORvhC08ChfafCvH1YXaKCKp7lHKzA3iDUdaYTWNK8PdzTx8E6eEdH2w7yOXyVpnsfQS0oercsn5euiEdmXM3XkAnMPD1toWw5yyIprvwO7q9OUXFdMLXzeJILQ/TfeZA/9q4onDnOyqBGgpRfJVGpE1JT9Er/ hadoop@master
    
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCdc3xdmHR4qY5lf0EyhD6cHRSOCOkp0H3heCeLso11hkSMVxowRZg+YCoRwTqfEWD+VkVGyws9EkRjLXXKJ8ND0l+1urlVzpM3tXMZeRfqF5YBHGlIQeAJ4lk1MnR9jev1TbITcKQLVA5GzAZx5MVQxE418UYnLYG2pLS1xIzMyFgr0CwpfpwA7owriclTIQOCa4nieUxYJDLdxmBsumBgPRp+195MCr1zrzwtb3UMEP+DLTHAw3+RBB15QoF3HEVVVhWSAN9ZjYjnit20dPwFnM2amGqpRS4p5wLxwkeyEsrdXkNsrtW+XbhtuxYo0YeusEp4spFhOTmv7lPhdfkv hadoop@slave1
    
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDtoqqsF2s8FfffiY7sZ545anJ69kf914nENQjwJd2GkgZSUujX4DJnXDSthyLUGr22cN3iaUmATQPm6qzpus/rag+QKms8fHMkaCLG6qeS9CCJZunJ2uWdTAd25H1d9RyTReTQ9tPqHZ4dFXlXeY4QYxiKHVCkOkXWrWLqCcKilDLkU7Z5HiqfPW23pt4WDmXerCIDZbM55152pcK3UF9AwFTBFA0ofQaq7IhrVHSpEsTHYQNWQxgEGYqyUT+ep3bGtvfu6a0621DIUScGlRsptrIFTFZPtO5Enih62VY/aKrA/YhHq1slTsh/tDSs29r6ZVtPNyNF0Ruu0kmPRveR hadoop@slave2
    
    将master中的authoized_keys远程到slave1和slave2中
    [hadoop@master ~]$ scp ~/.ssh/authorized_keys slave1:~/.ssh/
    [hadoop@master ~]$ scp ~/.ssh/authorized_keys slave2:~/.ssh/
    
    验证是否免密登录（第一次登录会有提示）,三台服务器都验证，如下以master为例
    [hadoop@master ~]$ ssh localhost
    [hadoop@master ~]$ ssh master
    [hadoop@master ~]$ ssh slave1
    [hadoop@master ~]$ ssh slave2
    ```

11. 安装mysql5.7

    ~~~
    mysql只安装在master服务器中
    添加mysql用户和组
    [root@master ~]# groupadd mysql
    [root@master ~]# useradd -g mysql mysql
    [root@master ~]# passwd mysql
    将mysql用户的密码修改为mysql
    
    将mysql-5.7.19-linux-glibc2.12-x86_64.tar.gz拷贝到master中
    创建mysql软件目录
    [root@master ~]# mkdir -p /usr/local/mysql-5.7.19
    将软件目录属主修改为mysql
    [root@master ~]# chown -R mysql:mysql /usr/local/mysql-5.7.19/
    创建mysql数据目录
    [root@master ~]# mkdir -p /var/lib/mysql/data
    创建mysql日志目录
    [root@master ~]# mkdir -p /var/lib/mysql/logs
    [root@master ~]# touch /var/lib/mysql/logs/mariadb.log
    [root@master ~]# touch /var/lib/mysql/logs/mariadb.pid
    将数据目录属主修改为mysql
    [root@master ~]# chown -R mysql:mysql /var/lib/mysql
    [root@master ~]# chmod -R 777 /var/lib/mysql
    设置文件my.cnf的权限
    [root@master ~]# chmod 644 /etc/my.cnf
    [root@master ~]# ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
    解压mysql
    [root@master ~]# tar -zxvf /home/mysql/mysql-5.7.19-linux-glibc2.12-x86_64.tar.gz -C /usr/local/mysql-5.7.19 --strip-components 1
    编辑my.conf
    [root@master ~]# vi /etc/my.cnf
    [mysqld]
    datadir=/var/lib/mysql/data
    socket=/var/lib/mysql/mysql.sock
    basedir=/usr/local/mysql-5.7.19
    port=3306
    user=mysql
    
    [mysqld_safe]
    log-error=/var/lib/mysql/logs/mariadb.log
    pid-file=/var/lib/mysql/logs/mariadb.pid
    
    进入mysql软件目录
    [root@master ~]# cd /usr/local/mysql-5.7.19
    初始化数据库
    [root@master mysql-5.7.19]# ./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql-5.7.19 --datadir=/var/lib/mysql/data
    将会出现如下提示
    [Note] A temporary password is generated for root@localhost: W/&Itv7Y%f1,
    用户root@localhost被赋予随机生成的一个密码，（W/&Itv7Y%f1,），该密码直接写在日志文件中，路径为
    ~/.mysql_secret
    
    启动数据库
    [root@master mysql-5.7.19]# ./bin/mysqld --defaults-file=/etc/my.cnf
    该窗口将一直运行，重新启动一个窗口
    [root@master ~]# cd /usr/local/mysql-5.7.19/
    [root@master mysql-5.7.19]# ./bin/mysql -uroot -p
    输入上面的密码（W/&Itv7Y%f1,）
    修改root用户的密码
    mysql> alter user user() identified by "mysql";
    
    添加启动服务
    [root@master mysql-5.7.19]# cp support-files/mysql.server /etc/init.d/mysqld
    [root@master mysql-5.7.19]# chown -R mysql:mysql /etc/init.d/mysqld
    赋予可执行权限
    [root@master mysql-5.7.19]# chmod +x /etc/init.d/mysqld
    添加服务
    [root@master mysql-5.7.19]# chkconfig --add mysqld
    [root@master mysql-5.7.19]# chkconfig --level 6 mysqld on
    显示服务列表
    chkconfig --list
    
    环境变量配置
    [root@master mysql-5.7.19]# ln -s /usr/local/mysql-5.7.19 /usr/local/mysql
    [root@master mysql-5.7.19]# vi /etc/profile
    在最后添加如下行
    export PATH=/usr/local/mysql/bin:$PATH
    配置生效
    [root@master mysql-5.7.19]# source /etc/profile
    
    重启
    [root@master mysql-5.7.19]# reboot
    检查mysql是否启动
    [root@master bin]# systemctl status mysqld
    
    创建数据库用户并授权
    [hadoop@master ~]$ mysql -uroot -pmysql
    mysql> create user hive identified by 'hive';
    mysql> grant all privileges on *.* to 'hive'@'%' with grant option;
    mysql> flush privileges;
    mysql> exit;
    
    创建数据库
    [hadoop@master ~]$ mysql -h192.168.139.100 -uhive -phive
    mysql> create database hive;
    ~~~

# 部署hadoop

1. 解压hadoop

   ~~~
   [hadoop@master ~]$ tar -zxvf hadoop-2.7.1.tar.gz
   检查hadoop是否可用,显示版本就是正常
   [hadoop@master ~]$ ./hadoop-2.7.1/bin/hadoop version
   ~~~

2. 创建目录，三台服务器都执行，以master服务器为例

   ~~~
   [hadoop@master ~]$ mkdir -p /home/hadoop/hadoop_data
   [hadoop@master ~]$ mkdir -p /home/hadoop/hadoop_data
   [hadoop@master ~]$ mkdir -p /home/hadoop/hadoop_name
   [hadoop@master ~]$ mkdir -p /home/hadoop/hadoop_tmp
   [hadoop@master ~]$ mkdir -p /home/hadoop/hbase_tmp
   [hadoop@master ~]$ mkdir -p /home/hadoop/zookeeper_data
   [hadoop@master ~]$ mkdir -p /home/hadoop/zookeeper_logs
   [hadoop@master ~]$ sudo mkdir -p /var/hadoop/pids
   [hadoop@master ~]$ sudo chown -R hadoop:hadoop /var/hadoop
   [hadoop@master ~]$ sudo chmod -R 777 /var/hadoop
   ~~~

3. 添加环境变量，三台服务器都执行，以master服务器为例

   ~~~
   [hadoop@master ~]$ vi ~/.bash_profile
   export HADOOP_HOME=/home/hadoop/hadoop-2.7.1
   export HADOOP_INSTALL=$HADOOP_HOME
   export HADOOP_COMMON_HOME=$HADOOP_HOME
   export HADOOP_HDFS_HOME=$HADOOP_HOME
   export HADOOP_MAPRED_HOME=$HADOOP_HOME
   export YARN_HOME=$HADOOP_HOME
   export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
   export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
   export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
   export HDFS_CONF_DIR=${HADOOP_HOME}/etc/hadoop
   export YARN_CONF_DIR=${HADOOP_HOME}/etc/hadoop
   
   export ZOOKEEPER=/home/hadoop/zookeeper-3.4.6
   export HIVE_HOME=/home/hadoop/hive-2.3.4
   export HBASE_HOME=/home/hadoop/hbase-1.2.1
   export PIG_HOME=/home/hadoop/pig-0.17.0
   export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin:$HIVE_HOME/bin:$ZOOKEEPER/bin:$PIG_HOME/bin
   
   [hadoop@master ~]$ source ~/.bash_profile
   ~~~

4. 修改hadoop-env.sh，修改如下三个参数

   ~~~
   [hadoop@master ~]$ cd hadoop-2.7.1/etc/hadoop/
   
   export JAVA_HOME=/usr/java/jdk1.8.0_77
   export HADOOP_CONF_DIR=/home/hadoop/hadoop-2.7.1/etc/hadoop
   export HADOOP_PID_DIR=/var/hadoop/pids
   ~~~

5. 修改core-site.xml

   ~~~
   <configuration>
   <property>
           <name>fs.defaultFS</name>
           <value>hdfs://master:9000</value>
   </property>
   <property>
           <name>hadoop.tmp.dir</name>
           <value>file:/home/hadoop/hadoop_tmp</value>
   </property>
   <property>
       <name>io.file.buffer.size</name>
       <value>131702</value>
   </property>
   <property>
       <name>hadoop.proxyuser.hduser.hosts</name>
       <value>*</value>
   </property>
   <property>
       <name>hadoop.proxyuser.hduser.groups</name>
       <value>*</value>
   </property>
   <property>
       <name>ha.zookeeper.quorum</name>
       <value>master:2181,slave1:2181,slave2:2181</value>
   </property>
   
   hdfs-site.xml
   <property>
           <name>dfs.ha.automatic-failover.enabled</name>
           <value>true</value>
   </property>
   <property>
           <name>dfs.namenode.name.dir</name>
           <value>file:/home/hadoop/hadoop_name</value>
   </property>
   <property>
           <name>dfs.datanode.data.dir</name>
           <value>file:/home/hadoop/hadoop_data</value>
   </property>
   <property>
           <name>dfs.replication</name>
           <value>2</value>
   </property>
   <property>
           <name>dfs.namenode.secondary.http-address</name>
           <value>master:9001</value>
   </property>
   <property>
           <name>dfs.webhdfs.enabled</name>
           <value>true</value>
   </property>
   <property>
           <name>ha.zookeeper.quorum</name>
           <value>master:2181,slave1:2181,slave2:2181</value>
   </property>
   </configuration>
   </property>
   <property>
       <name>hadoop.proxyuser.hduser.hosts</name>
       <value>*</value>
   </property>
   <property>
       <name>hadoop.proxyuser.hduser.groups</name>
       <value>*</value>
   </property>
   <property>
       <name>ha.zookeeper.quorum</name>
       <value>master:2181,slave1:2181,slave2:2181</value>
   </property>
   </configuration>
   ~~~

6. 修改hdfs-site.xml

   ~~~
   <configuration>
   <property>
           <name>dfs.ha.automatic-failover.enabled</name>
           <value>true</value> 
   </property> 
   <property>
           <name>dfs.namenode.name.dir</name>
           <value>file:/home/hadoop/hadoop_name</value>
   </property>
   <property>
           <name>dfs.datanode.data.dir</name>
           <value>file:/home/hadoop/hadoop_data</value>
   </property>
   <property>
           <name>dfs.replication</name>
           <value>2</value>
   </property>
   <property>
           <name>dfs.namenode.secondary.http-address</name>
           <value>master:9001</value>
   </property>
   <property>
           <name>dfs.webhdfs.enabled</name>
           <value>true</value>
   </property>
   <property> 
           <name>ha.zookeeper.quorum</name> 
           <value>master:2181,slave1:2181,slave2:2181</value> 
   </property>
   </configuration>
   ~~~

7. 修改mapred-site.xml

   ~~~
   [hadoop@master hadoop]$ cp mapred-site.xml.template mapred-site.xml
   <configuration>
   <property>
     <name>mapreduce.framework.name</name>
     <value>yarn</value>
   </property>
   <property>     
     <name>mapreduce.jobhistory.address</name>   
     <value>master:10020</value>  
   </property>
   <property>     
     <name>mapreduce.jobhistory.webapp.address</name>   
     <value>master:19888</value>  
   </property>
   </configuration>
   ~~~

8. 修改yarn-site.xml

   ~~~
   <configuration>
   <property>     
     <name>yarn.nodemanager.aux-services</name>   
     <value>mapreduce_shuffle</value>  
   </property>
   <property>     
     <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>   
     <value>org.apache.hadoop.mapred.ShuffleHandler</value>  
   </property>
   <property>     
     <name>yarn.resourcemanager.address</name>   
     <value>master:8032</value>  
   </property>
   <property>     
     <name>yarn.resourcemanager.scheduler.address</name>   
     <value>master:8030</value>  
   </property>
   <property>     
     <name>yarn.resourcemanager.resource-tracker.address</name>   
     <value>master:8031</value>  
   </property>
   <property>     
     <name>yarn.resourcemanager.admin.address</name>   
     <value>master:8033</value>  
   </property>
   <property>     
     <name>yarn.resourcemanager.webapp.address</name>   
     <value>master:8088</value>  
   </property>
   <property> 
           <name>ha.zookeeper.quorum</name> 
           <value>master:2181,slave1:2181,slave2:2181</value> 
   </property>
   <property>
     <name>yarn.resourcemanager.zk-state-store.address</name>
     <value>master:2181,slave1:2181,slave2:2181</value>
   </property>
   <property>
     <name>yarn.resourcemanager.store.class</name>
     <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
   </property>
   <property>
     <name>yarn.resourcemanager.zk-address</name>
     <value>master:2181,slave1:2181,slave2:2181</value>
   </property>
   </configuration>
   ~~~

9. 修改slaves 

   ~~~
   slave1
   slave2
   ~~~

10. 同步所有节点

    ~~~
    [hadoop@master hadoop]$ scp -r /home/hadoop/hadoop-2.7.1 slave1:/home/hadoop/
    [hadoop@master hadoop]$ scp -r /home/hadoop/hadoop-2.7.1 slave2:/home/hadoop/
    ~~~

    # 部署zookeeper

11. 解压

    ~~~
    [hadoop@master hadoop]$ cd
    [hadoop@master ~]$ tar -zxvf zookeeper-3.4.6.tar.gz
    ~~~

12. 编辑zoo.cfg

    ~~~
    [hadoop@master ~]$ cd zookeeper-3.4.6/conf/
    [hadoop@master conf]$ cp zoo_sample.cfg zoo.cfg
    
    tickTime=2000
    initLimit=5
    syncLimit=2
    dataDir=/home/hadoop/zookeeper_data
    dataLogDir=/home/hadoop/zookeeper_logs
    clientPort=2181
    server.1=master:2888:3888
    server.2=slave1:2888:3888
    server.3=slave2:2888:3888
    ~~~

13. 编辑myid

    ~~~
    master服务器上执行
    [hadoop@master conf]$ echo 1 > /home/hadoop/zookeeper_data/myid
    
    slave1服务器上执行
    [hadoop@slave1 ~]$ echo 2 > /home/hadoop/zookeeper_data/myid
    
    slave2服务器上执行
    [hadoop@slave1 ~]$ echo 3 > /home/hadoop/zookeeper_data/myid
    ~~~

14. 同步所有节点

    ~~~
    [hadoop@master conf]$ scp -r /home/hadoop/zookeeper-3.4.6 slave1:/home/hadoop/
    [hadoop@master conf]$ scp -r /home/hadoop/zookeeper-3.4.6 slave2:/home/hadoop/
    ~~~

    # 部署hbase

    1. 解压Hbase

       ~~~
       [hadoop@master conf]$ cd
       [hadoop@master ~]$ tar -zxvf hbase-1.2.1-bin.tar.gz 
       ~~~

    2. 配置regionservers

       ~~~
       [hadoop@master ~]$ cd hbase-1.2.1/conf/
       slave1
       slave2
       ~~~

    3. 编辑hbase-site.xml

       ~~~
       <configuration>
       <property>
               <name>hbase.rootdir</name>
               <value>hdfs://master:9000/hbase</value>
       </property>
       <property>
               <name>hbase.cluster.distributed</name>
               <value>true</value>
       </property>
       <property>
               <name>hbase.tmp.dir</name>
               <value>/home/hadoop/hbase_tmp</value>
       </property>
       <property>
               <name>hbase.zookeeper.quorum</name>
               <value>master,slave1,slave2</value>
       </property>
       <property>
               <name>hbase.zookeeper.property.clientPort</name>
               <value>2181</value>
       </property>
       <property>
               <name>hbase.zookeeper.property.dataDir</name>
               <value>/home/hadoop/zookeeper_data</value>
       </property>
       </configuration>
       ~~~

    4. 编辑hbase-env.sh

       ~~~
       export JAVA_HOME=/usr/java/jdk1.8.0_77
       export HBASE_MANAGES_ZK=false
       export HBASE_CLASSPATH=/home/hadoop/hadoop-2.7.1/etc/hadoop
       ~~~

    5. 同步所有节点

       ~~~
       [hadoop@master conf]$ scp -r /home/hadoop/hbase-1.2.1 slave1:/home/hadoop/
       [hadoop@master conf]$ scp -r /home/hadoop/hbase-1.2.1 slave2:/home/hadoop/
       ~~~

       # 部署hive

       1. 解压Hive

          ~~~
          [hadoop@master conf]$ cd
          [hadoop@master ~]$ tar -zxvf apache-hive-2.3.4-bin.tar.gz
          [hadoop@master ~]$ mv apache-hive-2.3.4-bin hive-2.3.4
          ~~~

       2. 编辑hive-env.sh

          ~~~
          [hadoop@master ~]$ cd hive-2.3.4/conf/
          [hadoop@master conf]$ cp hive-env.sh.template hive-env.sh
          ~~~

          

       3. 编辑hive-site.xml

          ~~~
          [hadoop@master conf]$ cp hive-default.xml.template hive-site.xml
          <configuration>
          <property>
                  <name>hive.metastore.warehouse.dir</name>
                  <value>hdfs://master:9000/user/hive/warehouse</value>
          </property>
          <property>
                  <name>datanucleus.readOnlyDatastore</name>
                  <value>false</value>
          </property>
          <property>
                  <name>datanucleus.fixedDatastore</name>
                  <value>false</value>
          </property>
          <property>
                  <name>datanucleus.autoCreateSchema</name>
                  <value>true</value>
          </property>
          <property>
                  <name>datanucleus.autoCreateTables</name>
                  <value>true</value>
          </property>
          <property>
                  <name>datanucleus.autoCreateColumns</name>
                  <value>true</value>
          </property>
          <property>
                  <name>javax.jdo.option.ConnectionURL</name>
                  <value>jdbc:mysql://192.168.139.100:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
          </property>
          <property>
                  <name>javax.jdo.option.ConnectionDriverName</name>
                  <value>com.mysql.jdbc.Driver</value>
          </property>
          <property>
                  <name>javax.jdo.option.ConnectionUserName</name>
                  <value>hive</value>
          </property>
          <property>
                  <name>javax.jdo.option.ConnectionPassword</name>
                  <value>hive</value>
          </property>
          <property>
                  <name>hive.metastore.uris</name>
                  <value>thrift://master:9083</value>
          </property>
          <property>
                  <name>hive.zookeeper.quorum</name>
              <value>master,slave1,slave2</value>
          </property>
          <property>
              <name>datanucleus.schema.autoCreateAll</name>
              <value>true</value>
          </property>
          </configuration>
          ~~~

       4. 添加mysql驱动

          ~~~
          [hadoop@master conf]$ cd
          [hadoop@master conf]$ cp /home/hadoop/mysql-connector-java.jar /home/hadoop/hive-2.3.4/lib/
          ~~~

          

       5. 复制jar包

          ~~~
          [hadoop@master conf]$ cd
          [hadoop@master ~]$ cp /home/hadoop/hbase-1.2.1/lib/hbase-client-1.2.1.jar /home/hadoop/hive-2.3.4/lib/
          [hadoop@master ~]$ cp /home/hadoop/hbase-1.2.1/lib/hbase-common-1.2.1.jar /home/hadoop/hive-2.3.4/lib/
          [hadoop@master ~]$ scp $JAVA_HOME/lib/tools.jar /home/hadoop/hive-2.3.4/lib/
          
          [hadoop@master ~]$ cp /home/hadoop/hive-2.3.4/lib/jline-2.12.jar /home/hadoop/hadoop-2.7.1/share/hadoop/yarn/lib/
          [hadoop@master ~]$ scp /home/hadoop/hive-2.3.4/lib/jline-2.12.jar slave1:/home/hadoop/hadoop-2.7.1/share/hadoop/yarn/lib/
          [hadoop@master ~]$ scp /home/hadoop/hive-2.3.4/lib/jline-2.12.jar slave2:/home/hadoop/hadoop-2.7.1/share/hadoop/yarn/lib/
          ~~~

       6. 同步所有节点

          ~~~
          [hadoop@master conf]$ scp -r /home/hadoop/hive-2.3.4 slave1:/home/hadoop/
          [hadoop@master conf]$ scp -r /home/hadoop/hive-2.3.4 slave2:/home/hadoop/
          ~~~

          # 启动

          1. 启动zookeeper，三台服务器都执行，如下以master为例

             ~~~
             [hadoop@master ~]$ zkServer.sh start
             ~~~

             > 输入jps，会显示启动进程
             >
             > QuorumPeerMain

          2. 启动hadoop，只在master执行

             > 首次启动需要先在 Master 节点执行 NameNode 的格式化

             ~~~
             [hadoop@master ~]$ hdfs namenode -format
             日志最后几行会显示如下信息:
             Storage directory /home/hadoop/hadoop_name has been successfully formatted
             启动hadoop
             [hadoop@master ~]$ start-all.sh
             ~~~

             > 浏览器输入如下地址查看集群明细
             >
             > [名称节点信息](http://192.168.139.100:50070)
             >
             > [集群信息](http://192.168.139.100:8088/)
             >
             > 输入jps，master会显示启动进程
             >
             > QuorumPeerMain、NameNode、SecondaryNameNode、ResourceManager
             >
             > 输入jps，slave1、slave2会显示启动进程
             >
             > QuorumPeerMain、DataNode、NodeManager

          3. 启动hbase，只在master执行

             ~~~
             [hadoop@master ~]$ start-hbase.sh
             ~~~

             > 输入jps，master会显示启动进程
             >
             > QuorumPeerMain、NameNode、SecondaryNameNode、ResourceManager、HMaster
             >
             > 输入jps，slave1、slave2会显示启动进程
             >
             > QuorumPeerMain、DataNode、NodeManager、HRegionServer

             ~~~
             进入Hbase命令行
             [hadoop@master ~]$ hbase shell
             查看表
             hbase(main):001:0> list
             创建表'mytable'，含有一个列族hb
             hbase(main):003:0> create 'mytable','hb'
             添加行
             hbase(main):008:0> put 'mytable','first','hb:data','hello HBase'
             查看表中所有数据
             hbase(main):013:0> scan 'mytable'
             按照行键查询数据
             hbase(main):014:0> get 'mytable','first'
             ~~~

          4. 启动hive，启动方式有5种

             ~~~
             在hdfs上创建hive存储数据目录
             查看hdfs的数据目录
             [hadoop@master ~]$ hadoop fs -ls /
             [hadoop@master ~]$ hadoop fs -mkdir /tmp
             [hadoop@master ~]$ hadoop fs -mkdir -p /user/hadoop/hive/warehouse
             元数据初始化
             [hadoop@master ~]$ schematool -dbType mysql -initSchema
             以metastore方式启动hive，nohup是指不挂断(用户退出)，&是指后台运行
             [hadoop@master ~]$ nohup hive --service metastore &> /home/hadoop/metastore.log &
             输入jps，会显示RunJar进程
             hive命令行连接
             [hadoop@master ~]$ hive
             查看进程会有两个RunJar进程
             hive> show databases;
             会显示所有的数据库
             ~~~

          5. 测试

             ~~~
             创建输入、输出目录
             [hadoop@master ~]$ hdfs dfs -mkdir /in
             [hadoop@master ~]$ hdfs dfs -mkdir /out
             将本地文件拷贝到hdfs中
             [hadoop@master ~]$ hdfs dfs -copyFromLocal /home/hadoop/hadoop-2.7.1/LICENSE.txt /in
             用wordcount分析LICENSE.txt
             [hadoop@master ~]$ hadoop jar /home/hadoop/hadoop-2.7.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar wordcount /in/LICENSE.txt /out
             查看结果
             [hadoop@master ~]$ hdfs dfs -cat /out/part-r-00000
             ~~~

             