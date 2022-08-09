## Solr安装

### 下载

[Solr下载官网](http://archive.apache.org/dist/lucene/solr/)

### 安装配置

- 解压

  ```sh
  tar -zxvf solr-${solr.version}.tar.gz
  ```
  > ${solr.version} 为 `solr` 版本

- 修改配置文件

  ```sh
  vi ${ZOOKEEPER_HOME}/conf/zoo.cfg
  ```
  
  ```sh
  4lw.commands.whitelist=mntr,conf,ruok
  ```
  > Solr 本身不需要配置, 但是 Cloud 模式需要在 Zookeeper 配置下增添该配置
  
### 启动

- 启动 & 停止

  ```sh
  ./bin/solr start -cloud -z ${ZOOKEEPER_HOST} -t ${SOLR_DATA_DIR} -j "-Dsolr.log.dir=${SOLR_LOG_DIR}" -force
  ```
  > `-cloud` 集群模式<br/>`-z` Zookeeper地址,多个用逗号分隔<br/>`-t` Solr Data 目录, 可不指定. 若指定, 必须提前创建好<br/>`-j` jetty启动参数, 可不指定<br/>`-force` 强制启动
  
  ```sh
  ./bin/solr stop
  ```

- 开机自启

  ```sh
  vi /usr/lib/systemd/system/solr.service
  ```

  ```sh
  [Unit]
  Description=Solr
  After=network.target zookeeper.target
  
  [Service]
  User=root
  Group=root
  Type=forking
  Environment="JAVA_HOME=${JAVA_HOME}"
  ExecStart=${SOLR_HOME}/bin/solr start -cloud -z ${ZOOKEEPER_HOST} -t ${SOLR_DATA_DIR} -j "-Dsolr.log.dir=${SOLR_LOG_DIR}" -force
  ExecStop=${SOLR_HOME}/bin/solr stop
  
  [Install]
  WantedBy=multi-user.target
  ```
  > `Zookeeper` 多个用逗号分隔

  ```sh
  systemctl daemon-reload
  ```
  
  ```sh
  systemctl enable solr
  ```

  ```sh
  systemctl start solr
  ```

- 开启端口

  ```sh
  firewall-cmd --zone=public --add-port=8983/tcp --permanent
  ```
  
  ```sh
  firewall-cmd --reload
  ```
