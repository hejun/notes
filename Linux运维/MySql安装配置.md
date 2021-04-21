#MySql安装配置

## 卸载 `Mariadb`

```
rpm -qa | grep mariadb

rpm -e --nodeps 上面查出来的Mariabd名称
```

## 下载mysql yum源

```
wget -P /etc/yum.repos.d/mysql-community.repo https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

## 修改默认下载版本

```
# vi /etc/yum.repos.d/mysql-community.repo

# 将 [mysql57-community] 下的 enabled 改为 1
# 将 [mysql80-community] 下的 enabled 改为 0
```

> 可以不执行,不修改的话默认下载 `MySql 8`

## 安装 `MySql`

```
yum -y install mysql-community-server
```

## 配置 `MySql`

```
# vi /etc/my.cnf

character-set-server=utf8
default-time_zone=+8:00
lower_case_table_names=1
group_concat_max_len=102400
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

> 按照实际情况配置,上述仅供参考

## 启动 `MySql` 并设置开机自启

```
systemctl enable mysqld

systemctl start mysqld
```

## 修改密码

```
# 查询默认密码
grep "password" /var/log/mysqld.log

# 使用初始密码登录mysql
mysql -uroot -p
# 输入查询到的初始密码

# 设置密码验证规则
set global validate_password_policy=0;
set global validate_password_length=4;

# 更新密码
set password=PASSWORD('1234');
```

> 密码规则可不修改, 依据实际情况处理

## 设置允许远程访问

```
use mysql;
update `user` set host='%' where `user`='root';
flush privileges;
```

> 远程访问可不修改, 依据实际情况处理

## 退出

```
exit
```
