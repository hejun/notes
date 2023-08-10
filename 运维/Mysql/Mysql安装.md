## Mysql安装

### 卸载 `Mariadb`

```sh
rpm -e --nodeps $(rpm -qa | grep mariadb)
```

### 安装 `Mysql`

  1. 配置 `Mysql` yum 源
     
     ```sh
     wget https://dev.mysql.com/get/mysql80-community-release-el7-9.noarch.rpm
     ```

     1. 安装 rpm
        
        ```sh
        rpm -ivh mysql80-community-release-el7-7.noarch.rpm
        ```

     2. 修改默认下载版本（非必须）

        ```sh
        yum-config-manager --disable mysql80-community
        yum-config-manager --enable mysql57-community
        ```
        > 这一步可以不执行,不修改的话默认下载 `MySql 8`, 修改后为 `Mysql5.7`

  2. 安装 `MySql`

     ```sh
     yum -y install mysql-community-server
     ```

### 配置 `Mysql`

  1. 配置 `my.cnf`
  
     ```sh
     vi /etc/my.cnf
     ```
  
     按需配置

     ```sh
     character-set-server=utf8
     default-time_zone=+8:00
     lower_case_table_names=1
     group_concat_max_len=102400
     sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
     ```

  2. 初始化 `Mysql`

     ```sh
     mysqld --initialize --user=mysql 
     ```

     > `--user` 意为指定Mysql启动用户, 可不添加. 添加时则需先把 `my.cnf` 中的相关目录（如: `datadir` ）创建并授权给 `--user` 后缀指定的用户<br/>非 `root` 用户启动还需关闭 `seinux`
     > ```sh
     > sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
     > setenforce 0
     > ```

  3. 设置开机自启及启动

     ```sh
     systemctl enable mysqld
     ```

     ```sh
     systemctl start mysqld
     ```

  4. 修改密码及开启远程访问

     1. 查询默认密码

        ```sh
        grep "password" /var/log/mysqld.log
        ```

        > 这里的目录是 `my.cnf` 里默认的日志目录，如有修改，需有相应改动

     2. 使用初始密码登录mysql

        ```sh
        mysql -uroot -p
        ```

     3. 登陆后修改密码及验证规则
        1. 修改验证规则 (非必须)

           ```mysql
           set global validate_password_policy=0;
           ```

           > 修改密码验证策略为low（只验证长度）

           ```mysql
           set global validate_password_length=4;
           ```

           > 修改密码验证策略密码长度的最小值

        2. 修改密码

           ```mysql
           set password=PASSWORD('1234');
           ```

     4. 登陆后设置远程访问

        ```mysql
        use mysql;
        ```

        ```mysql
        update `user` set host='%' where `user`='root';
        ```

        ```mysql
        flush privileges;
        ```

  5. 开放防火墙（非必须）

     ```sh
     firewall-cmd --zone=public --add-port=3306/tcp --permanent
     ```

     ```sh
     firewall-cmd --reload
     ```
