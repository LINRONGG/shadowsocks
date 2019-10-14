:)


1. 安装依赖
ubuntu 18.04 或更高：

apt update
apt install python-pip git libssl-dev python-dev libffi-dev libsodium-dev vim -y
ubuntu 16.04 :

apt update
apt install python-pip git libssl-dev python-dev libffi-dev software-properties-common vim -y
add-apt-repository ppa:ondrej/php -y && apt update
apt install libsodium-dev -y
下载后端程序
git clone -b manyuser https://github.com/NimaQu/shadowsocks
cd shadowsocks
安装依赖
pip install -r requirements.txt
2. 部署后端程序
创建配置文件
cp apiconfig.py userapiconfig.py
cp config.json user-config.json
修改配置文件
vim userapiconfig.py
配置文件填写正确后，执行 python server.py 进行调试，如 log 输出正常无报错，前端节点上线，则为配置正确，执行 bash run.sh 即可
3. 配置 systemd
cp -r /root/shadowsocks/ssr.service /etc/systemd/system/
如需启动多个只需复制一份该 service 改成不同文件名并修改 WorkingDirectory= 即可

启动程序并加入开机自启动
systemctl start ssr
systemctl enable ssr
echo "sshd: ALL" > /etc/hosts.allow
#防止 auto block 了自己无法连接 ssh
4. 更新内核以及开启 TCP BBR(可选)
Ubuntu 16.04 需要更新内核（内核版本4.9 以下）16.04 以上可跳过
apt install --install-recommends linux-generic-hwe-16.04 -y
update-grub  
reboot
开启 BBR
将 BBR 写入内核配置并保存生效
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
echo "vm.swappiness = 10" >> /etc/sysctl.conf
echo "vm.vfs_cache_pressure = 50" >> /etc/sysctl.conf
echo "net.core.default_qdisc = fq_codel" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control = bbr" >> /etc/sysctl.conf
echo "net.ipv4.tcp_fastopen = 3" >> /etc/sysctl.conf
sysctl -p
检查生效情况
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
lsmod | grep bbr
如果结果都有 bbr, 则证明你的内核已开启 bbr。看到有 tcp_bbr 模块即说明 BBR 已启动

userapiconfig.py，各项配置的意思
# Config

#管理面板对应的节点id

NODE_ID = 1

#自动化测速，为0不测试，此处以小时为单位，要和 ss-panel 设置的小时数一致

SPEEDTEST = 6

#云安全，自动上报与下载封禁IP，1为开启，0为关闭

CLOUDSAFE = 1

#自动封禁SS密码和加密方式错误的 IP，1为开启，0为关闭

ANTISSATTACK = 0

#是否接受上级下发的命令，如果你要用这个命令，请参考我之前写的东西，公钥放在目录下的 ssshell.asc

AUTOEXEC = 1

多端口单用户混淆域名，需要和前端 .config.php 一致

MU_SUFFIX = ‘zhaoj.in’

多端口单用户混淆域名前缀参数，需要和前端 .config.php 一致。

MU_REGEX = ‘%5m%id.%suffix’

#不明觉厉

SERVER_PUB_ADDR = ‘127.0.0.1’ # mujson_mgr need this to generate ssr link

#对接方式，glzjinmod (数据库方式连接)，modwebapi (webapi)

API_INTERFACE = ‘glzjinmod’

#mudb，不要管

MUDB_FILE = ‘mudb.json’

# 你的站点地址，如果开启了 https 需要修改为https ，

WEBAPI_URL = ‘https://zhaoj.in‘

# .config.php里设置的对接口令

WEBAPI_TOKEN = ‘glzjin’

# MYSQL 地址

MYSQL_HOST = ‘127.0.0.1’

# MYSQL 端口

MYSQL_PORT = 3306

# MYSQL 登陆用户名

MYSQL_USER = ‘ss’

# MYSQL 登陆密码

MYSQL_PASS = ‘ss’

# 面板所在的数据库名

MYSQL_DB = ‘shadowsocks’

# 是否启用SSL连接，0为关，1为开

MYSQL_SSL_ENABLE = 0

# 客户端证书目录，请看 https://github.com/glzjin/shadowsocks/wiki/Mysql-SSL%E9%85%8D%E7%BD%AE

MYSQL_SSL_CERT = ‘/root/shadowsocks/client-cert.pem’

MYSQL_SSL_KEY = ‘/root/shadowsocks/client-key.pem’

MYSQL_SSL_CA = ‘/root/shadowsocks/ca.pem’

# API，不用管

API_HOST = ‘127.0.0.1’

API_PORT = 80

API_PATH = ‘/mu/v2/’

API_TOKEN = ‘abcdef’

API_UPDATE_TIME = 60

# Manager 不用管

MANAGE_PASS = ‘ss233333333’

#如果要在其他服务器中管理，则应将此值设置为此服务器IP。

MANAGE_BIND_IP = ‘127.0.0.1’

#请确保此端口空闲。

MANAGE_PORT = 23333

#安全设置，限制在线 IP 数所需，下面这个参数随机设置，并且所有节点需要保持一致。

IP_MD5_SALT = ‘randomforsafety’
