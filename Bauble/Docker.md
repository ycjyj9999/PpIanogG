# docker总结
## 为什么用docker
虽然上手docker的第一天遇到了点坑，但是总体来看，docker确实快捷方便，可以和他人共享，可以方便自己随时随地切换环境，也方便公司电脑与自己笔记本配置一样的环境。
## 安装docker
我们使用的是docker-ce
```
sudo apt-get update
sudo apt-get install docker-ce
```
## 第一次运行
1.首先我们要下载一个镜像文件，这里以我自己的docker仓库中的ycjyj9999/liuyichen为例
```
sudo docker pull ycjyj9999/liuyichen
```
我们可以用images来查看我们所下载的镜像，镜像的id用来唯一识别这个镜像
```
sudo docker images
```

2.然后运行这个镜像文件，这时候docker会自动分配给这个镜像分配一个容器
```
sudo docker run -i -t ycjyj9999/liuyichen /bin/bash
```
-i是为了让其在分配容器后处于up状态，-t是分配一个终端进程来交互，这里的/bin/bash是默认的参数,可以不写
## 第一次修改
1.在执行上面的命令后会自动进入终端，你可以进行自己的修改，安装应用，修改环境，部署自己所需要的东西
2.用exit命令退出容器的终端，在回到宿主终端后，执行ps命令
```
sudo docker ps -l
```
这里会显示你上一次操作的容器的详情，记录下它的CONTAINER ID，可以用来操作这个容器的开关，修改这个容器的名字等等

ps命令也可以看现在已经创建的所有容器，用-a参数:
```
sudo docker ps -a
```
2.举个例子，现在有一个容器,它的id是bf56f067b588：

①开启/关闭容器
```
sudo docker start/stop bf56f067b588
```
②当你用exit退出一个容器的终端后，你想重新进入这个容器，你不能用run，因为run会重新开启分配一个容器给你选中的镜像，而不是进入你原来的容器，你需要用exec这个命令
```
sudo docker exec -i -t 你要进入的容器id /bin/bash
```
③修改容器名字
```
sudo docker rename 容器id/旧名字 新名字
```

④删除容器
```
sudo docker rm bf56f067b588
```
## 第一次上传
1.当你容器修改完成并用ps查看记录下这个容器的id后，就可以进行commit了
```
sudo docker commit 修改容器id 镜像名字
```
注意：

①这里的镜像名称不是镜像id，是REPOSITORY属性。

②如果镜像名字是没有被占用的新名字，则会创建以此命名的记录了修改内容的新镜像文件。

③如果镜像名字已经存在，则会旧镜像已经存在的名字会被commit生成的新镜像夺走，旧镜像名字和tag都会变为none。
若是想修改

2.在push之前你需要登录自己的docker账号
```
sudo docker login
```
3.当你将你的修改好的镜像记录成功，就可以进行上传了
```
sudo docker push 镜像名字
```
push后你会在你的仓库里发现自己的镜像已经上传成功！

4.修改镜像名字
```
sudo docker tag 原镜像id 新名字:tag
```
注意：

①这里的修改，并不是真正的覆盖修改，而是创建一个以新名字为REPOSITORY的新镜像，所以如果新名字与原来id所对应的镜像相同，则会导致原镜像的tag出现none

②可以发现由同一镜像文件派生出来的新镜像，它们的create time是一样的。

③:tag可以不用加上，默认值的tag为latest

4.删除镜像

如果有不需要的镜像，可以用命令删除
```
sudo docker rmi 镜像id/镜像名字:tag
```
注意：

①删除的镜像若为父镜像，需要优先删除子镜像

②删除的镜像若仍在容器中被使用，需要先关闭使用并删除容器

## 访问宿主主机文件
 Docker容器启动的时候，如果要挂载宿主机的一个目录，可以用-v参数指定。

譬如我要启动一个容器，宿主机的/test目录挂载到容器的/soft目录，可通过以下方式指定：

```
docker run -it -v /test:/soft  镜像名字:tag/镜像id  /bin/bash
```

这样在容器启动后，容器内会自动创建/soft的目录。通过这种方式，我们可以明确一点，即-v参数中，冒号":"前面的目录是宿主机目录，后面的目录是容器内目录。

注意：

①容器目录不可以为相对路径

②宿主机目录如果不存在，则会自动生成

③宿主机的目录如果为相对路径，则需要通过docker inspect命令，查看容器“Mounts”那一部分，得到所创建的文件夹位置
```
sudo docker inspect 容器id
```
[文件共享详解](http://blog.csdn.net/magerguo/article/details/72514813)
