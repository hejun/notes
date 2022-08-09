## Kafka安装

### 下载

[Kafka下载官网](https://archive.apache.org/dist/kafka/)
  
### 安装配置

- 解压

  ```sh
  tar -zxvf kafka_${kafka.version}.tgz
  ```
  > ${kafka.version} 为 `kafka` 版本

- 修改配置文件

  ```sh
  vi config/server.properties
  ```
  
  ```sh
  broker.id=${broker-id}
  listeners=PLAINTEXT://0.0.0.0:9092
  advertised.listeners=PLAINTEXT://${server-ip}:9092
  log.dirs=/hejun/data/kafka
  zookeeper.connect=${zookeeper-host}:2181
  ```
  > `broker-id` 为指定的节点id, 不可重复.<br/>`server-ip`为服务器ip.<br/>`log.dirs` 在 `kafka` 中不是日志地址，是每个 `Topic` 接收消息的存储地址。<br/>`zookeeper.connect` 指定了 `Zookeeper` 的地址,多个zk地址用逗号分割

- 修改logs路径

  ```sh
  vi bin/kafka-run-class.sh
  ```
  
  ```sh
  LOG_DIR=/hejun/logs/kafka
  ```
  > 在顶部下面新加

### 启动

- 启动 & 停止

  ```sh
  bin/kafka-server-start.sh -daemon config/server.properties
  ```
  > 启动
  
  ```sh
  bin/kafka-server-stop.sh
  ```
  > 停止
  
- 开机自启

  ```sh
  vi /usr/lib/systemd/system/kafka.service
  ```
  
  ```sh
  [Unit]
  Description=Kafka
  After=network.target zookeeper.target
  
  [Service]
  User=root
  Group=root
  Type=forking
  Environment="JAVA_HOME=${JAVA_HOME}"
  ExecStart=${Kafka_Home}/bin/kafka-server-start.sh -daemon ${Kafka_Home}/config/server.properties
  ExecStop=${Kafka_Home}/bin/kafka-server-stop.sh
  
  [Install]
  WantedBy=multi-user.target
  ```

  ```sh
  systemctl daemon-reload
  ```
  
  ```sh
  systemctl enable kafka
  ```

  ```sh
  systemctl start kafka
  ```
  > 启动
  
  ```sh
  systemctl stop kafka
  ```
  > 停止

- 开放端口

  ```sh
  firewall-cmd --zone=public --add-port=9092/tcp --permanent
  ```

  ```sh
  firewall-cmd --reload
  ```
