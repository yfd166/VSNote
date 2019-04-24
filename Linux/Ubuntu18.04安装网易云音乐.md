安装：在官网下载Linux平台客户端，Ubuntu16.04版本

问题：网易云音乐1.0 Ubuntu版本在Ubuntu18.04中安装之后点击图标打不开，只能在终端添加sudo权限启动。
解决方案：
```
sudo gedit /etc/sudoers 
//修改/etc/sudoders文件，加一行
YOURNAME ALL = NOPASSWORD: /usr/bin/netease-cloud-music
//YOURNAME为你的用户名

sudo gedit /usr/share/application/netease-cloud-music.desktop
//修改Exec的值为
sudo netease-cloud-music %U
```
经过以上步骤后一个是百分百解决的