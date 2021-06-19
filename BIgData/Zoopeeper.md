# Zookeeper (3.5.5)

1. 下载 [Zookeeper](https://archive.apache.org/dist/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz)
2. 解压
  ```
  tar -zxvf apache-zookeeper-3.5.5-bin.tar.gz -C /hejun/software
  mv apache-zookeeper-3.5.5-bin apache-zookeeper-3.5.5
  ```
3. 修改配置文件
  ```
  # cd /hejun/software/apache-zookeeper-3.5.5
  # cp conf/zoo_sample.cfg conf/zoo.cfg
  # vi conf/zoo.cfg
  
  # 修改: dataDir=/hejun/data/zookeeper
  # 添加: dataLogDir=/hejun/logs/zookeeper
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
  ExecStart=/hejun/software/apache-zookeeper-3.5.5/bin/zkServer.sh start /hejun/software/apache-zookeeper-3.5.5/conf/zoo.cfg
  ExecReload=/hejun/software/apache-zookeeper-3.5.5/bin/zkServer.sh restart
  ExecStop=/hejun/software/apache-zookeeper-3.5.5/bin/zkServer.sh stop

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
  ```
  firewall-cmd --zone=public --add-port=2181/tcp --permanent
  firewall-cmd --reload
  ```
