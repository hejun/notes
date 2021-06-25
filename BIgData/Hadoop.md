# Hadoop (3.2.0)

1. 下载 [Hadoop](https://archive.apache.org/dist/hadoop/common/hadoop-3.2.0/hadoop-3.2.0.tar.gz)
2. 解压
  ```
  tar -zxvf hadoop-3.2.0.tar.gz -C /hejun/software
  ```
3. 修改配置文件
  ```
  cd /hejun/software/hadoop-3.2.0
  ```
  - 修改hosts
    ```
    # vi /etc/hosts
    # 在 127.0.0.1 这行后面加上 hostname 命令查出来的值
    ```
  - 生成 ssh
    - 单机环境
      ```
      ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
      cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
      chmod 0600 ~/.ssh/authorized_keys
      ```
    - 分布式环境
      ```
      ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
      
      ssh-copy-id -i ~/.ssh/id_rsa.pub <hostname | ip>  # hostname 所有机器都得执行一遍,包括本机
      ```
  - 修改配置文件
    - hadoop-env.sh
      ```
      # vi etc/hadoop/hadoop-env.sh

      export JAVA_HOME=/hejun/software/jdk1.8.0_271     # 打开 export JAVA_HOME 并修改
      export HADOOP_HOME=/hejun/software/hadoop-3.2.0   # 打开 export HADOOP_HOME 并修改
      export HADOOP_LOG_DIR=/hejun/logs/hadoop          # 打开 export HADOOP_LOG_DIR 并修改
      export HDFS_NAMENODE_USER=root                    # 打开 HDFS_NAMENODE_USER 并修改 root
      export HDFS_DATANODE_USER=root                    # 新增
      export HDFS_SECONDARYNAMENODE_USER=root           # 新增
      ```
    - yarn-env.sh
      ```
      # vi etc/hadoop/yarn-env.sh

      export YARN_NODEMANAGER_USER=root  
      export YARN_RESOURCEMANAGER_USER=root
      ```
    - core-site.xml
      ```
      # vi etc/hadoop/core-site.xml                     # 在 configuration 中添加

      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://<master-hostname>:9000</value>
      </property>
      <property>
        <name>hadoop.tmp.dir</name>
        <value>/hejun/data/hadoop</value>
      </property>
      ```
    - hdfs-site.xml
      - 单机环境
        ```
        # vi etc/hadoop/hdfs-site.xml                   # 在 configuration 中添加

        <property>
          <name>dfs.replication</name>
          <value>1</value>                              # 根据实际情况定,默认是3,应小于datanode数量
        </property>
        ```
      - 分布式环境
        ```
        # vi etc/hadoop/hdfs-site.xml                   # 在 configuration 中添加
        
        <property>
          <name>dfs.namenode.name.dir</name>
          <value>/hejun/data/hdfs/name</value>
        </property>
        <property>
          <name>dfs.datanode.data.dir</name>
          <value>/hejun/data/hdfs/data</value>
        </property>
        ```
    - yarn-site.xml
      ```
      # vi etc/hadoop/yarn-site.xml                    # 在 configuration 中添加

      <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
      </property>
      ```
    - mapred-site.xml
      ```
      # vi etc/hadoop/mapred-site.xml

      <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
      ```
    - workers (分布式下才配置)
      ```
      # vi etc/hadoop/workers
      # 删除 localhost , 改为 worker 机器的 hostname
      ```
4. 格式化文件系统 (只在 namenode 做)
  ```
  ./bin/hdfs namenode -format
  ```
5. 设置开机自启  (只在 namenode 做, 其他节点会跟随 namenode 启动)
  - Dfs
  ```
  # vi /usr/lib/systemd/system/hdfs.service
  
  [Unit]
  Description=Hdfs
  After=network.target

  [Service]
  User=root
  Group=root
  Type=forking
  ExecStart=/hejun/software/hadoop-3.2.0/sbin/start-dfs.sh
  ExecStop=/hejun/software/hadoop-3.2.0/sbin/stop-dfs.sh
  PIDFile=/tmp/hadoop-root-namenode.pid

  [Install]
  WantedBy=multi-user.target
  ```
  ```
  systemctl daemon-reload
  systemctl enable hdfs
  ```
  - Yarn
  ```
  # vi /usr/lib/systemd/system/yarn.service
  
  [Unit]
  Description=Yarn
  After=network.target

  [Service]
  User=root
  Group=root
  Type=forking
  ExecStart=/hejun/software/hadoop-3.2.0/sbin/start-yarn.sh
  ExecStop=/hejun/software/hadoop-3.2.0/sbin/stop-yarn.sh
  PIDFile=/tmp/hadoop-root-resourcemanager.pid

  [Install]
  WantedBy=multi-user.target
  ```
  ```
  systemctl daemon-reload
  systemctl enable yarn
  ```
6. 启动&停止   (只在 namenode 做, 其他节点会跟随 namenode 启动)
  - Hdfs
  ```
  # 启动 HDFS
  systemctl start hdfs
  # 停止 HDFS
  systemctl stop hdfs
  ```
  - Yarn
  ```
  # 启动 Yarn
  systemctl start yarn
  # 停止 Yarn
  systemctl stop yarn
  ```
7. 开启端口
  - namenode 节点
  ```
  firewall-cmd --zone=public --add-port=8088/tcp --add-port=9000/tcp --add-port=9870/tcp --permanent
  firewall-cmd --reload
  ```
  - worker 节点
  ```
  firewall-cmd --zone=public --add-port=8031/tcp --permanent
  ```
8. NameNode 高可用
  - [HDFSHighAvailabilityWithQJM](https://hadoop.apache.org/docs/r3.2.0/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html)
  - [HDFSHighAvailabilityWithNFS](https://hadoop.apache.org/docs/r3.2.0/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithNFS.html)
