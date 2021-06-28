# Spark (3.1.2)

## 前置条件
  - Java8
  - [Maven-3.6.3](https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz) (需配置国内镜像源)
## 安装
1. 下载 [Spark](https://archive.apache.org/dist/spark/spark-3.1.2/spark-3.1.2.tgz)
2. 解压
  ```
  tar -zxvf spark-3.1.2.tgz -C /hejun/software
  ```
3. 编译
  ```
  cd /hejun/software/spark-3.1.2
  ```
  ```
  # vi pom.xml , repositories 节点下清空并添加下方节点, pluginRepositories 清空所有节点
  <repository>
     <id>cloudera</id>
    <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
  </repository>
  ```
  ```  
  export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=1g"
  
  ./dev/make-distribution.sh --tgz -Phive -Phive-thriftserver -Pyarn
  ```
  ```
  cp spark-3.1.2-bin-3.2.0.tgz ../
  ```
 4. 安装
  - 解压
    ```
    tar -zxvf spark-3.1.2-bin-3.2.0.tgz -C /hejun/software/

    mv spark-3.1.2-bin-3.2.0 spark-3.1.2
    ```
  - 修改配置文件
    ```
    cd /hejun/software/spark-3.1.2
    ```
    ```
    cp conf/spark-env.sh.template conf/spark-env.sh
    cp conf/workers.template conf/workers
    ```
    ```
    # vi conf/spark-env.sh
    
    export JAVA_HOME=/hejun/software/jdk1.8.0_271
    export SCALA_HOME=/hejun/software/scala-2.12.10
    export HADOOP_HOME=/hejun/software/hadoop-3.2.0
    export SPARK_LOG_DIR=/hejun/logs/spark
    ```
    ```
    # vi conf/workers
    # 删除 localhost , 改为 worker 机器的 hostname
    ```
  - 设置开机自启 (只在 Master 做)
    ```
    # vi /usr/lib/systemd/system/spark.service
    
    [Unit]
    Description=Spark
    After=hdfs.target,yarn.target

    [Service]
    User=root
    Group=root
    Type=forking
    Environment="SPARK_HOME=/hejun/software/spark-3.1.2"
    ExecStart=/hejun/software/spark-3.1.2/sbin/start-all.sh
    ExecStop=/hejun/software/spark-3.1.2/sbin/stop-all.sh
    PIDFile=/tmp/spark-root-org.apache.spark.deploy.master.Master-1.pid

    [Install]
    WantedBy=multi-user.target
    ```
    ```
    systemctl daemon-reload
    systemctl enable spark
    ```
  - 开放端口
    ```
    firewall-cmd --zone=public --add-port=7077/tcp --add-port=8080/tcp --permanent
    firewall-cmd --reload
    ```
