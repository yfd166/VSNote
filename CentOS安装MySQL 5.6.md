# CentOS在线安装MySQL
![MySQL](/image/MySQL_icon.gif)
MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下产品。MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件。       
下面介绍CentOS安装MySQL 5.6的详细过程
1. 安装MySQL的yum repository
    ```shell
    wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
    ```
    如果没有wget使用下面的命令安装wget，在进行上一步
    ```shell
    sudo yum install wget
    ```
    ![yum](/image/安装MySQL的yum仓库.jpg)
2. 下载rpm包
    ```shell
    rpm -ivh mysql-community-release-el7-5.noarch.rpm
    ```
    ![rpm](/image/rpm.png)
3. 安装MySQL服务
    ```shell
    sudo yum install mysql-server
    ```
    ![install](/image/install.png)
    安装过程中会询问类似于是否继续安装的信息，全部输入'y'即可
4. 启动MySQL服务
    ```shell
    #启动
    sudo systemctl start mysqld     
    #关闭
    sudo systemctl stop mysqld
    #重启
    sudo systemctl restart mysqld
    #查看MySQL运行状态
    sudo systemctl status mysqld
    ```
    ![启动MySQL和查看MySQL运行状态](/image/MySQL_start_and_status.png)
5. 修改MySQL root用户密码，并设置root用户可远程登陆
    ```shell
    #登陆MySQL
    mysql -uroot -p
    ```
    ![Login](/image/login_mysql.png)
    我们安装的是MySQL5.6，并没有默认密码，所以第一次登陆不需要密码输入直接回车就可以登陆到MySQL了。
    ```shell
    #设置root用户密码
    set password = password('root');
    ```
    ![setpassword](/image/set_password.png)
    ```sql
    # 授权远程登陆，注意将youpassowrd修改为之前设置的用户密码
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
    # 重载授权表
    FLUSH PRIVILEGES;
    # 退出MySQL
    exit;
    ```
    执行完后，输入exit退出MySQL，重启MySQL
    ```shell
    systemctl restart mysqld
    ```
