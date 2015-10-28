_*[Back Home](https://github.com/bluefalconjun/bluefalconjun.github.io)*_  
***  


_**bashrc:**_  

    force_color_prompt=yes
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u\[\033[00m\]:\[\033[01;34m\]\W\[\033[00m\]\$ '    
    export USE_CCACHE=1

    # Aliases
    alias ls='ls -h --color=auto'
    alias ll='ls -l'
    alias la='ls -A'
    alias l='ls -CF'
    alias svim='sudo vim'
    alias h='cd'
    alias ..='cd ..'
    alias cd..='cd ..'
    alias ...='cd ../..'
    alias vi='vim'
    alias back='cd $OLDPWD'
    alias root='sudo su'
    alias runlevel='sudo /sbin/init'
    alias grep='grep --color=auto'
    alias dfh='df -h'

    export PATH=$PATH:/usr/local/bin
    export EDITOR=vim

    #golang related
    export GOROOT=/home/junxu/workspace/Backup/golang/go
    export GOPATH=/home/junxu/workspace/Backup/golang/workspace
    export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

    # for repo use proxy
    #export HTTP_PROXY=http://127.0.0.1:8118
    #export HTTPS_PROXY=http://127.0.0.1:8118



***  
_*[Back Home](https://github.com/bluefalconjun/bluefalconjun.github.io)*_  


