# Zookeeper (3.5.7)

1. 下载 [Zookeeper](https://archive.apache.org/dist/zookeeper/zookeeper-3.5.7/apache-zookeeper-3.5.7-bin.tar.gz)
2. 解压
  ```
  tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz -C /hejun/software
  mv apache-zookeeper-3.5.7-bin zookeeper-3.5.7
  ```
3. 修改配置文件
  - 单机配置
    ```
    # cd /hejun/software/zookeeper-3.5.7
    # cp conf/zoo_sample.cfg conf/zoo.cfg
    # vi conf/zoo.cfg

    dataDir=/hejun/data/zookeeper               # 修改
    dataLogDir=/hejun/logs/zookeeper            # 新增
    admin.enableServer=false                    # 新增
    ```
  - 分布式配置
    ```
    # 在 zoo.cfg 追加配置
    server.<master-id>=<master-ip>:2888:3888
    server.<slave-id>=<slave-ip>:2888:3888
    ```
    ```
    # 在 master 服务器
    echo <master-id> > <上面配置的dataDir>/myid
    ```
    ```
    # 在 slave 服务器
    echo <slave-id> > <上面配置的dataDir>/myid
    ```
4. 修改logs路径(非必须)
  ```
  # vi bin/zkServer.sh
  # 在 ZOO_DATALOGDIR 这一行下面添加 ZOO_LOG_DIR="$($GREP "^[[:space:]]*dataLogDir" "$ZOOCFG" | sed -e 's/.*=//')"
  ```
5. 设置开机自启
  ```
  # vi /usr/lib/systemd/system/zookeeper.service
  
  [Unit]
  Description=Zookeeper
  After=network.target

  [Service]
  User=root
  Group=root
  Type=forking
  Environment="JAVA_HOME=/hejun/software/jdk1.8.0_271"
  ExecStart=/hejun/software/zookeeper-3.5.7/bin/zkServer.sh start /hejun/software/zookeeper-3.5.7/conf/zoo.cfg
  ExecReload=/hejun/software/zookeeper-3.5.7/bin/zkServer.sh restart
  ExecStop=/hejun/software/zookeeper-3.5.7/bin/zkServer.sh stop

  [Install]
  WantedBy=multi-user.target
  ```
  ```
  systemctl daemon-reload
  systemctl enable zookeeper
  ```
6. 启动&停止
  ```
  # 启动
  systemctl start zookeeper
  # 停止
  systemctl stop zookeeper
  ```
 7. 开放端口
  - 单机配置
    ```
    firewall-cmd --zone=public --add-port=2181/tcp --permanent
    firewall-cmd --reload
    ```
  - 分布式配置
    ```
    firewall-cmd --zone=public --add-port=2888/tcp --add-port=3888/tcp --permanent
    firewall-cmd --reload
    ```
