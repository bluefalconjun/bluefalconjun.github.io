_*[Back Home](https://bluefalconjun.github.io)*_  
***


mactype will render Fonts in windows like MAC !!!

1.download MacTray 20131231 Candy Win8.1use.7z  
2.install MacTray  
3.setup as your wish  

4.change chrome to enable mactype: Update with chrome50
    chrome://flags/#disable-direct-write 启用
    chrome://flags/#num-raster-threads 改为 1
    chrome://flags/#ignore-gpu-blacklist 启用 (Chrome50已改为默认启用)
    chrome://flags/#enable-zero-copy 停用
    chrome启动参数添加 --disable-directwrite-for-ui

5.change atom to enable mactype:  
    a.open $atom_install/app-$atom-version/resources/app.asar  
    b.change 'direct-write': true, to 'direct-write':false, remember the length of whole string should be same.  

***
_*[Back Home](https://bluefalconjun.github.io)*_  
