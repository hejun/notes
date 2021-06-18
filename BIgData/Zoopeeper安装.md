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
5. 启动 Zookeeper
  ```
  ./bin/zkServer.sh start ./conf/zoo.cfg
  ```
6. 停止 Zookeeper
  ```
  ./bin/zkServer.sh stop
  ```
