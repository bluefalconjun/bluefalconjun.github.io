_*[Back Home](https://bluefalconjun.github.io)*_  
***  

**> on Ubuntu (server&client)**  

_install the ss server/client in python version._  

    sudo apt-get install python-pip privoxy
    sudo pip install shadowsocks
    sudo pip install -U shadowsocks  //update ss   

_config format for server/client._  

    /etc/ss.cfg:  
    {  
     "server":"xx.xx.xx.xx",  
     "server_port":xx,  
     "local_port":xx,    //clients only
     "password":"xxxxx",  
     "method":"aes-256-cfb",  
    }

_use **supervisor** to auto start **ssserver** on host._  

    /etc/supervisor/supervisord.conf:
    [program:shadowsocks]
    command=ssserver -c /etc/ss.cfg
    autostart=true
    autorestart=true
    user=root

_start **sslocal** on client_  

    sudo sslocal -c /etc/ss.cfg -d start  


_optimize for server

/etc/security/limits.conf:

	* soft nofile 51200
	* hard nofile 51200

/etc/pam.d/common-session:

	session required pam_limits.so

/etc/profile:

	ulimit -SHn 51200

/etc/sysctl.conf:

	fs.file-max = 51200
	net.core.rmem_max = 67108864
	net.core.wmem_max = 67108864
	net.core.netdev_max_backlog = 250000
	net.core.somaxconn = 4096
	net.ipv4.tcp_syncookies = 1
	net.ipv4.tcp_tw_reuse = 1
	net.ipv4.tcp_tw_recycle = 0
	net.ipv4.tcp_fin_timeout = 30
	net.ipv4.tcp_keepalive_time = 1200
	net.ipv4.ip_local_port_range = 10000 65000
	net.ipv4.tcp_max_syn_backlog = 8192
	net.ipv4.tcp_max_tw_buckets = 5000
	net.ipv4.tcp_rmem = 4096 87380 67108864
	net.ipv4.tcp_wmem = 4096 65536 67108864
	net.ipv4.tcp_mtu_probing = 1
	net.ipv4.tcp_congestion_control = hybla

***

**> on Windows (client)**  
**[shadowsocks-csharp @ _github_](https://github.com/shadowsocks/shadowsocks-csharp)**  
***

**> on Android (client)**  
**[Shadowsocks @ _google playstore_](https://play.google.com/store/apps/details?id=com.github.shadowsocks)**  

***  
_*[Back Home](https://bluefalconjun.github.io)*_  
