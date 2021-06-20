# Scala (2.13.6)

1. 下载 [Scala](https://downloads.lightbend.com/scala/2.13.6/scala-2.13.6.tgz)
2. 解压
  ```
  tar -zxvf scala-2.13.6.tgz -C /hejun/software
  ```
3. 配置环境变量
  ```
  # vi /etc/profile
  
  export SCALA_HOME=/hejun/software/scala-2.13.6
  export PATH=$PATH:$SCALA_HOME/bin
  ```
  ```
  source /etc/profile
  ```
