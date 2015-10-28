_*[Back Home](https://github.com/bluefalconjun/bluefalconjun.github.io)*_  
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

***

**> on Windows (client)**  
**[shadowsocks-csharp @ _github_](https://github.com/shadowsocks/shadowsocks-csharp)**  
***

**> on Android (client)**  
**[Shadowsocks @ _google playstore_](https://play.google.com/store/apps/details?id=com.github.shadowsocks)**  
***  
_*[Back Home](https://github.com/bluefalconjun/bluefalconjun.github.io)*_  



