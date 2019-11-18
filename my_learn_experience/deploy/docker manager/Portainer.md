# Portainer

## install

```dockerfile
docker run -d -p 9000:9000 --restart=always \
--name portainer \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/nine/portainer/data:/data docker.io/portainer/portainer
```

## config file

```markdown
/usr/nine/portainer/data  //this directory is custom.
```