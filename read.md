## 安装步骤
#### 环境要求
````
PHP 7.1 （必须）
MYSQL 5.5 （推荐5.6+）
内存 1G+ 
磁盘空间 10G+
KVM

注意：
PHP必须开启gd、fileinfo组件

````
#### 拉取代码
````
cd /home/wwwroot/
git clone https://github.com/a327958099/myssrserver.git
//#地址二
git clone https://github.com/ssrpanel/ssrpanel.git
````

#### 先配置数据库
````
mysql 创建一个数据库，然后自行导入sql\db.sql
config\database.php 中的mysql选项自行配置数据库信息
````

#### 编辑php.ini
````
找到php.ini
vim /usr/local/php/etc/php.ini

搜索disable_function
删除proc_开头的所有函数
````


#### NGINX配置文件加入
````
root /data/wwwroot/default/ssrpanel/public; #站点地址路径，注意：必须是ssrpanel/public目录

#server内部加入以下代码：
location / {
    try_files $uri $uri/ /index.php$is_args$args;
}
````
#### 重启NGINX和PHP-FPM
````
service nginx restart
service php-fpm restart
````

#### 配置一下
````
cd ssrpanel/
php composer.phar install  #安装依赖
php artisan key:generate
chown -R www:www storage/
chmod -R 777 storage/
````

#### 出现500错误
````
理论上操作到上面那些步骤完了应该是可以正常访问网站了，如果网站出现500错误，请看WIKI，很有可能是fastcgi的错误
请看WIKI：https://github.com/ssrpanel/ssrpanel/wiki/%E5%87%BA%E7%8E%B0-open_basedir%E9%94%99%E8%AF%AF
修改完记得重启NGINX和PHP-FPM
````

## 定时任务（所有自动发邮件的地方都要用到，所以请务必配置）
````
**编辑crontab**
crontab -e

然后加入如下（请自行修改ssrpanel路径）
* * * * * php /home/wwwroot/ssrpanel/artisan schedule:run >> /dev/null 2>&1
````

#### 发送邮件配置
````
config\mail.php 修改其中的配置
````

## 日志分析（目前仅支持单机单节点）
````
找到SSR服务端所在的ssserver.log文件
进入ssrpanel所在目录，建立一个软连接，并授权
cd /home/wwwroot/ssrpanel/public/storage/app/public
ln -S ssserver.log /root/shadowsocksr/ssserver.log
chown www:www ssserver.log
````

## SSR部署
###### 手动部署
````
cp server/ssr-3.4.0.zip /root/
cd /root
unzip ssr-3.4.0.zip
cd shadowsocksr
sh initcfg.sh
把 userapiconfig.py 里的 API_INTERFACE 设置为 glzjinmod
把 user-config.json 里的 connect_verbose_info 设置为 1
````
###### 安装依赖(cymysql)
````
./setup_cymysql.sh
````
###### 初始化配置
````
./initcfg.sh
````
###### SSR服务端usermysql.json配置

````
{
    "host": "112.74.102.56", //数据库地址
    "port": 3306, //端口
    "user": "wangfei", //用户名
    "password": "7DMKneljBW", //密码
    "db": "ss2", //数据库名
    "node_id": 1, //NODE_ID就是节点ID，对应ss_node里面的ID
    "transfer_mul": 1.0,
    "ssl_enable": 0,
    "ssl_ca": "",
    "ssl_cert": "",
    "ssl_key": ""
}
````
###### SSR服务端user-config.json配置

````
{
    "server": "192.168.31.253",
    "server_ipv6": "::",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "m",
    "method": "aes-128-ctr",  #加密方式
    "protocol": "auth_chain_a", #认证协议，强烈推荐auth_chain_a// 例如 auth_aes128_md5 或者 auth_aes128_sha1，目前只有这两种
    "protocol_param": "", #协议参数：每个端口最大设备连接数（建议最少2个），比如 限制最大 5个设备同时链接，那么改为："protocol_param": "5",协议为原版origin协议或其他协议兼容原版协议时无效
    "obfs": "tls1.2_ticket_auth", #混淆插件，强烈推荐tls1.2_ticket_auth
    "obfs_param": "",
    "speed_limit_per_con": 0, #单线程限速
    "speed_limit_per_user": 0, #端口总限速

    "additional_ports" : {}, // only works under multi-user mode
    "additional_ports_only" : false, // only works under multi-user mode
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 1,
    "redirect": "",
    "fast_open": false
}
````
#### 服务端运行与停止

```
python server.py
```
这时可查看有运行情况，检查有没有错误。如果服务端没有错误，而连接不上，需要检查iptables或firewall(centos7)的防火墙配置

后台运行（无log，ssh窗口关闭后也继续运行）

```
./run.sh
```
后台运行（输出log，ssh窗口关闭后也继续运行）

```
./logrun.sh
```
后台运行时查看运行情况

```
./tail.sh
```
停止运行

```
./stop.sh
```
注：通过脚本运行默认日志会保存在根目录的ssserver.log，可手动查看。

#### 开机启动
首先设置开机启动文件的权限，并打开该文件。

**Centos系统：**

```
chmod +x /etc/rc.d/rc.local
vi /etc/rc.d/rc.local
```
**Ubuntu/Debian系统：**

```
chmod +x /etc/rc.local
vi /etc/rc.local
```
然后在 exit 0 这一句代码（只有ubuntu/debian有这个 exit 0）的前面加上 下面这句代码

```
bash /root/shadowsocksr/run.sh
```
#### 禁用防火墙

```
//centos 7 设置
systemctl stop firewalld.service  //关闭防火墙
systemctl disable firewalld.service  //禁止防火墙开机启动
```

#### SSR服务端限制设备连接数
限制设备连接数的这个功能，很早就有了，就是修改协议参数： protocol_param
找到协议参数（参数为空 "" 时，默认限制 64个设备数）

```
"protocol_param": "",
```
在协议参数中设置你要限制 每个端口最大设备连接数（建议最少2个），比如 限制最大 5个设备同时链接，那么改为：

```
"protocol_param": "5",
//注意：协议参数仅在服务端 协议设置(protocol)为 非原版(origin)协议并不兼容原版(_compatible) 时才有效！
```
如果你服务端 协议设置(protocol)的是 原版(origin) 时，设备数限制无效。

如果你服务端 协议设置(protocol)的是 协议兼容原版 ，那么当用户使用原版协议(origin)连接Shadowsocks账号时，设备数限制无效。

###### SSR服务端限制端口速度
新增的两个参数分别是（参数为 0 时，默认代表不限速）：

```
"speed_limit_per_con": 0,
"speed_limit_per_user": 0,
```
单位是 KB/S ，也就是我们平时下载文件的速度单位，我们家庭宽带100兆就是： 100Mbps / 8 = 12.5MB/S * 1024 =12800KB/S 。
比如我们要设置 单线程限速 1MB/S ，端口总限速 3MB/S ，那么就这样写：

```
"speed_limit_per_con": 1024,
"speed_limit_per_user": 3072,
//speed_limit_per_con 指的是，单线程限速。
//speed_limit_per_user 指的是，端口总限速。
```


## 网卡流量监控一键脚本
````
wget -N --no-check-certificate https://raw.githubusercontent.com/ssrpanel/ssrpanel/master/server/deploy_vnstat.sh;chmod +x deploy_vnstat.sh;./deploy_vnstat.sh
````
## 安装谷歌BBR加速

```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh

chmod +x bbr.sh

./bbr.sh
```
###### BBR加速查询

```
uname -r  //查看内核版本，含有 4.13 就表示 OK 了

sysctl net.ipv4.tcp_available_congestion_control //返回值一般为：net.ipv4.tcp_available_congestion_control = bbr cubic reno

sysctl net.ipv4.tcp_congestion_control //返回值一般为：net.ipv4.tcp_congestion_control = bbr

sysctl net.core.default_qdisc  //返回值一般为：net.core.default_qdisc = fq

lsmod | grep bbr  //返回值有 tcp_bbr 模块即说明 bbr 已启动。注意：并不是所有的 VPS 都会有此返回值，若没有也属正常。

```


## SSR服务端一键自动部署
````
wget -N --no-check-certificate https://raw.githubusercontent.com/ssrpanel/ssrpanel/master/server/deploy_ssr.sh;chmod +x deploy_ssr.sh;./deploy_ssr.sh
````

## 更新代码
````
chmod a+x update.sh && sh update.sh

如果每次更新都会出现数据库文件被覆盖
请先执行一次 chmod a+x fix_git.sh && sh fix_git.sh
````

## 单端口多用户
````
编辑节点的 user-config.json 文件：
vim user-config.json

将 "additional_ports" : {}, 改为以下内容：
"additional_ports" : {
    "80": {
        "passwd": "统一认证密码", // 例如 SSRP4ne1，不要出现除大小写字母数字以外的任何字符
        "method": "统一认证加密方式", // 例如 aes-128-ctr
        "protocol": "统一认证协议", // 例如 auth_aes128_md5 或者 auth_aes128_sha1，目前只有这两种
        "protocol_param": "#",
        "obfs": "tls1.2_ticket_auth_compatible",
        "obfs_param": ""
    },
    "443": {
        "passwd": "统一认证密码",
        "method": "统一认证加密方式",
        "protocol": "统一认证协议",
        "protocol_param": "#",
        "obfs": "tls1.2_ticket_auth_compatible",
        "obfs_param": ""
    }
},

保存，然后重启SSR服务。
客户端设置：

远程端口：80
密码：password
加密方式：aes-128-ctr
协议：auth_aes128_md5
混淆插件：tls1.2_ticket_auth
协议参数：1026:@123 (SSR端口:SSR密码)

或

远程端口：443
密码：password
加密方式：aes-128-ctr
协议：auth_aes128_sha1
混淆插件：tls1.2_ticket_auth
协议参数：1026:@123 (SSR端口:SSR密码)

经实测账号的协议可以是：auth_chain_a，建议节点后端使用auth_sha1_v4_compatible，方便兼容

注意：如果想强制所有账号都走80、443这样自定义的端口的话，记得把 user-config.json 中的 additional_ports_only 设置为 true
警告：我测试单端口多用户时发现锐速无法正常加速（可能是我的问题），可以换BBR试试
````

## 说明
````
1.多节点账号管理面板
2.需配合SSR 3.4 Python版后端使用
3.强大的管理后台、美观的界面、简单易用的开关、支持移动端自适应
4.内含简单的购物、优惠券、流量兑换、邀请码、推广返利&提现、文章管理、工单等模块
5.节点支持分组，不同级别的用户可以看到不同级别分组的节点
6.SS配置转SSR配置，轻松一键导入SS账号
7.流量日志、单机单节点日志分析功能，知道用户最近都看了哪些网站
8.强大的定时定时任务
9.所有邮件投递都有记录
10.账号临近到期、流量不够都会自动发邮件提醒，自动禁用到期、流量异常的账号
11.后台一键添加加密方式、混淆、协议、等级
12.强大的后台一键配置功能
13.屏蔽常见爬虫
14.支持单端口多用户
15.账号、节点24小时和近30天内的流量监控
````
**修改服务器的最大连接数**

如果运行一段时间后，你发现服务器无法连接，同时ssh连上去后，执行

```
netstat -ltnap | grep -c CLOSE_WAIT
```
显示的数值很大（超过50是严重不正常），那么请修改服务器的最大连接数

```
//ubuntu/centos
vim /etc/security/limits.conf
//添加两行

* soft nofile 32768
* hard nofile 131072
```

## 预览（这个是第一版的预览，已过时）
![Markdown](http://i4.bvimg.com/1949/aac73bf589fbd785.png)
![Markdown](http://i4.bvimg.com/1949/a7c21b7504805130.png)
![Markdown](http://i4.bvimg.com/1949/ee4e72cab0deb8b0.png)
![Markdown](http://i4.bvimg.com/1949/ee21b577359a638a.png)
![Markdown](http://i1.ciimg.com/1949/6741b88c5a02d550.png)
![Markdown](http://i1.ciimg.com/1949/a12612d57fdaa001.png)
![Markdown](http://i1.ciimg.com/1949/c5c80818393d585e.png)
![Markdown](http://i1.ciimg.com/1949/c52861d84ed70039.png)
![Markdown](http://i1.ciimg.com/1949/83354a1cd7fbd041.png)
![Markdown](http://i1.bvimg.com/1949/13b6e4713a6d29c2.png)
