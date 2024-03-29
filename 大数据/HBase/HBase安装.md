## HBase安装

### 下载

[HBase下载官网](https://archive.apache.org/dist/hbase/)
> 版本选择：https://hbase.apache.org/book.html#hadoop


### 前置条件

- [免密登录](https://github.com/hejun/notes/blob/master/%E8%BF%90%E7%BB%B4/Linux/SSH%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95.md)

- NTP 时间校准

### 安装配置

- 解压

  ```sh
  tar -zxvf hbase-${hbase.version}-bin.tar.gz
  ```
  > ${hbase.version} 为 `hbase` 版本

- 修改配置文件

  - `hbase-env.sh`
  
    ```sh
    vi conf/hbase-env.sh
    ```
    
    ```sh
    export JAVA_HOME=${JAVA_HOME}
    export HBASE_LOG_DIR=${HBASE_LOG_DIR}
    export HBASE_MANAGES_ZK=false
    ```
    > 打开注释并替换 `JAVA_HOME`, `HBASE_LOG_DIR` 为真实地址.<br/>`HBASE_MANAGES_ZK` 为是否启用HBase自带 `Zookeeper`

  - `hbase-site.xml`
  
    ```sh
    vi conf/hbase-site.xml
    ```
    
    ```xml
    <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
    </property>
    <property>
      <name>hbase.tmp.dir</name>
      <value>${HBASE_TMP_DIR}</value>
    </property>
    <property>
      <name>hbase.unsafe.stream.capability.enforce</name>
      <value>false</value>
    </property>
    <property>
      <name>hbase.zookeeper.quorum</name>
      <value>${ZOOKEEPER_HOST}</value>
    </property>
    <property>
      <name>hbase.rootdir</name>
      <value>${HADOOP_FS.DEFAULTFS}/hbase</value>
    </property>
    <property>
      <name>hbase.zookeeper.property.dataDir</name>
      <value>${ZOOKEEPER_DATA_DIR}</value>
    </property>
    ```
    > `HADOOP_FS.DEFAULTFS` 是 hadoop 的 `core-site.xml` 里的 `fs.defaultFS` 配置<br/>`ZOOKEEPER_DATA_DIR` 是 Zookeeper 的 data 目录
    
    例:
    
    ```xml
    <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
    </property>
    <property>
      <name>hbase.tmp.dir</name>
      <value>/hejun/data/hbase/tmp</value>
    </property>
    <property>
      <name>hbase.unsafe.stream.capability.enforce</name>
      <value>false</value>
    </property>
    <property>
      <name>hbase.zookeeper.quorum</name>
      <value>node-01:2181,node-02:2181,node-03:2181</value>
    </property>
    <property>
      <name>hbase.rootdir</name>
      <value>hdfs://node-01:9000/hbase</value>
    </property>
    <property>
      <name>hbase.zookeeper.property.dataDir</name>
      <value>/hejun/data/zookeeper</value>
    </property>
    ```
  
  - `regionservers`
  
    ```sh
    vi conf/regionservers
    ```
    > 打开后删除 `localhost`, 添加 RegionServers 的 hostname

    例:
    
    ```sh
    node-02
    node-03
    ```

### 启动

- 启动 & 停止

  > 启停只在HMaster做, HRegionServer 会通过 `conf/regionservers` 配置的 hostname 进行启动

  ```sh
  bin/start-hbase.sh
  ```
  > 启动
  
  ```sh
  bin/stop-hbase.sh
  ```
  > 停止
  
- 开机自启

  > 开机自启只在HMaster做, HRegionServer 会通过 `conf/regionservers` 配置的 hostname 进行启动

  ```sh
  vi /usr/lib/systemd/system/hbase.service
  ```
  
  ```sh
  [Unit]
  Description=Hbase
  After=network.target zookeeper.target dfs.target
  
  [Service]
  User=root
  Group=root
  Type=forking
  ExecStart=${HBASE_HOME}/bin/start-hbase.sh
  ExecStop=${HBASE_HOME}/bin/stop-hbase.sh
  PIDFile=/tmp/hbase-root-master.pid
  
  [Install]
  WantedBy=multi-user.target
  ```
  
  ```sh
  systemctl daemon-reload
  ```
  
  ```sh
  systemctl enable hbase
  ```
  
- 开启端口

  - HMaster
  
    ```sh
    firewall-cmd --zone=public --add-port=16000/tcp --add-port=16010/tcp --permanent
    ```
  
  - HRegionServer
  
    ```sh
    firewall-cmd --zone=public --add-port=16020/tcp --add-port=16030/tcp --permanent
    ```
    
  - 重载
    
    ```sh
    firewall-cmd --reload
    ```
