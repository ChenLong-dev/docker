# Portainer
##  官网
```
https://www.portainer.io/
https://blog.csdn.net/shark_chili3007/article/details/123366179
```
## portainer 的 docker 启动命令
```
docker run -itd  --name portainer -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v /data/docker/portainer/data:/data --restart always --privileged=true portainer/portainer:latest
```

## docker-compose 启动命令
```
docker-compose -f portainer.yml up -d
```

## web界面（admin账号密码）
```
admin/520yu***
```