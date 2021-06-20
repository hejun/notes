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
  # vi pom.xml , repositories 节点下清空并添加下方节点, pluginRepositories 清空所有节点
  <repository>
     <id>cloudera</id>
    <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
  </repository>
  ```
  ```  
  export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=1g"
  
  ./dev/make-distribution.sh --name spark-3.1.1-bin --tgz -Pyarn -Phadoop-3.1 -Phive -Phive-thriftserver -Phadoop.version=3.1.1 -Pscala.version=2.13.6
  ```
