# Kafka (2.12-2.6.0)

1. 下载 [Kafka](https://archive.apache.org/dist/kafka/2.6.0/kafka_2.12-2.6.0.tgz)
2. 解压
  ```
  tar -zxvf kafka_2.12-2.6.0.tgz -C /hejun/software
  ```
3. 修改配置文件
  ```
  # cd /hejun/software/kafka_2.12-2.6.0
  # vi config/server.properties
  
  broker.id=<broker-id>
  listeners=PLAINTEXT://0.0.0.0:9092
  advertised.listeners=PLAINTEXT://<ip>:9092  # 不要用 hostname
  log.dirs=/hejun/data/kafka
  zookeeper.connect=<zookeeper-ip>:2181       # 分布式环境下,用 逗号 分隔
  ```
4. 修改logs路径(非必须)
  ```
  # vi bin/kafka-run-class.sh
  # 在顶部下面添加 LOG_DIR=/hejun/logs/kafka
  ```
5. 设置开机自启
  ```
  # vi /usr/lib/systemd/system/kafka.service
  
  [Unit]
  Description=Kafka
  After=zookeeper.target

  [Service]
  User=root
  Group=root
  Type=forking
  Environment="JAVA_HOME=/hejun/software/jdk1.8.0_271"
  ExecStart=/hejun/software/kafka_2.12-2.6.0/bin/kafka-server-start.sh -daemon /hejun/software/kafka_2.12-2.6.0/config/server.properties
  ExecStop=/hejun/software/kafka_2.12-2.6.0/bin/kafka-server-stop.sh

  [Install]
  WantedBy=multi-user.target
  ```
  ```
  systemctl daemon-reload
  systemctl enable kafka
  ```
6. 启动&停止
  ```
  # 启动
  systemctl start kafka
  # 停止
  systemctl stop kafka
  ```
 7. 开放端口
  ```
  firewall-cmd --zone=public --add-port=9092/tcp --permanent
  firewall-cmd --reload
  ```
