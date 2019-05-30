# Portainer


。
```dockerfile
docker run -d -p 25565:9000 --restart=always \
--name portainer \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/nine/portainer/data:/data docker.io/portainer/portainer
```

