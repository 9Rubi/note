# **Don't Starve Together**

需要先查阅Steamcmd 安装

见笔记 Steamcmd

使用Steamcmd 安装 所需的服务端文件

 

```
./steamcmd.sh
login anonymous
force_install_dir ./dst/
app_update 343050 validate
 
```

## 安装运行库

```
Ubuntu/Debian 64-Bit sudo apt-get install lib32gcc1 screen RedHat/CentOS 32-Bit yum -y install glibc libstdc++ screen libcurl RedHat/CentOS 64-Bit yum -y install glibc.i686 libstdc++.i686 screen libcurl.i686
见 https://dontstarve.fandom.com/wiki/Guides/Don%E2%80%99t_Starve_Together_Dedicated_Servers
```

## 运行 Don’t Strave Together Dedicated Server

可能会因为 libstdc++.so.6 库版本过低，出现无法运行

```
yum install libcurl.i686
ln -s libcurl.so.4 libcurl-gnutls.so.4
 
```



## 配置Don’t Starve Together



### 生成默认配置文件

```
cd ~/DST/bin
./dontstarve_dedicated_server_nullrenderer
 
```

当看到以下提示

```
[200] Account Failed (6): "E_INVALID_TOKEN"
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!! Your Server Will Not Start !!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
 
```

按Ctrl+C中断，然后完善生成的默认配置

### 目录结构

```
Cluster_1
├── cluster.ini
├── cluster_token.txt
├── Caves
│   ├── modoverrides.lua
│   ├── server.ini
│   └── worldgenoverride.lua
└── Master
├── modoverrides.lua
├── server.ini
└── worldgenoverride.lua      #重写世界具体物品数量等参数配置
```

#### cluster_token.txt

```
可用指令，也可以在klei官网生成
```

#### cluster.ini

```
[GAMEPLAY]
game_mode = survival                  ;Endless无尽模式，Wildern荒野模式，Survival生存模式
max_players = 6                       ;最大玩家数，1-64任意一个数
pvp = false                                 
pause_when_empty = true               ;世界没人时是否自动暂停
enable_snapshots = true
enable_autosaver = true
 
[NETWORK]
cluster_description = Live it         ;游戏房间描述
cluster_name = LiveIt007              ;游戏名称
cluster_intention = cooperative       ;游戏模式 
cluster_password = 123123123          ;密码
 
[MISC]
console_enabled = true                ;控制台
 
[SHARD]
shard_enabled = true                  ;地下世界是否启动
bind_ip = 127.0.0.1                   ;固定IP
master_ip = 127.0.0.1                 ;地上世界IP
master_port = 10889                   ;地上世界端口
cluster_key = supersecretkey          ;地下世界连接地上世界的钥匙
 
```

#### Master/server.ini

```
[NETWORK]
server_port = 11000
 
[SHARD]
is_master = true
 
[STEAM]
master_server_port = 27018
authentication_port = 8768
 
```

#### Caves/server.ini

```
[NETWORK]
server_port = 11001
 
[SHARD]
is_master = false
name = Caves
 
[STEAM]
master_server_port = 27019
authentication_port = 8769
```

#### Caves/worldgenoverride.lua

```
return {
override_enabled = true,
preset = "DST_CAVE",
}
 
```

#### Start.sh

```
#!/bin/bash
 
cluster_name="cluster_1"
conf_dir="DoNotStarveTogether"
persistent_storage_root="/usr/rubi/steam/dst/persistent"
 
run_shared=(./dontstarve_dedicated_server_nullrenderer)
run_shared+=(-console)
run_shared+=(-persistent_storage_root "$persistent_storage_root")
run_shared+=(-conf_dir "$conf_dir")
run_shared+=(-cluster "$cluster_name")
run_shared+=(-monitor_parent_process $$)
run_shared+=(-shard)
 
"${run_shared[@]}" Caves  | sed 's/^/Caves:  /'&
"${run_shared[@]}" Master | sed 's/^/Master: /'
 
```

实际是在

```
"$persistent_storage_root"/"$conf_dir"/"$cluster_name"
/usr/rubi/steam/dst/persistent/DoNotStarveTogether/cluster_1
 
```



## 增加Mod



### dedicated_server_mods_setup.lua

饥荒通过DST/mods路径下的dedicated_server_mods_setup.lua文件确认需要下载那些mod。

首先去创意工坊找些 Mod，并获得其 id，或者找些 Mod 合集，将 Mod id 按以下形式（换行复制粘贴）保存在文件中。以下是完整文件内容

```
 
--There are two functions that will install mods, ServerModSetup and ServerModCollectionSetup. Put the calls to the functions in this file and they will be executed on boot.
--ServerModSetup takes a string of a specific mod's Workshop id. It will download and install the mod to your mod directory on boot.
--The Workshop id can be found at the end of the url to the mod's Workshop page.
--Example: http://steamcommunity.com/sharedfiles/filedetails/?id=350811795
--ServerModSetup("350811795")
--ServerModCollectionSetup takes a string of a specific mod's Workshop id. It will download all the mods in the collection and install them to the mod directory on boot.
--The Workshop id can be found at the end of the url to the collection's Workshop page.
--Example: http://steamcommunity.com/sharedfiles/filedetails/?id=379114180
--ServerModCollectionSetup("379114180")
ServerModSetup("358749986")                --Extended Indicators WIP 队友靠近会显示小图标
ServerModSetup("362175979")                --Wormhole Marks [DST]  彩色虫洞
ServerModSetup("378160973")                --Global Positions 共享队友位置
ServerModSetup("438293817")                --Friendly Flingomatics 友好的灭火器
ServerModSetup("666155465")                --Show Me
ServerModSetup("770901818")                --Hound Attack   舔狗预警
ServerModSetup("782961570")                --PartyHUD - Team Health Display
ServerModSetup("462434129")                --restart
ServerModSetup("1207269058")                      --简易血条DST
ServerModSetup("1621458706")                      --皮肤法杖
--ServerModCollectionSetup("id")
 
```

上面是常用的几个Mod。但是dedicated_server_mods_setup.lua只是用于下载Mod，至于Mod是否启用以及配置则是modoverrides.lua 文件的功能。

### modoverrides.lua

```
return {
--Extended Indicators WIP 队友靠近会显示小图标
["workshop-358749986"]={
    configuration_options={ IndicatorSize=3, MaxIndicator=7000, PlayerIndicators=1 },
    enabled=true 
    },
--Wormhole Marks [DST]  彩色虫洞
["workshop-362175979"]={ configuration_options={ ["Draw over FoW"]="disabled" }, enabled=true },
--Global Positions 共享队友位置
["workshop-378160973"]={
    configuration_options={
      ENABLEPINGS=true,
      FIREOPTIONS=2,
      OVERRIDEMODE=false,
      SHAREMINIMAPPROGRESS=true,
      SHOWFIREICONS=true,
      SHOWPLAYERICONS=true,
      SHOWPLAYERSOPTIONS=2 
    },
    enabled=true 
    },
--Friendly Flingomatics 友好的灭火器
["workshop-438293817"]={ configuration_options={  }, enabled=true },
--Show Me
["workshop-666155465"]={
    configuration_options={
      food_estimation=-1,
      food_order=0,
      food_style=0,
      lang="auto",
      show_food_units=-1 
    },
    enabled=true 
  },    
--Hound Attack   舔狗预警
["workshop-770901818"]={
    configuration_options={ days=2, enable_houndattack=true, format="complex" },
    enabled=true 
},
--PartyHUD - Team Health Display
["workshop-782961570"]={ configuration_options={ layout=1, position=0 }, enabled=false },
--restart
["workshop-462434129"]={
    configuration_options={
      MOD_RESTART_ALLOW_KILL=true,
      MOD_RESTART_ALLOW_RESTART=true,
      MOD_RESTART_ALLOW_RESURRECT=true,
      MOD_RESTART_CD_BONUS=0,
      MOD_RESTART_CD_KILL=3,
      MOD_RESTART_CD_MAX=0,
      MOD_RESTART_CD_RESTART=5,
      MOD_RESTART_CD_RESURRECT=7,
      MOD_RESTART_FORCE_DROP_MODE=0,
      MOD_RESTART_IGNORING_ADMIN=true,
      MOD_RESTART_MAP_SAVE=2,
      MOD_RESTART_RESURRECT_HEALTH=0,
      MOD_RESTART_TRIGGER_MODE=1,
      MOD_RESTART_WELCOME_TIPS=true,
      MOD_RESTART_WELCOME_TIPS_TIME=6 
    },
    enabled=true 
},
--简易血条DST
["workshop-1207269058"]={ configuration_options={  }, enabled=true },
--皮肤法杖
["workshop-1621458706"]={ configuration_options={  }, enabled=true }
}
 
要注意，这两个文件的Mod Id是一一对应的。同时，将modoverrides.lua分别复制到Master和Caves文件下。
 
```



## 设定管理员



### Cluster_1/adminlist.txt

```
KU_*****
```



## 用户黑名单，用户白名单



### Cluster_1/blocklist.txt

```
KU_*******
```

### Cluster_1/whitelist.txt

```
KU_*******
```



## 下面是最终的文件结构



```
Cluster_1
├── cluster.ini
├── cluster_token.txt
├── adminlist.txt
├── blocklist.txt
├── whitelist.txt
├── Caves
│   ├── modoverrides.lua
│   ├── server.ini
│   └── worldgenoverride.lua
└── Master
├── modoverrides.lua
├── server.ini
└── worldgenoverride.lua      #重写世界具体物品数量等参数配置
```

 

 