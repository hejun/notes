## Redis安装

### 下载

[Redis下载官网](https://download.redis.io/releases/)

### 配置优化

- Tcp Block
  
  ```
  echo net.core.somaxconn=1024 >> /etc/sysctl.conf
  ```
  
  ```
  echo vm.overcommit_memory=1 >> /etc/sysctl.conf
  ```
  
  ```
  sysctl -p
  ```

### 安装配置

- 解压

  ```
  tar -zxvf redis-${redis.version}.tar.gz
  ```
  > ${redis.version} 为 `redis` 版本

- 编译

  ```
  cd redis-${redis.version}
  ```

  ```
  yum install centos-release-scl -y
  ```
  
  ```
  yum install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils -y
  ```
  
  ```
  scl enable devtoolset-9 bash
  ```
  > 临时有效，退出 shell 或重启会恢复原 gcc 版本

  ```
  make MALLOC=libc PREFIX=${REDIS_HOME} install
  ```
  > ${REDIS_HOME} 为 `redis` 想要安装的目录
  
  例:

  ```
  make MALLOC=libc PREFIX=/hejun/software/redis-6 install
  ```
  
  ```
  cp redis.conf ${REDIS_HOME}/conf
  ```
  > 粘贴配置文件到安装目录,`conf` 目录不存在，需要先创建，该目录不是必须

  例:

  ```
  mkdir /hejun/software/redis-6/conf
  cp redis.conf ../redis-6/conf
  ```

- 开机自启

  ```
  useradd -s /bash/false -M redis
  ```
  > 添加 `redis` 用户以方便用非 `root` 身份启动,一旦非 `root` 启动,需要把 `redis` 相关目录设置权限

  ```
  cp utils/redis_init_script /etc/init.d/redis
  ```
  
  ```
  vi /etc/init.d/redis
  ```
  > 修改 `redis` 自启文件, 需要更改 `EXEC` `CLIEXEC` `CONF` 的目录, 修改 启动 `$EXEC $CONF` 命令为 `sudo -u redis $EXEC $CONF` 

  ```
  chkconfig --add /etc/init.d/redis
  ```

- 开放防火墙（非必须）

  ```
  firewall-cmd --zone=public --add-port=6379/tcp --permanent
  ```

  ```
  firewall-cmd --reload
  ```