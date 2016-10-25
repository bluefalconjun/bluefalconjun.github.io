_*[Back Home](https://bluefalconjun.github.io)*_  
***

*git config*

~/.gitconfig

    [user]
        name = Xu Jun
        email = jun.xu.falcon@gmail.com
        
    [core]
        editor = vim
        autocrlf = false
        safecrlf = true
        
    [commit]
        template = /$home/.gitcm.template
        
    [help]
        autocorrect = true
        
    [color]
        ui = auto
        
    [color "diff"]
        meta = blue black bold
        
    [alias]
        st = status
        lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
        br = branch
        cm = commit
    
    [credential]
        helper = cache --timeout=3600000 //this is for github to record your username/pwd        

    [url "ssh://$username@$ipaddr:29418"]
        insteadof = ssh://$ipaddr //this is for quick replace some cmd line.
        


~/.gitcm.template

    [commit from xxx host]


remove git certification from mac

    $git credential-osxkeychain erase
    host=github.com
    protocol=https
    [Press Return]

*** 
_*[Back Home](https://bluefalconjun.github.io)*_  
