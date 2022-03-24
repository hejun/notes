## Hadoop安装

### 下载

[Hadoop下载官网](https://archive.apache.org/dist/hadoop/common/)

### 前置条件

- [免密登录](https://github.com/hejun/notes/blob/master/%E8%BF%90%E7%BB%B4/Linux/SSH%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95.md)
  
### 安装配置

- 解压

  ```
  tar -zxvf hadoop-${hadoop.version}.tar.gz
  ```
  > ${hadoop.version} 为 `hadoop` 版本

- 修改配置文件

  - `hadoop-env.sh`
  
    ```
    vi etc/hadoop/hadoop-env.sh
    ```

    ```
    export JAVA_HOME=${JAVA_HOME}
    export HADOOP_HOME=${HADOOP_HOME}
    export HADOOP_LOG_DIR=${HADOOP_LOG_DIR}
    export HDFS_NAMENODE_USER=root
    export HDFS_DATANODE_USER=root
    export HDFS_SECONDARYNAMENODE_USER=root
    ```
    > 打开注释并替换 `JAVA_HOME`, `HADOOP_HOME` 为真实地址.<br/>`HADOOP_LOG_DIR` 默认为 `${HADOOP_HOME}/logs`, 可换可不换.<br/>`HDFS_DATANODE_USER` 和 `HDFS_SECONDARYNAMENODE_USER` 不存在, 需新增

  - `yarn-env.sh`
  
    ```
    vi etc/hadoop/yarn-env.sh
    ```
  
    ```
    export YARN_NODEMANAGER_USER=root
    export YARN_RESOURCEMANAGER_USER=root
    ```
    > 该配置不存在, 需新增

  - `core-site.xml`
  
    ```
    vi etc/hadoop/core-site.xml
    ```
    
    ```
    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://${NameNode_URI}:9000</value>
    </property>
    <property>
      <name>hadoop.tmp.dir</name>
      <value>${Tmp_Dir}</value>
    </property>
    ```
    > 在 `configuration` 节点中加入上述配置

    例：
    ```
    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://node-01:9000</value>
    </property>
    <property>
      <name>hadoop.tmp.dir</name>
      <value>/hejun/data/hadoop/tmp</value>
    </property>
    ```
  
  - `hdfs-site.xml`
  
    ```
    vi etc/hadoop/hdfs-site.xml
    ```
    
    ```
    <property>
      <name>dfs.namenode.name.dir</name>
      <value>${NameNode_Dir}</value>
    </property>
    <property>
      <name>dfs.datanode.data.dir</name>
      <value>${DataNode_Dir}</value>
    </property>
    ```
    
    例:
    
    ```
    <property>
      <name>dfs.namenode.name.dir</name>
      <value>/hejun/data/hadoop/hdfs/name</value>
    </property>
    <property>
      <name>dfs.datanode.data.dir</name>
      <value>/hejun/data/hadoop/hdfs/data</value>
    </property>
    ```
  
  - `yarn-site.xml`
  
    ```
    vi etc/hadoop/yarn-site.xml
    ```
    
    ```
    <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>${Master_Hostname}</value>
    </property>
    ```
    
    例:
    
    ```
    <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>node-01</value>
    </property>
    ```

  - `workers`
  
    ```
    vi etc/hadoop/workers
    ```
    > 打开后删除 `localhost`, 添加 DataNode 的 hostname
    
    例:
    
    ```
    node-02
    node-03
    ```

  - 格式化文件系统
  
    ```
    ./bin/hdfs namenode -format
    ```
    > 该命令只在 NameNode 执行

### 启动

- 启动 & 停止

  > 启停只在NameNode做, DataNode会通过 `etc/hadoop/workers` 配置的 hostname 进行启动

  ```
  sbin/start-dfs.sh
  ```
  > 启动
  
  ```
  sbin/stop-dfs.sh
  ```
  > 停止
  
- 开机自启

  > 开机自启只在NameNode做, DataNode会通过 `etc/hadoop/workers` 配置的 hostname 进行启动

  - dfs
  
    ```
    vi /usr/lib/systemd/system/dfs.service
    ```
    
    ```
    [Unit]
    Description=dfs
    After=network.target
    
    [Service]
    User=root
    Group=root
    Type=forking
    ExecStart=${HADOOP_HOME}/sbin/start-dfs.sh
    ExecStop=${HADOOP_HOME}/sbin/stop-dfs.sh
    PIDFile=/tmp/hadoop-root-namenode.pid
    
    [Install]
    WantedBy=multi-user.target
    ```

  - yarn
  
    ```
    vi /usr/lib/systemd/system/yarn.service
    ```
    
    ```
    [Unit]
    Description=Yarn
    After=network.target
    
    [Service]
    User=root
    Group=root
    Type=forking
    ExecStart=${HADOOP_HOME}/sbin/start-yarn.sh
    ExecStop=${HADOOP_HOME}/sbin/stop-yarn.sh
    PIDFile=/tmp/hadoop-root-resourcemanager.pid
    
    [Install]
    WantedBy=multi-user.target
    ```
    
  - 开启自启
  
    ```
    systemctl daemon-reload
    ```
    
    ```
    systemctl enable dfs
    ```
    
    ```
    systemctl enable yarn
    ```

- 开启端口

  - NameNode
  
    ```
    firewall-cmd --zone=public --add-port=9000/tcp --add-port=9820/tcp --add-port=9870-9871/tcp --permanent
    ```
  
  - SecondNameNode
  
    ```
    firewall-cmd --zone=public --add-port=9868/tcp --add-port=9869/tcp --permanent
    ```
    
  - Yarn
  
    ```
    firewall-cmd --zone=public --add-port=8088/tcp --add-port=8030-8033/tcp --permanent
    ```
    
  - DataNode
  
    ```
    firewall-cmd --zone=public --add-port=9864-9867/tcp --permanent
    ```

  - 重载
  
    ```
    firewall-cmd --reload
    ```
    
### NameNode 高可用

  - [HDFSHighAvailabilityWithQJM](https://hadoop.apache.org/docs/r3.3.1/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html)
  - [HDFSHighAvailabilityWithNFS](https://hadoop.apache.org/docs/r3.3.1/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithNFS.html)
