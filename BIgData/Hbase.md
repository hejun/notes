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
  - hbase-env.sh
    ```
    # vi conf/hbase-env.sh

    export JAVA_HOME=/hejun/software/jdk1.8.0_271     # 打开 export JAVA_HOME 并修改
    export HBASE_LOG_DIR=/hejun/logs/hbase            # 打开 export HBASE_LOG_DIR 并修改
    export HBASE_MANAGES_ZK=false                     # 打开 export HBASE_MANAGES_ZK 并修改
    ```
  - hbase-site.xml
    ```
    # vi conf/hbase-site.xml 清空 configuration 子节点后在 configuration 中添加

    <property>
      <name>hbase.rootdir</name>
      <value>hdfs://<master-hostname>:9000/hbase</value>
    </property>
    <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
    </property>
    <property>
      <name>hbase.zookeeper.quorum</name>
      <value><zookeeper-ip>:2181</value>              # 多个用逗号分隔
    </property>
    ```
  - regionservers (分布式环境才配置)
    ```
    # vi conf/regionservers
    # 删除 localhost, 改为节点 hostname
    ```
4. 设置开机自启  (只在主节点做, 其他节点会跟随主节点启动)
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
5. 启动&停止 (只在主节点做, 其他节点会跟随主节点启动)
  ```
  # 启动 Hbae
  systemctl start hbase
  # 停止 Hbae
  systemctl stop hbase
  ```
6. 开启端口 (只在主节点做)
  ```
  firewall-cmd --zone=public --add-port=16000/tcp --add-port=16010/tcp --permanent
  firewall-cmd --reload
  ```
