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
$ mkdir mybot
$ cd mybot
```

### Generate bot
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
_Note:_ `Bot adapter: slack`

### Remove unuse script

* Open file `external-scripts.json` and delete row about `heroku`, `redis`
* Remove file `hubot-scripts.json`

### Run it
```sh
$ ./bin/hubot
```

### Test
```sh
mybot> mybot help
mybot> mybot ping
```

Press `ctrl+c` to quit.

### Integration for slack
* Access `https://your-team.slack.com/apps/search?q=hubot` and add hubot app
* Copy token and save setting

### Make startup script
```sh
$ vi mybot.sh
```

```sh
#!/bin/bash
export HUBOT_SLACK_TOKEN=xoxb-Your-token
./bin/hubot --adapter slack &
```

* Run `mybot.sh`
* Invite `mybot` from your slack channel `#general`
* Typing `mybot ping` in your slack channel `#general`
* May be `mybot` response `PONG`.

### Let hubot execute shell script

* install hubot-script-shellcmd

```sh
npm install hubot-script-shellcmd
cp -R node_modules/hubot-script-shellcmd/bash ./
```

* Open file `external-scripts.json` and add row `hubot-script-shellcmd`

* Customize your shell in directory `bash/handlers`, `helloworld` and `update` is sample.

* Run `mybot.sh` and try it.

```sh
$ sh mybot.sh
mybot> mybot shellcmd update
```

### Additional

* Alias from `shellcmd` to `run`
  * open file `mybot.sh` and add row `export HUBOT_SHELLCMD_KEYWORD=run`
  * restart
* keep online
  * use module `forever`
