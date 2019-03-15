# Steamcmd



## 首先安装 steamcmd  



### 一般安装

```
Ubuntu/Debian 
sudo apt-get install steamcmd
RedHat/CentOS 
yum install steamcmd
Or
wget http://media.steampowered.com/installer/steamcmd_linux.tar.gz
tar -zxvf steamcmd_linux.tar.gz
```

### Docker 式安装 

```
docker pull cm2network/steamcmd 
docker run -it --name=steamcmd cm2network/steamcmd bash
```

### 创建快捷方式

```
ln -s /usr/games/steamcmd steamcmd
```



# 安装运行库

 

```
Ubuntu/Debian 64-Bit 
sudo apt-get install lib32gcc1
RedHat/CentOS 
yum install glibc libstdc++
RedHat/CentOS 64-Bit 
yum install glibc.i686 libstdc++.i686
```



# 开始使用



### 启动

```
./steamcmd.sh
```

### 匿名登录

```
login anonymous
```

### 设置安装路径

```
force_install_dir <path>    
例子：  force_install_dir ./cs_go/
```

### 安装应用的文件：

```
app_update <app_id> [-beta <betaname>] [-betapassword <password>] [validate]
例子：   
app_update 740 validate
app_update 90 -beta beta validate
```

### 退出

```
quit
```



# 进阶



### 编写更新脚本（以csgo为例子





#### 命令行式：

```
steamcmd +login anonymous +force_install_dir ../csgo_ds +app_update 740 +quit
 
```

#### 脚本式：

```
vim update_csgo_ds.txt
 
@ShutdownOnFailedCommand 1 //set to 0 if updating multiple servers at once @NoPromptForPassword 1 login anonymous  force_install_dir ../csgo_ds app_update 740 validate quit
 
./steamcmd +runscript csgo_ds.txt
```



[各种服务端的appid]: https://developer.valvesoftware.com/wiki/Dedicated_Servers_List	"."

