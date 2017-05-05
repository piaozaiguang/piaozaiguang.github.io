_适用环境：CentOS_

### Add Node.js Yum Repository
```sh
# yum install -y gcc-c++ make
# curl -sL https://rpm.nodesource.com/setup_6.x | sudo -E bash -
```

### Install Node.js and NPM
```sh
# yum install nodejs
```

### Install yo, hubot, coffee and generator
```sh
# npm install -g yo hubot coffee-script generator-hubot
```

### Make directory
```sh
$ cd ~
$ mkdir hogebot
$ cd hogebot
```

### generate bot
```sh
$ yo hubot
? ==========================================================================
We're constantly looking for ways to make yo better! 
May we anonymously report usage statistics to improve the tool over time? 
More info: https://github.com/yeoman/insight & http://yeoman.io
========================================================================== Yes
                     _____________________________  
                    /                             \ 
   //\              |      Extracting input for    |
  ////\    _____    |   self-replication process   |
 //////\  /_____\   \                             / 
 ======= |[^_/\_]|   /----------------------------  
  |   | _|___@@__|__                                
  +===+/  ///     \_\                               
   | |_\ /// HUBOT/\\                             
   |___/\//      /  \\                            
         \      /   +---+                            
          \____/    |   |                            
           | //|    +===+                            
            \//      |xx|                            

? Owner yourname
? Bot name mybot
? Description mybot
? Bot adapter slack
   create bin/hubot
   create bin/hubot.cmd
   create Procfile
   create README.md
   create external-scripts.json
   create hubot-scripts.json
   create .gitignore
   create package.json
   create scripts/example.coffee
   create .editorconfig
                     _____________________________  
 _____              /                             \ 
 \    \             |   Self-replication process   |
 |    |    _____    |          complete...         |
 |__\\|   /_____\   \     Good luck with that.    / 
   |//+  |[^_/\_]|   /----------------------------  
  |   | _|___@@__|__                                
  +===+/  ///     \_\                               
   | |_\ /// HUBOT/\\                             
   |___/\//      /  \\                            
         \      /   +---+                            
          \____/    |   |                            
           | //|    +===+                            
            \//      |xx|                            

```
