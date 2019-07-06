# ArchiSteamFarm



##  1.docker install

```shell
yum install docker

systemctl start docker 

systemctl enable docker

docker pull justarchi/archisteamfarm

docker run --name asf -d --net host \
-v /usr/nine/asf/config:/app/config \
docker.io/justarchi/archisteamfarm
```

## 2.config file

```
/usr/nine/asf/config  //this diretory is custom.it require IPC.config and ASF.json
```

IPC.config

```json
{     
	"Kestrel": {         
		"Endpoints": {             
			"IPv4-http": {                 
				"Url": "http://0.0.0.0:80"             
			}         
		},         
	"PathBase": "/"    
	} 
}
```

ASF.json

```json
{
	"Blacklist": [
		267420,
		303700,
		335590,
		368020,
		425280,
		480730
	],
    "Headless": true,
    "IdleFarmingPeriod": 3,
    "IPC": true,
    "IPCPassword": "1234",
    "LoginLimiterDelay": 15,
    "SteamOwnerID": your steam profile id. like  7656....
}
```



