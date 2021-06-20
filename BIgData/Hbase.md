# Hbase (2.3.5)

1. 下载 [Hbase](https://archive.apache.org/dist/hbase/2.3.5/hbase-2.3.5-bin.tar.gz)
2. 解压
  ```
  tar -zxvf hbase-2.3.5-bin.tar.gz -C /hejun/software
  ```
3. 修改配置文件
  ```
  cd /hejun/software/hbase-2.3.5
  ```
  ```
  # vi conf/hbase-env.sh
  
  export JAVA_HOME=/hejun/software/jdk1.8.0_271     # 打开 export JAVA_HOME 并修改
  export HBASE_LOG_DIR=/hejun/logs/hbase            # 打开 export HBASE_LOG_DIR 并修改
  export HBASE_MANAGES_ZK=false                     # 打开 export HBASE_MANAGES_ZK 并修改
  ```
  ```
  # vi conf/hbase-site.xml 在 configuration 中添加
  
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://<ip>:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value><ip>:2181</value>
  </property>
  <property>
    <name>hbase.master.info.port</name>
    <value>60010</value>
   </property>
  ```
4. 设置开机自启
  ```
  # vi /usr/lib/systemd/system/hbase.service
  
  [Unit]
  Description=Hbase
  After=zookeeper.target dfs.target

  [Service]
  User=root
  Group=root
  Type=forking
  ExecStart=/hejun/software/hbase-2.3.5/bin/start-hbase.sh
  ExecStop=/hejun/software/hbase-2.3.5/bin/stop-hbase.sh
  PIDFile=/tmp/hbase-root-master.pid

  [Install]
  WantedBy=multi-user.target
  ```
  ```
  systemctl daemon-reload
  systemctl enable hbase
  ```
5. 启动&停止
  ```
  # 启动 Hbae
  systemctl start hbase
  # 停止 Hbae
  systemctl stop hbase
  ```
6. 开启端口
  ```
  firewall-cmd --zone=public --add-port=60010/tcp --permanent
  firewall-cmd --reload
  ```
