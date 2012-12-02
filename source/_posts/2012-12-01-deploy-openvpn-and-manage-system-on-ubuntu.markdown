---
layout: post
title: "在Ubuntu上部署openvpn服务以及其管理系统"
date: 2012-12-01 17:58
comments: true
categories: dev openvpn ubuntu linux
---
首先描述一下我的需求:

* 使用MySQL进行用户管理
* 可以实现流量监测
* 使用用户名及密码进行登陆
* 由于要分发给朋友，所以希望有设置账号密码的功能

###安装OpenVPN

安装相关软件
    
    apt-get update
    apt-get install openvpn udev mysql-server libpam-mysql
        
    
OK，这样已经安装了所需的所有软件,现在开始进行配置。

将`easy_rsa`密钥工具包拷贝到配置目录备用

    cp -R /usr/share/doc/openvpn/examples/easy-rsa/ /etc/openvpn

修改`/etc/openvpn/easy_rsa/var`文件，将默认配置修改为你所需要的值

    export KEY_COUNTRY="CN"
    export KEY_PROVINCE="SH"
    export KEY_CITY="SH"
    export KEY_ORG="lalala"
    export KEY_EMAIL="vvtommy@gmail.com"

###配置你的密钥
首先初始化[PKI]

    cd /etc/openvpn/easy-rsa/2.0/
    source . /etc/openvpn/easy-rsa/2.0/vars
    . /etc/openvpn/easy-rsa/2.0/clean-all
    . /etc/openvpn/easy-rsa/2.0/build-ca

    cd /etc/openvpn/easy-rsa/2.0/
    source . /etc/openvpn/easy-rsa/2.0/vars
    . /etc/openvpn/easy-rsa/2.0/clean-all
    . /etc/openvpn/easy-rsa/2.0/build-ca

接着生成服务器私钥。会有一些问题和提示，选择默认和Y就可以了。

    . /etc/openvpn/easy-rsa/2.0/build-key-server server

最后生成Diffie Hellman Parameters。

    . /etc/openvpn/easy-rsa/2.0/build-dh
    Generating DH parameters, 1024 bit long safe prime, generator 2
    This is going to take a long time

如果一切正常，会出现如上所示的信息。
密钥的配置基本就是这样，最后把生成的密钥文件和easy-rsa提供的服务器端配置文件样本拷贝到配置文件夹下。

    cd /etc/openvpn/easy-rsa/2.0/keys
    cp ca.crt ca.key dh1024.pem server.crt server.key /etc/openvpn
    cd /usr/share/doc/openvpn/examples/sample-config-files
    gunzip -d server.conf.gz
    cp server.conf /etc/openvpn/
    cp client.conf ~/
    cd ~/

配置OpenVPN链接
密钥的配置保证了客户端和VPN服务器的链接。还要配置OpenVPN对网络包地转发规则。
修改`/etc/openvpn/server.conf`,添加如下语句

    push "redirect-gateway def1"

修改`/etc/sysctl.conf`,添加如下语句

    net.ipv4.ip_forward=1

为VPN会话添加下列变量。

    echo 1 > /proc/sys/net/ipv4/ip_forward
    iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -A FORWARD -s 10.8.0.0/24 -j ACCEPT
    iptables -A FORWARD -j REJECT
    iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE

为了保证每次启动这边规则都生效，可以把后面四个语句添加到`/etc/rc.local`中去。
除了IP包地转发规则，我们还需要处理来自客户端的dns请求，安装dnsmasq来解决这个问题。

    apt-get update
    apt-get install dnsmasq

修改`/etc/openvpn/server.conf`,添加如下语句

    push "dhcp-option DNS 10.8.0.1"
    push "dhcp-option DNS 8.8.8.8"
    push "dhcp-option DNS 8.8.4.4"


OK，一切搞定，启动OpenVPN服务吧。

    /etc/init.d/openvpn restart

###使用MySQL进行用户认证以及流量管理

登录MySQL
    
    mysql -uroot -p

为pam_mysql初始化数据库。

{% gist 4187735 %}

编辑 `/etc/pam.d/openvpn` 为pam_mysql配置数据库

    auth            sufficient      pam_mysql.so \
    user=openvpn passwd=openvpn host=localhost db=openvpn \
    table=user usercolumn=username passwdcolumn=password \
    where=active=1 sqllog=0 crypt=1
     
    account         required        pam_mysql.so \
    user=openvpn passwd=openvpn host=localhost db=openvpn \
    table=user usercolumn=username passwdcolumn=password \
    where=active=1 sqllog=0 crypt=1
其中数据库、用户名、密码按照自己的实际情况设置。其中`crypt`表示密码在数据库中加密存储的方式:

* 0 (or “plain”)：不加密，明文存储。不推荐使用。
* 1 (or “Y”)：使用crypt(3)函数，相当于MySQL中的ENCRYPT()函数。
* 2 (or “mysql”)：使用MySQL的PASSWORD()函数。PAM可能与MySQL的函数不同，不推荐使用。
* 3 (or “md5″)：使用MD5。
* 4 (or “sha1″)：使用SHA1。

重启saslauthd

    /etc/init.d/saslauthd restart

如果出现以下提示：

    To enable saslauthd, edit /etc/default/saslauthd and set START=yes (warning).

说明saslauthd未配置成自动启动，则需修改`/etc/default/saslauthd`文件，将`START=no`改为`START=yes`，再重启服务即可。

这样，就可以在`openvpn`表中对用户进行管理了，例如添加用户:

    USE openvpn;
    INSERT INTO user (username,password) VALUES ('test',ENCRYPT('123456'));

退出后，可以运行以下命令：

    testsaslauthd -u test -p 123456 -s openvpn

如果显示

    0: OK "Success."

则说明配置成功。否则，请根据 `/var/log/auth.log` 日志查找原因。

复制OpenVPN PAM认证模块

    cp /usr/lib/openvpn/openvpn-auth-pam.so /etc/openvpn/

编辑 `/etc/openvpn/server.conf` 添加以下配置以配置pam

    plugin ./openvpn-auth-pam.so openvpn
    client-cert-not-required
    username-as-common-name

为了防止世界被破坏，我们在 `/etc/openvpn/server.conf` 中添加如下配置

    cipher AES-256-CBC

重启 OpenVPN

    /etc/init.d/openvpn restart

OK, 现在已经建立起一套基于MySQL认证的OpenVPN系统

###流量控制

> 未完待续











1. [Secure Communications with OpenVPN on Ubuntu 10.04 (Lucid)][]
2. [Tunnelblick][]
3. [OpenVPN GUI for Windows][]


[PKI]: http://zh.wikipedia.org/wiki/PKI "公开金钥基础架构"
[Secure Communications with OpenVPN on Ubuntu 10.04 (Lucid)]: http://library.linode.com/networking/openvpn/ubuntu-10.04-lucid "Secure Communications with OpenVPN on Ubuntu 10.04 (Lucid)"
[Tunnelblick]: http://code.google.com/p/tunnelblick/ "Tunnelblick"
[OpenVPN GUI for Windows]: http://openvpn.se/download.html "OpenVPN GUI for Windows"
[1]: http://google.com/        "Google"
[2]: http://search.yahoo.com/  "Yahoo Search"
[3]: http://search.msn.com/    "MSN Search"