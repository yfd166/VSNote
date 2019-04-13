## Linux系统通用 安装JDK1.8
1. 官网下载JDK      
[JDK官方下载链接](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)     需要同意协议    
![点击同意协议](/image/Ubuntu安装JDK官网截图.jpg)
选择对应的版本开始下载
2. 解压
可以直接在Ubuntu资源管理器中解压，也可以使用命令解压，这里我们使用命令解压
* 切换到之前保存文件的目录
    ```shell
    cd downloads/           
    ```
* 移动文件到usr local目录下
    ```shell
    sudo mv jdk-8u201-linux-x64.tar.gz /usr/lcoal      
    ```
* 解压文件
    ```shell
    cd /usr/local/
    sudo tar -zxvf jdk-8u201-linux-x64.tar.gz        
    ```
3. 配置环境变量     
* 编辑配置文件
    ```shell
    sudo vim /etc/profile
    ```
    有些发行版可能默认编辑器不是vim，可以使用默认编辑器或者下载vim
* 按i键进入编辑模式 在文件末尾加入以下代码
    ```shell
    export JAVA_HOME=/usr/local/jdk1.8.0_201 	//jdk所在目录
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${JAVA_HOME}/bin:$PATH
    ```
    之后保存退出：Esc   :wq
* 退出vim后刷新配置文件
    ```shell
    sudo source /etc/profile
    ```
4. 用以下命令测试是否安装成功
    ```shell
    java -version
    java
    javac
    ```
    输出以下信息既是安装成功    
![测试JDK是否安装成功](/image/测试JDK是否安装成功.jpg)