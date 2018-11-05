# ServerStatus中文版：   

* ServerStatus中文版是一个酷炫高逼格的云探针、云监控、服务器云监控、多服务器探针~，该云监控（云探针）是ServerStatus（ https://github.com/BotoX/ServerStatus ）项目的中文（优化）版。
* 在线演示：https://tz.cloudcpp.com 

安装前请注意以下事项：
* 建议切换到非特权用户或创建一个用户。
* 需要在服务器上打开35601端口。

这个版本是91yun在原来的基础进行了以下改动：
* 增加了探测被墙的状态
* 增加了服务器连接数的统计（总连接数/2）
* 把js文件放在了本地
* 流量改成用vnstat统计当月流量（原来统计的是开机以来的总流量）
* css适配了手机移动端的显示

# 目录介绍：

* clients  客户端文件
* server   服务端文件
* web      网站文件
* other    配置文件

# 安装教程：     
---
## 服务端配置

注意这里的服务器不支持CentOS 6及以下 

### 服务器端依赖环境安装           
```
yum install -y epel-release vnstat
yum install -y vnstat
service vnstat start
chkconfig vnstat on

```

### 下载代码并试运行

```
cd /usr/local/share
git clone https://github.com/nicky1605/ServerStatus.git
cd ServerStatus/server
make
./sergate

```
如果提示端口被占用请自行解决。

### 修改服务器配置文件

修改config.json文件，注意username, password的值需要和客户端对应一致
password可以所有客户端都一样，但是username必须确保所有客户端都是唯一的
注意这里的username只作为鉴定服务器，不需要在客户端建立实际用户
name——服务器识别名
type——虚拟类型
host——节点名
location——实际位置

```
{"servers":
	[
		{
			"username": "s01",
			"name": "Mainserver 1",
			"type": "Dedicated Server",
			"host": "GenericServerHost123",
			"location": "Austria",
			"password": "some-hard-to-guess-copy-paste-password"
		},
	]
}       
```

如果要暂时禁用服务器，可以添加

```
"disabled": true
```
程序还支持命令行开关，查找帮助需要添加-h

```
    -h, --help            显示帮助命令
    -v, --verbose         冗长输出
    -c, --config=<str>    设置配置文件
    -d, --web-dir=<str>   设置web目录
    -b, --bind=<str>      绑定到地址
    -p, --port=<int>      监听端口
    
 ```
### web运行环境配置
需要先把web文件夹中的内容拷贝到apache或是nginx文件夹中，这里以nginx为例

```
cp -r ../web/ /usr/share/nginx/html

```
### 试运行

```
./sergate --config=config.json --web-dir=/usr/share/nginx/html/web/
```
在浏览器中输入 ip/web  即可查看效果


### 配置服务
如果不喜欢每次都手动启动服务，可以使用相关脚本来配置成系统服务。
#### Debian 为例
先修改init.d代码

```
vi ServerStatus/other/sergate.initd

```
根据自己服务器的相关配置文件修改以下文件中的对应项：
DAEMON_PATH——sergate文件目录
WEB_PATH——web输出目录
RUNAS——http程序用户

```
# Change this according to your setup!
DAEMON_PATH="/usr/local/share/ServerStatus/server"
WEB_PATH="/usr/share/nginx/html/web/"
DAEMON="sergate"
OPTS="-d $WEB_PATH"
RUNAS="www-data"
```

用root用户将修改好的init.d代码复制到/etc/init.d
```
cp ServerStatus/other/sergate.initd /etc/init.d/sergate

```
用root用户启动服务

```
service sergate start

```
#### CentOS 7 或Arch Linux为例

先修改service代码

```
vi ServerStatus/other/sergate.service

```
根据自己服务器的相关配置文件修改以下文件中的对应项：
WorkingDirectory——sergate文件目录
ExecStart——sergate文件地址及web输出目录
User——http程序用户
Group——http程序用户组

```
[Unit]
Description=ServerStatus Master Server
After=syslog.target
After=network.target

[Service]
Type=simple
WorkingDirectory=/usr/local/share/ServerStatus/server
User=botox.bz
Group=http
ExecStart=/usr/local/share/ServerStatus/server/sergate -d /usr/share/nginx/html/web/
ExecReload=/bin/kill -HUP $MAINPID
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=sergate

[Install]
WantedBy=multi-user.target

```
用root用户将修改好的配置文件拷贝到系统目录：

```
cp ServerStatus/other/sergate.service /etc/systemd/system/sergate.service

```
用root用户启动服务

```
systemctl start sergate

```
再将服务添加到自启动中

```
systemctl enable sergate

```
# #注意：如果用非root用户配置，需要确保该用户可以访问Web路径，并可以写入web/json目录。

## 客户端配置
如果是CentOS 6需要讲Python升级到2.7
### 服务器端依赖环境安装

CentOS6 升级Python

```
yum -y install zlib zlib-devel openssl openssl-devel xz
wget https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tar.xz
tar -xvf Python-2.7.8.tar.xz
xz -d Python-2.7.8.tar.xz
tar -xf Python-2.7.8.tar.xz

cd Python-2.7.8
./configure --prefix=/usr/local/python27
make
make install
make clean
make distclean

mv /usr/bin/python /usr/bin/python2.6.6
ln -s /usr/local/python27/bin/python2.7 /usr/bin/python

#最后将yum配置文件中/usr/bin/python修改为/usr/bin/python2.6.6 
vi /usr/bin/yum

```

### 修改配置文件client-linux.py或client-psutil.py
# #注意这里的USER跟PASSWORD需要跟服务端的config.json中用户名密码一致。
```
SERVER = "status.botox.bz"
PORT = 35601
USER = "s01"
PASSWORD = "some-hard-to-guess-copy-paste-password"
INTERVAL = 1 # Update interval

```
### 试运行
修改完毕后可以试运行
```
client-linux.py
```
无报错后可以使用nohup常驻后台。
```
nohup ./client.py &> /dev/null &
```
### 配置服务
如果不喜欢每次都手动启动服务，可以使用相关脚本来配置成系统服务。
#### Debian或CentOS 6 为例
修改/etc/rc.local文件
```
vi /etc/rc.local
```
并将以下代码输入进去设置成自启动。实际位置根据自己的服务器配置确定。

```
su -l $USERNAME -c "/path/to/client.py &> /dev/null &"

```
以下为我的配置。
```
su -l root -c "/usr/local/share/ServerStatus/clients/client.py &> /dev/null &"
```
#### CentOS 7 或Arch Linux为例

直接在服务文件夹中添加服务

```
vi /etc/systemd/system/serverstatus.service

```
在其中输入以下代码，实际位置根据自己的服务器配置确定。
```
[Unit]
Description=ServerStatus Client
After=network.target

[Service]
Type=simple
IgnoreSIGPIPE=no
User=$USERNAME
ExecStart=/path/to/client.py

[Install]
WantedBy=multi-user.target
```
以下为我的配置。
```
[Unit]
Description=ServerStatus Client
After=network.target

[Service]
Type=simple
IgnoreSIGPIPE=no
User=root
ExecStart=/usr/local/share/ServerStatus/clients/client.py

[Install]
WantedBy=multi-user.target
```
# 相关开源项目，感谢： 

* 91yun: https://github.com/91yun/ServerStatus
* ServerStatus: https://github.com/BotoX/ServerStatus
* mojeda's ServerStatus: https://github.com/mojeda/ServerStatus
* BlueVM's project: http://www.lowendtalk.com/discussion/comment/169690#Comment_169690
