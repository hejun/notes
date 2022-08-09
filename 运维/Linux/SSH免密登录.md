## SSH免密登录

- 配置 `hosts`
  
  ```sh
  vi /etc/hosts
  ```
  
  ```sh
  192.168.2.2 node-01
  192.168.2.3 node-02
  192.168.2.4 node-03
  ```
  > 根据主机实际IP与 `hostname` 进行配置, 在文件后新启一行添加
                                                                                                                               
- 生成密钥

  ```sh
  ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  ```
  
- 分发密钥

  ```sh
  ssh-copy-id -i ~/.ssh/id_rsa.pub node-01
  ```
  > 在每台机器上都得执行上述命令, `node-01` 即为之前配置的主机名，每个主机名都需执行一边
