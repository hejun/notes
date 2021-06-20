# Scala (2.12.10)

1. 下载 [Scala](https://downloads.lightbend.com/scala/2.12.10/scala-2.12.10.tgz)
2. 解压
  ```
  tar -zxvf scala-2.12.10.tgz -C /hejun/software
  ```
3. 配置环境变量
  ```
  # vi /etc/profile
  
  export SCALA_HOME=/hejun/software/scala-2.12.10
  export PATH=$PATH:$SCALA_HOME/bin
  ```
  ```
  source /etc/profile
  ```
