# Spark (2.4.5)

## 前置条件
  - Java8
  - [Maven-3.5.4](https://archive.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz) (需配置国内镜像源)
## 安装
1. 下载 [Spark](https://archive.apache.org/dist/spark/spark-2.4.5/spark-2.4.5.tgz)
2. 解压
  ```
  tar -zxvf spark-2.4.5.tgz -C /hejun/software
  ```
3. 编译
  ```
  # vi pom.xml , repositories 节点下清空并添加下方节点, pluginRepositories 清空所有节点
  <repository>
     <id>cloudera</id>
    <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
  </repository>
  
  # 修改
  <version>3.2.0</version>
  ```
  ```
  ./dev/change-scala-version.sh 2.12
  export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=1g"
  ./dev/make-distribution.sh --name spark-2.4.5 --tgz -Pyarn -Phadoop-3.1 -Phive -Phive-thriftserver -Phadoop.version=3.1.1
  ```
