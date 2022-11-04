## Zookeeper安装

### 下载

[Zookeeper下载官网](https://archive.apache.org/dist/zookeeper/)
  
### 安装配置

- 解压

  ```sh
  tar -zxvf apache-zookeeper-${zookeeper.version}-bin.tar.gz
  ```
  > ${zookeeper.version} 为 `zookeeper` 版本

- 修改配置文件

  ```sh
  cp conf/zoo_sample.cfg conf/zoo.cfg
  ```
  
  ```sh
  vi conf/zoo.cfg
  ```
  
  ```sh
  dataDir=/hejun/data/zookeeper
  dataLogDir=/hejun/logs/zookeeper
  admin.enableServer=false
  ```
  > `dataDir` 是 `Zookeeper` 存放数据的目录.<br/> `dataLogDir` 默认不存在, 需新增, 是 `Zookeeper` 存放日志的地方.<br/> `admin.enableServer` 是 `3.5.0` 版本中新增特性, `ture` 为开启, `false` 为关闭.

  - 在分布式环境下还需追加以下配置

    ```sh
    server.${id}=${hostname}:2888:3888
    ```
    
    例如:
    
    ```sh
    server.1=node-01:2888:3888
    server.2=node-02:2888:3888
    server.3=node-03:2888:3888
    ```
    
    指定本机 `Zookeeper` 的 `id`
    
    ```sh
    echo ${id} > ${dataDir}/myid
    ```
    > `id` 是机器需要指定的id, `dataDir` 是在配置文件 `conf/zoo.cfg` 中指定的目录. 每台机器都得执行该命令, 每台机器的 `id` 不可重复
    
    例如：
    
    ```sh
    echo 1 > /hejun/data/zookeeper/myid
    ```

  - 修改logs路径
    
    ```sh
    vi bin/zkServer.sh
    ```
    
    ```sh
    ZOO_LOG_DIR="$($GREP "^[[:space:]]*dataLogDir" "$ZOOCFG" | sed -e 's/.*=//')"
    ```
    > 在 `ZOO_DATALOGDIR` 这一行下面添加, 配置文件中添加 `dataLogDir` 还不够, 需要这一句

### 启动

- 启动 & 停止

  ```sh
  bin/zkServer.sh start conf/zoo.cfg
  ```
  > 启动
  
  ```sh
  bin/zkServer.sh stop
  ```
  > 停止
  
- 开机自启

  ```sh
  vi /usr/lib/systemd/system/zookeeper.service
  ```
  
  ```sh
  [Unit]
  Description=Zookeeper
  After=network.target
  
  [Service]
  User=root
  Group=root
  Type=forking
  Environment="JAVA_HOME=${JAVA_HOME}"
  ExecStart=${ZOOKEEPER_HOME}/bin/zkServer.sh start ${ZOOKEEPER_HOME}/conf/zoo.cfg
  ExecReload=${ZOOKEEPER_HOME}/bin/zkServer.sh restart
  ExecStop=${ZOOKEEPER_HOME}/bin/zkServer.sh stop
  
  [Install]
  WantedBy=multi-user.target
  ```
  > 需要修改 `${JAVA_HOME}`, `${ZOOKEEPER_HOME}` 为真实目录
  
  ```sh
  systemctl daemon-reload
  ```
  
  ```sh
  systemctl enable zookeeper
  ```
  
  ```sh
  systemctl start zookeeper
  ```
  > 启动
  
  ```sh
  systemctl stop zookeeper
  ```
  > 停止

- 开放端口

  - 单机
  
    ```sh
    firewall-cmd --zone=public --add-port=2181/tcp --permanent
    firewall-cmd --reload
    ```
  
  - 分布式
  
    ```sh
    firewall-cmd --zone=public --add-port=2181/tcp --add-port=2888/tcp --add-port=3888/tcp --permanent
    firewall-cmd --reload
    ```
