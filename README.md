<!--
 * @Author: ChenLong longchen2008@126.com
 * @Date: 2024-05-16 15:01:17
 * @LastEditors: ChenLong longchen2008@126.com
 * @LastEditTime: 2024-05-16 15:01:51
 * @FilePath: \docker\README.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
# docker 相关命令

### 使用 docker rmi 命令删除
```
docker rmi $(docker images -a | grep "<none>" | awk '$1=="<none>" {print $3}')
```
### 查看是否有无效的 none 镜像
```
docker images -f "dangling=true"
```

### 如果有的话就执行以下删除命令
```
docker rmi $(docker images -f "dangling=true" -q) -f
```

### 构建镜像过程中强制删除中间镜像
可以通过再构建镜像的命令中添加 --force-rm 参数来删除镜像构建过程中生成的中间镜像：
```
docker build --force-rm -t myimage .
```

### 构建镜像
```
docker build --force-rm -t jobs:v1 -f Dockerfile.timerjobs .
```

### 运行容器
```
docker run -itd --name my-jobs jobs:v1
```

### 进入容器
```
docker exec -it my-jobs sh
```

### 带host构建镜像
```
docker build --add-host mirrors.sangfor.org:200.200.1.241 -t jobs:v2 -f Dockerfile.timerjobs .

```

dockerfile文件中使用
```
ENV GOPROXY http://mirrors.sangfor.org/nexus/repository/go-proxy
```

### 带host运行容器
```
docker run -itd --add-host mirrors.sangfor.org:200.200.1.241 golang:1.18.7-alpine3.16
```
容器中的/etc/hosts文件内容
```
$ docker exec -it 5c291b9c2b4c sh
/go # cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
200.200.1.241   mirrors.sangfor.org
172.17.0.2      5c291b9c2b4c
/go #
```

# docker镜像
## docker 镜像的导出和导入
### 导出镜像
一旦找到要导出的镜像，我们可以使用docker save命令将其导出到本地系统。以下是导出镜像的示例命令：
```
docker save -o <导出路径/文件名.tar> <镜像名称>
```
- -o参数指定导出路径和文件名。
- <镜像名称>是要导出的镜像的名称或ID。

例如，要将名为myimage的镜像导出到当前目录下的myimage.tar文件中，可以使用以下命令：
```
docker save -o myimage.tar myimage
```
### 导入镜像（验证导出的镜像）
导出完成后，可以使用docker load命令验证导出的镜像。以下是载入镜像的示例命令：
```
docker load -i <导出路径/文件名.tar>
```
- -i参数指定导入的镜像文件。

例如，要验证刚刚导出的myimage.tar文件，可以使用以下命令：
```
docker load -i ./myimage.tar
```

- 成功导入后，将显示类似以下信息：
```
Loaded image: myimage:latest
```

