mactype will render Fonts in windows like MAC !!!

1.download MacTray 20131231 Candy Win8.1use.7z  
2.install MacTray  
3.setup as your wish  

4.change chrome to enable mactype:  
    a.open chrome://flags  
    b.set Disable DirectWrite Windows  
      Disables the use of experimental DirectWrite font rendering system. #disable-direct-write  
      Disable  

5.change atom to enable mactype:  
    a.open $atom_install/app-$atom-version/resources/app.asar  
    b.change 'direct-write': true, to 'direct-write':false, remember the length of whole string should be same.  
