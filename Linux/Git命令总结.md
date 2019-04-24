# Git命令常用系列
Git是一个分布式的版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理  
1. 基本操作   
    ```
    git init    //初始化本地仓库

    //设置系统级别签名
    git config --global user.name 'username'
    git config --global user.email 'username@163.com'
    //设置项目级别签名
    git config user.name 'username'
    git config user.email 'username@163.com'

    git status      //查看工作区、暂存区的状态

    //将新建、修改内容添加到暂存区
    git add [filename] 

    //提交文件
    git commit -m '说明' [filename]     //不写filename即为提交所有暂存区文件
    ```
2. 分支管理
    ```
    git branch -v   查看所有分支
    git branch [branchname]     创建分支
    git checkout [branchname]   切换分支
    #合并分支
    git checkout [branchname]   1.首先切换到接收合并的分支
    git marge [branchname]      2.marge需要合并的分支
    ```
3. 忽略文件     
在实际项目开发中，有些文件是不需要上传的，所以就需要设置忽略
    ```
    touch .gitignore    //创建忽略文件，将需要忽略上传的文件或目录名写入到此文件中即可
    ```