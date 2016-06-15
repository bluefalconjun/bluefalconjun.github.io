_*[Back Home](https://bluefalconjun.github.io)*_  
***

*git config*

~/.gitconfig

    [user]
        name = Xu Jun
        email = jun.xu.falcon@gmail.com
    [core]
        editor = vim
    [commit]
        template = /$home/.gitcm.template
    [help]
        autocorrect = 1
    [color]
        ui = true
    [color "diff"]
        meta = blue black bold
    [alias]
        st = status
        lg = log
        br = branch
        cm = commit

~/.gitcm.template

    [commit from xxx host]


remove git certification from mac

    $git credential-osxkeychain erase
    host=github.com
    protocol=https
    [Press Return]

*** 
_*[Back Home](https://bluefalconjun.github.io)*_  
