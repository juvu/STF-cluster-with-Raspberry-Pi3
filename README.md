# STF-cluster-with-Raspberry-Pi3
- STF集群部署，树莓派作为子节点，使用Docker部署
- 参考2篇博文，对关键步骤细化描述，需要使用的service文件在第二个博文的github链接中
- https://testerhome.com/topics/12755
- https://testerhome.com/topics/15027
## 树莓派3配置
### 前置步骤
- 镜像下载： http://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2018-06-29/
- 写入SD卡工具： Win32DiskImager

### 树莓派ADB连接手机配置
- 安装ADB:sudo apt-get install android-tools-adb
- 配置 51-android.rules：
sudo nano /etc/udev/rules.d/51-android.rules
- 重启udev 服务：
sudo service udev restart
- ADB对外开放5037端口(telnet 测试5037端口通过）  
adb -a -P 5037 server nodaemon(正式环境不需要）

### 树莓派Docker安装
- curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
- sudo usermod -aG docker pi

### 树莓派安装相应的Docker应用
- docker pull openstf/stf-armv7l:latest
- docker pull yaming116/arm32v7-adb:latest
### 树莓派systemd 运行2个service
- 导入adbd.service/stf-provider-15.service 到/lib/systemd/system
- systemctl start stf-provider-15
- systemctl start adbd
PS:若为内网环境，需要删除每个service文件中 （ExecStartPre=/usr/bin/docker pull xxx）一行

## Ubuntu主节点配置
### Docker安装及镜像获取
Docker安装与树莓派上相同，需要外网环境  
拉取如下镜像：  
- docker pull openstf/stf:latest  
- docker pull sorccu/adb:latest   
- docker pull rethinkdb:latest   
- docker pull nginx:latest  

### 方法一：主节点直接连接手机 （单机部署）
docker run -d --name rethinkdb -v /srv/rethinkdb:/data --net host rethinkdb rethinkdb --bind all --cache-size 8192 --http-port 8090  
docker run -d --name adbd --privileged -v /dev/bus/usb:/dev/bus/usb --net host sorccu/adb:latest  
docker run -d --name stf --net host openstf/stf stf local --public-ip ${Master_IP} --allow-remote

### 复杂方法二： 开启相关服务-组建集群连接树莓派3
- 导入 /Ubuntu主节点 所有service文件到 /lib/systemd/system  
- 导入 opt/conf/nginx.conf 到 /opt/conf/nginx.conf  
- 依次执行 run.txt中所有命令

PS:若为内网环境，需要删除每个service文件中 （ExecStartPre=/usr/bin/docker pull xxx）一行
