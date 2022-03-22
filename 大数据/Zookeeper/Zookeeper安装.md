## Zookeeper安装

### 下载

[Zookeeper下载官网](https://archive.apache.org/dist/zookeeper/)
  
### 安装配置

- 解压

  ```
  tar -zxvf apache-zookeeper-${zookeeper.version}-bin.tar.gz
  ```
  > ${zookeeper.version} 为 `zookeeper` 版本

- 修改配置文件

  ```
  cp conf/zoo_sample.cfg conf/zoo.cfg
  ```
  
  ```
  vi conf/zoo.cfg
  ```
  
  ```
  dataDir=/hejun/data/zookeeper
  dataLogDir=/hejun/logs/zookeeper
  admin.enableServer=false
  ```
  > `dataDir` 是 `Zookeeper` 存放数据的目录.<br/> `dataLogDir` 默认不存在, 需新增, 是 `Zookeeper` 存放日志的地方.<br/> `admin.enableServer` 是 `3.5.0` 版本中新增特性, `ture` 为开启, `false` 为关闭.

  - 在分布式环境下还需追加以下配置

    ```
    server.${id}=${hostname}:2888:3888
    ```
    
    例如:
    
    ```
    server.1=node-01:2888:3888
    server.2=node-02:2888:3888
    server.3=node-03:2888:3888
    ```
    
    指定本机 `Zookeeper` 的 `id`
    
    ```
    echo ${id} > ${dataDir}/myid
    ```
    > `id` 是机器需要指定的id, `dataDir` 是在配置文件 `conf/zoo.cfg` 中指定的目录. 每台机器都得执行该命令, 每台机器的 `id` 不可重复
    
    例如：
    
    ```
    echo 1 > /hejun/data/zookeeper/myid
    ```

  - 修改logs路径
    
    ```
    vi bin/zkServer.sh
    ```
    
    ```
    ZOO_LOG_DIR="$($GREP "^[[:space:]]*dataLogDir" "$ZOOCFG" | sed -e 's/.*=//')"
    ```
    > 在 `ZOO_DATALOGDIR` 这一行下面添加, 配置文件中添加 `dataLogDir` 还不够, 需要这一句

### 启动

- 启动 & 停止

  ```
  bin/zkServer.sh start conf/zoo.cfg
  ```
  > 启动
  
  ```
  bin/zkServer.sh stop
  ```
  > 停止
  
- 开启自启

  ```
  vi /usr/lib/systemd/system/zookeeper.service
  ```
  
  ```
  [Unit]
  Description=Zookeeper
  After=network.target
  
  [Service]
  User=root
  Group=root
  Type=forking
  Environment="JAVA_HOME=${JAVA_HOME}"
  ExecStart=${Zookeeper_Home}/bin/zkServer.sh start ${Zookeeper_Home}/conf/zoo.cfg
  ExecReload=${Zookeeper_Home}/bin/zkServer.sh restart
  ExecStop=${Zookeeper_Home}/bin/zkServer.sh stop
  
  [Install]
  WantedBy=multi-user.target
  ```
  > 需要修改 ${JAVA_HOME}, ${Zookeeper_Home} 为真实目录
  
  ```
  systemctl daemon-reload
  ```
  
  ```
  systemctl enable zookeeper
  ```
  
  ```
  systemctl start zookeeper
  ```
  > 启动
  
  ```
  systemctl stop zookeeper
  ```
  > 停止

- 开放端口

  - 单机
  
  ```
  firewall-cmd --zone=public --add-port=2181/tcp --permanent
  ```
  
  ```
  firewall-cmd --reload
  ```
  
  - 分布式
  
  ```
  firewall-cmd --zone=public --add-port=2181/tcp --add-port=2888/tcp --add-port=3888/tcp --permanent
  ```
  
  ```
  firewall-cmd --reload
  ```
