Atom

Set Proxy on Atom:
Edit ~/.atom/.apmrc

    https-proxy=http://[user]:[pwd]@[host]:[port]
    http-proxy=http://[user]:[pwd]@[host]:[port]
    strict-ssl=false


Change for bookmakr keymap in file: keymap.cson

    #add for bookmark
    '.platform-win32 atom-text-editor':
      'ctrl-f2': 'bookmarks:toggle-bookmark'
    'atom-text-editor':
      'ctrl-alt-f2': 'bookmarks:view-all'
      
Useful Packages:

    convert-to-utf8
    editorconfig
    file-icons
    go-plus
    highlight-selected
    project-manager
    
