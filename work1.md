Manjaro 安装 Apache、Mysql、PHP 环境
Manjaro 安装 Apache、Mysql、PHP 环境，也同样没有 Ubuntu 省心，Ubuntu 基本上通过 apt install 后就能满足基本的使用了，无需做什么设置，而 Manjaro 却不行，安装 Mysql 的过程没有设置 root 用户名和密码静悄悄的，安装完后却一直开不起来服务，找了许多地方最后在 Manjaro 官方论坛找到教程，最后 PHP 安装后，本地安装 WordPress 死活第二步 500 错误，最后看了 Apache 错误日志才知道原来 Mysql_connect() 错误，比较蛋疼。

安装软件前 update
sudo pacman -Syu

安装 Apache
sudo pacman -S apache

#Apache配置文件位置
/etc/httpd/conf/httpd.conf

#http 服务文件夹
/srv/http/

#查看 Apache 状态和版本信息

sudo systemctl status httpd
apachectl -v 或 httpd -v
#设置开机启动和重启 Apache 服务

sudo systemctl enable httpd
sudo systemctl restart httpd
#如果 Apache 启动提示 Could not reliably determine the server’s fully qualified domain name 错误
在 Apache 配置文件 /etc/httpd/conf/httpd.conf 里修改或加入一行
ServerName localhost:80
然后重启 Apache

安装 Mysql
sudo pacman -S mysql

#初始化MariaDB数据目录，没有这步 mysql 就不能用
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

#查看mysql状态
sudo systemctl status mysqld

#开机启动mysql服务

sudo systemctl enable mysqld
sudo systemctl start mysqld
#设置mysql root用户密码
sudo mysql_secure_installation

#默认密码是空的，回车后设置root用户密码，后面就回车回车

安装 PHP
sudo pacman -S php php-apache

#修改apache配置
sudo nano /etc/httpd/conf/httpd.conf

#注释掉
LoadModule mpm_event_module modules/mod_mpm_event.so

#去掉下一行的注释
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so

#在配置文件最后面添加

LoadModule php7_module modules/libphp7.so
AddHandler php7-script php
Include conf/extra/php7_module.conf
#重启apache
sudo systemctl restart httpd

以上设置来自 forum.manjaro.org 里面还有 PhpMyAdmin 的安装，我就没装了。

安装 WordPress 填完数据库信息后下一步 500 错误问题
搭配了环境后当然要安装 WordPress 可在第一步填写了数据库信息后点下一步死活 500 错误，一直以为是文件夹权限的问题，折腾了许久还是没有效果，也修改了 apache 配置文件里的 AllowOverride 和 Require all，也是没用，不过在看 apache 配置文件的时候看到日志文件位置 /var/log/httpd/error_log，打开 apache 错误日志一看，一溜的提示都是 PHP Fatal error: Uncaught Error: Call to undefined function mysql_connect()，原来是这么个鬼。

于是找到 php.ini 文件，把和数据库有关的 extension 前面的分号 ; 全部删除，保存后重启 apache 服务，500 错误问题立马解决。
