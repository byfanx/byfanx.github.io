---
title: Docker部署应用
route: docker-deployApp
date: 2020-04-20 10:41:53
tags: [服务器,Docker]
categories: 服务器
image: /images/cover/docker-deployApp.png
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之前写过一篇关于[Docker初使用](/2019/11/11/docker-initUse/)的笔记，简单介绍了Docker和基本使用。这里记录一下怎么将自己的项目打包成Docker镜像然后部署在服务器的Docker里。

<!-- more -->

# Docker部署应用

## 1. 制作镜像

### 1.1 准备项目

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打包镜像首先需要将自己的项目打包成jar包，我这里准备的是一个SpringBoot项目打包成的jar包。

### 1.2 编写Dockerfile

#### 1.2.1 关于Dockerfile

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Dockerfile是一个包含用于组合映像的命令的文本文档。可以使用在命令行中调用任何命令。 Docker通过读取`Dockerfile`中的指令自动生成映像。一般包含四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。Docker以从上到下的顺序运行Dockerfile的指令。为了指定基本映像，第一条指令必须是***FROM***。一个声明以`＃`字符开头则被视为注释。可以在Docker文件中使用`RUN`，`CMD`，`FROM`，`EXPOSE`，`ENV`等指令。

#### 1.2.2 演示文件

这次演示的Dockerfile文件内容如下：

```dockerfile
# This my first testapp Dockerfile
# Version 1.0

# Base images 基础镜像
FROM java:8
# 挂载目录
VOLUME /tmp
# 文件放在当前目录下，拷过去会自动解压
ADD testapp-0.0.1-SNAPSHOT.jar /app.jar
# 配置容器，使其可执行化。配合CMD可省去"application"，只使用参数。
ENTRYPOINT ["java","-Djava.security.edg=file:/dev/./urandom","-jar","/app.jar"]
```

> **#** ： 为 Dockerfile 中的注释。

#### 1.2.3 Dockerfile详解

+ **FROM**：指定基础镜像，需要在哪个镜像建立。必须为第一个命令。

```dockerfile
# 格式
　　FROM <image>
　　FROM <image>:<tag>
　　FROM <image>@<digest>
# 示例
   FROM mysql：5.6
# 提示
   tag或digest是可选的，如果不使用这两个值时，会使用latest版本的基础镜像
```

+ **MAINTAINER**：指定维护者信息。

```dockerfile
# 格式
   MAINTAINER <name>
# 示例
   MAINTAINER byFan
   MAINTAINER byfanx@163.com
```

+ **RUN**：构建镜像时执行的命令。

```dockerfile
RUN用于在镜像容器中执行命令，其有以下两种命令执行方式：
shell执行
# 格式
   RUN <command>
# 示例
   RUN apk update
   
exec执行
# 格式
   RUN ["executable", "param1", "param2"]
# 示例
   RUN ["/bin/bash","-c","echo hello"]
# 提示
   RUN指令创建的中间镜像会被缓存，并会在下次构建中使用。如果不想使用这些缓存镜像，可以在构建时指定--no-cache参数，如：docker build --no-cache
```

+ **WORKDIR**：指定当前工作目录，相当于 `cd`。

```dockerfile
# 格式
   WORKDIR /path/to/workdir
# 示例
   WORKDIR /a  (这时工作目录为/a)
   WORKDIR b  (这时工作目录为/a/b)
   WORKDIR c  (这时工作目录为/a/b/c)
# 提示
   通过WORKDIR设置工作目录后，Dockerfile中其后的命令RUN、CMD、ENTRYPOINT、ADD、COPY等命令都会在该目录下执行。在使用docker run运行容器时，可以通过-w参数覆盖构建时所设置的工作目录。
```

+ **EXPOSE**：指定容器要打开的端口。

```dockerfile
# 格式
   EXPOSE <port> [<port>...]
# 示例
   EXPOSE 80 443
   EXPOSE 11211/tcp
# 提示
   EXPOSE并不会让容器的端口访问到主机。要使其可访问，需要在docker run运行容器时通过-p来发布这些端口，或通过-P参数来发布EXPOSE导出的所有端口
```

+ **ENV**：定义环境变量

```dockerfile
# 格式
   ENV <key> <value>    # 会被后续 RUN 指令使用，并在容器运行时保持。<key>之后的所有内容均会被视为其<value>的组成部分，因此，一次只能设置一个变量
   ENV <key>=<value> ...   # 可以设置多个变量，每个变量为一个"<key>=<value>"的键值对，如果<key>中包含空格，可以使用\来进行转义，也可以通过""来进行标示；另外，反斜线也可以用于续行
# 示例
   ENV myName byFan
   ENV mySex Man
```

+ **ADD**：将本地文件添加到容器中，tar类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似wget。

```dockerfile
# 格式
   ADD <src> <dest>
   ADD ["<src>","<dest>"]  # 用于支持包含空格的路径
# 示例
   ADD hom* /mydir/         # 添加所有以"hom"开头的文件
    ADD hom?.txt /mydir/    # ? 替代一个单字符,例如："home.txt"
    ADD test relativeDir/   # 添加 "test" 到 `WORKDIR`/relativeDir/
    ADD test /absoluteDir/  # 添加 "test" 到 /absoluteDir/
```

+ **COPY**：复制本地主机的 （为 Dockerfile 所在目录的相对路径）到容器中的。功能类似ADD，但是是不会自动解压文件，也不能访问网络资源。

```dockerfile
# 格式
   COPY <src> <dest>
   COPY ["<src>","<dest>"]  # 用于支持包含空格的路径
# 示例
   COPY test.txt /tmp
```

+ **VOLUME**：挂载目录，创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

```dockerfile
# 格式
   VOLUME ["/path/to/dir"]
# 示例
   VOLUME ["/data"]
   VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"
# 提示
   一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：
   (1) 卷可以容器间共享和重用
   (2) 容器并不一定要和其它容器共享卷
   (3) 修改卷后会立即生效
   (4) 对卷的修改不会对镜像产生影响
   (5) 卷会一直存在，直到没有任何容器在使用它
```

+ **USER**：指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。使用USER指定用户时，可以使用用户名、UID或GID，或是两者的组合。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户。

```dockerfile
# 格式
   USER user
　　USER user:group
　　USER uid
　　USER uid:gid
　　USER user:gid
　　USER uid:group
# 示例
   USER byfan
# 提示
   使用USER指定用户后，Dockerfile中其后的命令RUN、CMD、ENTRYPOINT都将使用该用户。镜像构建完成后，通过docker run运行容器时，可以通过-u参数来覆盖所指定的用户。
```

+ **ENTRYPOINT**：配置容器，使其可执行化。配合CMD可省去"application"，只使用参数。

```dockerfile
# 格式
   ENTRYPOINT ["executable", "param1", "param2"]   # (可执行文件, 优先)
    ENTRYPOINT command param1 param2   # (shell内部命令)
# 示例
   FROM ubuntu
   ENTRYPOINT ["top","-b"]
   CMD ["-c"]
# 提示
   ENTRYPOINT与CMD非常类似，不同的是通过docker run执行的命令不会覆盖ENTRYPOINT，而docker run命令中指定的任何参数，都会被当做参数再次传递给ENTRYPOINT。Dockerfile中只允许有一个ENTRYPOINT命令，多指定时会覆盖前面的设置，而只执行最后的ENTRYPOINT指令。
```

+ **CMD**：构建容器后调用，也就是在容器启动时才进行调用。

```dockerfile
# 格式
   CMD ["executable","param1","param2"]   # (执行可执行文件，优先)
    CMD ["param1","param2"]   # (设置了ENTRYPOINT，则直接调用ENTRYPOINT添加参数)
    CMD command param1 param2   # (执行shell内部命令)
# 示例
   CMD echo "This is a test." | wc -
   CMD ["/usr/bin/wc","--help"]
# 提示
   CMD不同于RUN，CMD用于指定在容器启动时所要执行的命令，而RUN用于指定镜像构建时所要执行的命令。每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。
```

> ENTRYPOINT 和 CMD 的区别：ENTRYPOINT 指定了该镜像启动时的入口，CMD 则指定了容器启动时的命令，当两者共用时，完整的启动命令像是 ENTRYPOINT + CMD 这样。使用 ENTRYPOINT 的好处是在我们启动镜像就像是启动了一个可执行程序，在 CMD 上仅需要指定参数；另外在我们需要自定义 CMD 时不容易出错。
>
> 可以使用以下命令覆盖默认的参数，方便调试 Dockerfile 中的 bug：
>
> `docker run -it --entrypoint=/bin/dockerfile feiyu/entrypoint:1`

+ **ONBUILD**：用于设置镜像触发器。

```dockerfile
# 格式
   ONBUILD [INSTRUCTION]
# 示例
   ONBUILD ADD . /app/src
　　ONBUILD RUN /usr/local/bin/python-build --dir /app/src
# 提示
   在构建本镜像时不生效，在基于此镜像构建镜像时生效。
```

+ **LABLE**：用于为镜像添加元数据。

```dockerfile
# 格式
   LABEL <key>=<value> <key>=<value> <key>=<value> ...
# 示例
   LABEL version="1.0" description="这是一个Web服务器" by="IT笔录"
# 提示
   使用LABEL指定元数据时，一条LABEL指定可以指定一或多条元数据，指定多条元数据时不同元数据之间通过空格分隔。推荐将所有的元数据通过一条LABEL指令指定，以免生成过多的中间镜像。
```

+ **ARG**：用于指定传递给构建运行时的变量。

```dockerfile
# 格式
   ARG <name>[=<default value>]
# 示例
   ARG site
   ARG build_user=www
```

### 1.3 创建镜像

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将准备好的项目jar包和Dockerfile文件上传到你的Linux服务器的同一目录下。然后执行下面命令来创建一个镜像。

```shell
docker build -t testapp:1.0 .
```

> **-t**：指定镜像名字和TAG。
>
> **注意**：这里最后面有一个 ’   **.**   ‘，代表的是Dockerfile文件所在的路径(当前目录)，也可以替换为一个具体的Dockerfile的路径。

然后就可以看到下面的执行过程。

![](image-20200421115234434.png)

接下来通过`docker images`命令进行查看，就可以看到我们刚刚创建的镜像。

![](image-20200421115653201.png)

### 1.4 运行项目

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;运行项目就是通过刚刚我们制作的镜像来创建一个容器，然后进行运行，运行成功返回容器的id。

```shell
docker run -d -p 8080:8080  --name testApp testapp:1.0
```

> **-d：**标识是让docker容器在后台运行。
>
> **-p：**标识端口映射，前面是主机对外服务端口，后面是映射到docker容器的端口。
>
> **–name：**定义一个容器的名字，方便后面执行操作。如果没有指定name，那么deamon会自动生成一个随机数字符串当作UUID。
>
> 最后一个是你创建容器使用的镜像名称和TAG。

然后就可以运行`docker ps -a`命令进行查看运行的容器了。

## 2. 部署Nginx

### 2.1 拉取Nginx镜像

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;拉取当前最新的Nginx的镜像。

```shell
docker pull nginx
```

### 2.2 运行Nginx

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据刚刚拉取下来的nginx镜像，运行一个容器。

```shell
docker run -d -p 80:80 --name nginxV1 nginx
```

这样nginx服务器就运行完成了，就可以进行访问了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为要对容器内部的配置文件进行编辑，而容器内部是没有`vim`编辑器的，可以下载一个，但是每次配置都学要进入到容器内部操作。显得太麻烦。也可以使用目录挂载的形式。首先创建本地目录，用于存放Nginx容器的相关文件信息。

```shell
mkdir -p /home/nginx/html /home/nginx/logs /home/nginx/conf
```

> **-p**：确保目录名称存在，不存在的就建一个。

其中：

+ html：目录将映射为nginx容器配置的虚拟目录。
+ logs：目录将映射为nginx容器目录的日志内容。
+ conf：目录里的配置文件将映射为nginx容器的配置文件。

将nginxV1容器内nginx默认的配置文件拷贝到当前目录下的conf目录。也可以通过容器id进行复制，容器id可以通过`docker ps -a`命令进行查看。

```shell
docker cp nginxV1:/etc/nginx/nginx.conf /home/nginx/conf/
```

然后进行部署:

```shell
docker run -d -p 80:80 --name nginxV2 -v /home/nginx/html:/usr/share/nginx/html -v /home/nginx/logs:/var/log/nginx -v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf nginx
```

> **-v /home/nginx/www:/usr/share/nginx/html**：将我们自己创建的 www 目录挂载到容器的 /usr/share/nginx/html。
>
> **-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf**：将我们自己创建的 nginx.conf 挂载到容器的 /etc/nginx/nginx.conf。
>
> **-v /home/nginx/logs:/var/log/nginx**：将我们自己创建的 logs 挂载到容器的 /var/log/nginx。

启动成功之后，就可以在本地配置我们的nginx.conf了，配置好之后重新启动容器即可生效。

```shell
docker restart nginx
```

在nginx原来的配置文件中，它的`server`块没有直接写在nginx.conf里，而是通过`include /etc/nginx/conf.d/*.conf;`引入的。在容器的`/etc/nginx/conf.d/`目录下，有一个`default.conf`文件，进入容器

```shell
docker exec -it nginxV2 /bin/bash
```

> 上述操作也可以将容器名替换成容器id，退出容器时执行**exit**就行。

查看`default.conf`文件内容如下：

```json
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

可以看到这里就是一个`server`块。如果不使用想外部引入，那么就可以把上面nginx.conf文件中的`include`那一句给去掉，然后直接在nginx.conf里进行`server`块的配置。每次重新配置完nginx.conf文件之后，都需要重启容器。

### 2.4 部署SSL证书

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先获取SSL证书，[阿里云免费证书](https://www.aliyun.com/product/cas?spm=5176.12825654.eofdhaal5.19.3dbd2c4ayD5k2Q&aly_as=nsmL9Pxx)是*Symantec*颁发的，下载的压缩包里面有两个文件，一个是`.pem`文件，一个是`.key`文件。[腾讯云的免费证书](https://console.cloud.tencent.com/ssl/apply)是*TrustAsia*颁发的，下载的压缩包里面有两个文件，一个是`.crt`文件，一个是`.key`文件。二者只是证书颁发商不同，实际都是可以满足需求的。这里选用腾讯云申请的证书。因为我们之前启动容器时只映射了80端口，而`https`默认的时443端口，所以我们现在重新启动一个容器。

```shell
docker run -d -p 80:80 -p 443:443 --name nginxV3 -v /home/nginx/html:/usr/share/nginx/html -v /home/nginx/logs:/var/log/nginx -v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf nginx
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先将下载到的两个证书文件上传到Linux服务器，然后在nginx配置文件目录下创建`cert`目录，将证书文件放到该目录下。然后将存放证书文件的目录复制到容器中存放nginx配置文件的目录下。

```shell
docker cp /home/nginx/conf/cert nginxV3:/etc/nginx/
```

然后修改配置文件。

```json
server {

        listen 443 ssl;  #新版本启用SSL功能。
    	# ssl on;   旧版本设置为on开启ssl功能
        server_name example.com;  # 替换成你的域名
        ssl_certificate cert/example.com.crt;   #替换成你的pem/crt文件名称
        ssl_certificate_key cert/example.com.key;   #替换成你的key文件名称
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  #使用此加密套件。
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   #使用该协议进行配置。
        ssl_prefer_server_ciphers on;  


        # listen       80;
        # server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
}
```

对于用户不知道网站可以进行 https 访问的情况下，可以使用全站加密，让服务器自动把 `http` 的请求重定向到 `https`。在`nginx.conf`文件中添加以下配置：

```json
server {
        listen       80;
        server_name  example.com;#替换成你的域名


        location / {
            # 配置请求转发，将http的请求转发到https上面
            rewrite ^(.*)$ https://example.com/$1 permanent;#替换成你的域名
        }
    }
```

然后重启容器即可。

## 3. 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇主要记录了一下如何将自己的项目制作成Docker镜像，然后能在docker运行。以及Nginx的相关部署。关于docker还有很多东西要学，这里就对自己学的一点点东西做一个笔记，以方便下次使用。其余的等啥时候用到了在进行学习，这次就到这吧。

