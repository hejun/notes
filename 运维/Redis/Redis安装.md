## Redis安装

### 配置优化

- Tcp Block
  
  ```sh
  echo net.core.somaxconn=1024 >> /etc/sysctl.conf
  ```
  
  ```sh
  echo vm.overcommit_memory=1 >> /etc/sysctl.conf
  ```
  
  ```sh
  sysctl -p
  ```

### 安装

- 下载

  最新稳定版:
  ```
  wget https://download.redis.io/redis-stable.tar.gz
  ```

  其他版本

  [Redis下载官网](https://download.redis.io/releases/)

- 解压

  ```sh
  tar -xzvf redis-stable.tar.gz
  ```

- 编译

  ```sh
  cd redis-stable
  ```

  ```sh
  yum install centos-release-scl -y
  ```
  
  ```sh
  yum install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils -y
  ```
  
  ```sh
  scl enable devtoolset-9 bash
  ```
  > 临时有效，退出 shell 或重启会恢复原 gcc 版本

  ```sh
  make MALLOC=libc PREFIX=${REDIS_HOME} install
  ```
  > ${REDIS_HOME} 为 `redis` 想要安装的目录
  
  例:

  ```sh
  make MALLOC=libc PREFIX=/opt/redis install
  ```
  
  ```sh
  cp redis.conf ${REDIS_HOME}/conf
  ```
  > 粘贴配置文件到安装目录,`conf` 目录不存在，需要先创建，该目录不是必须

  例:

  ```sh
  mkdir /opt/redis/conf
  cp redis.conf /opt/redis/conf
  ```

- 开机自启

  ```sh
  useradd -s /bash/false -M redis
  ```
  > 添加 `redis` 用户以方便用非 `root` 身份启动,一旦非 `root` 启动,需要把 `redis` 相关目录设置权限

  ```sh
  cp utils/redis_init_script /etc/init.d/redis
  ```
  
  ```sh
  vi /etc/init.d/redis
  ```
  > 修改 `redis` 自启文件, 需要更改 `EXEC` `CLIEXEC` `CONF` 的目录, 修改 启动 `$EXEC $CONF` 命令为 `sudo -u redis $EXEC $CONF` 

  ```sh
  chkconfig --add /etc/init.d/redis
  ```

- 开放防火墙（非必须）

  ```sh
  firewall-cmd --zone=public --add-port=6379/tcp --permanent
  ```

  ```sh
  firewall-cmd --reload
  ```
