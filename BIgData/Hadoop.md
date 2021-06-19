# Hadoop (3.1.1)

1. 下载 [Hadoop](https://archive.apache.org/dist/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz)
2. 解压
  ```
  tar -zxvf hadoop-3.1.1.tar.gz -C /hejun/software
  ```
3. 修改配置文件
  - 修改hosts
  ```
  # vi /ets/hosts
  # 在 127.0.0.1 这行后面加上 hostname 命令查出来的值
  ```
  - 生成 ssh
  ```
  ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  chmod 0600 ~/.ssh/authorized_keys
  ```
  - 修改配置文件
  ```
  # vi etc/hadoop/hadoop-env.sh
  
  export JAVA_HOME=/hejun/software/jdk1.8.0_271     # 打开 export JAVA_HOME 并修改
  export HADOOP_HOME=/hejun/software/hadoop-3.1.1   # 打开 export HADOOP_HOME 并修改
  export HADOOP_LOG_DIR=/hejun/logs/hadoop          # 打开 export HADOOP_LOG_DIR 并修改
  export HDFS_NAMENODE_USER=root                    # 打开 HDFS_NAMENODE_USER 并修改 root
  export HDFS_DATANODE_USER=root                    # 新增
  export HDFS_SECONDARYNAMENODE_USER=root           # 新增
  ```
  ```
  # vi etc/hadoop/yarn-env.sh
  
  export YARN_RESOURCEMANAGER_USER=root
  export YARN_NODEMANAGER_USER=root
  ```
  ```
  # vi etc/hadoop/core-site.xml 在 configuration 中添加
  
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://<ip>:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/hejun/data/hadoop</value>
  </property>
  ```
  ```
  # vi etc/hadoop/hdfs-site.xml 在 configuration 中添加
  
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  ```
  ```
  # vi etc/hadoop/mapred-site.xml
  
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  ```
  ```
  # vi etc/hadoop/yarn-site.xml 在 configuration 中添加
  
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  ```
4. 格式化文件系统
  ```
  ./bin/hdfs namenode -format
  ```
5. 设置开机自启
  ```
  # vi /usr/lib/systemd/system/hadoop.service
  
  [Unit]
  Description=Hadoop
  After=network.target

  [Service]
  Type=forking
  ExecStart=/hejun/software/hadoop-3.1.1/sbin/start-dfs.sh
  ExecStop=/hejun/software/hadoop-3.1.1/sbin/stop-dfs.sh

  [Install]
  WantedBy=multi-user.target
  ```
  ```
  systemctl daemon-reload
  systemctl enable hadoop
  ```
6. 启动&停止
  ```
  # 启动
  systemctl start hadoop
  # 停止
  systemctl stop hadoop
  ```
7. 开启端口
  ```
  firewall-cmd --zone=public --add-port=9870/tcp --permanent
  firewall-cmd --reload
  ```
