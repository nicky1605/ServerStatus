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

# 安装教程：     
---
## 服务端配置
***          
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

---
以下待修改
---

【客户端配置】
```
yum -y install epel-release
yum -y install python-pip
yum clean all
yum -y install gcc
yum -y install python-devel
pip install psutil
mkdir -p /home/serverstatus
cd /home/serverstatus
wget https://github.com/91yun/ServerStatus-1/raw/master/clients/client-psutil.py
```
编辑客户端配置文件
    vim client-psutil.py
```
SERVER = "127.0.0.1" #改成呢你的服务器地址
PORT = 3561
USER = "USER" #改成唯一的客户端用户名，服务器根据这个字段判断是哪台服务器
PASSWORD = "USER_PASSWORD" #修改你的密码，和其他客户端可以是相同的
```
启动客户端
```
nohup python /home/serverstatus/client-psutil.py &> /dev/null &
```

# 相关开源项目，感谢： 

* ServerStatus：https://github.com/BotoX/ServerStatus
* mojeda: https://github.com/mojeda 
* mojeda's ServerStatus: https://github.com/mojeda/ServerStatus
* BlueVM's project: http://www.lowendtalk.com/discussion/comment/169690#Comment_169690
