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
配置 usermysql.json 里的数据库链接，NODE_ID就是节点ID，对应面板后台里添加的节点的自增ID，所以请先把面板搭好，搭好后进后台添加节点
````
###### SSR服务端配置

````

````

## 网卡流量监控一键脚本
````
wget -N --no-check-certificate https://raw.githubusercontent.com/ssrpanel/ssrpanel/master/server/deploy_vnstat.sh;chmod +x deploy_vnstat.sh;./deploy_vnstat.sh
````

###### SSR服务端一键自动部署
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
