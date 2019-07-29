# docker-note


## docker常用命令

- 查看所有image： `docker image ls`或`docker images`

- 查看所有container：`docker container ls -a`或`docker ps -l`

- 创建一个新的容器：`docker run  <arguments>  <image name>`

  - arguments：
    - -i -t / -it：配合使用可进入bash，如`docker run -it <image name> bash`
    - --rm：终止容器后容器自动销毁
    - --name：命名容器
    - --dns：容器无法联网时配置DNS
    - ~~--mount：挂载宿主机文件/文件夹~~
    - -v：挂载宿主机文件/文件夹
    - --network=host:使容器使用本地网络

    例如

    ```bash
    docker run \
    -it \
    --dns 192.168.1.1 \
    --rm \
    --runtime=nvidia \
    -v /home/iccd901/cmp/models:/models \
    ubuntu16.04-cmp \
    bash
    ```
    还有更多其他的参数可以根据需要添加，比如下面的图形界面等

- 打开一个已经存在的容器``docker exec <arguments> <container name>``(使用exit不会终止容器)；

  或者使用``docker attach <container name>``（使用exit会终止容器）；

  启动已终止容器之前需要``docker container start <container name>``,再配合`attach`进入容器.

## 修改docker的镜像地址,加快docker pull的速度
修改为中科大源([参考](http://mirrors.ustc.edu.cn/help/dockerhub.html))
### ubuntu16.04
```bash
sudo /etc/docker/daemon.json
```
在这个文件中的大括号里加入
```
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
```
如果这个文件中还有其他内容,要用逗号分隔.
修改完成保存后重启docker服务:
```bash
sudo systemctl restart docker
```
### ubuntu14.04
在配置文件`/etc/default/docker`中的`DOCKER_OPTS`中配置Hub地址：
```
DOCKER_OPTS="--registry-mirror=https://docker.mirrors.ustc.edu.cn/"
```
重启服务:
```bash
sudo service docker restart
```
除了中科大源,还可以使用[daocloud](https://www.daocloud.io/mirror)提供的源:
```
http://f1361db2.m.daocloud.io
```

## nvidia-docker启动命令

### 1.0

```bash
nvidia-docker run <arguments> <image name>
```

### 2.0

```bash
docker run --runtime=nvidia <arguments> <image name>
```

## docker保存与加载镜像
保存镜像
```bash
docker save [image name]>image_name.tar
```
加载镜像
```bash
docker load<image_name.tar
```

## 从容器创建镜像
有的时候我们在容器中进行了一些修改，希望把当前容器保存为新的镜像，可以使用`commit`命令
`docker commit [OPTIONS] CONTAINER_ID [REPOSITORY[:TAG]]`
常用参数有：
-a :提交的镜像作者；
-c :使用Dockerfile指令来创建镜像；
-m :提交时的说明文字；
-p :在commit时，将容器暂停。

## 修改默认软件源为中科大源

```bash
sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```


## Dockerfile联网问题

RUN apt等命令无法成功，需要在本机上的``/etc/docker/daemon.json``中添加dns地址

```bash
nmcli dev show | grep 'IP4.DNS'	#查询ip
```

```bash
{                                                                          
    "dns": ["x.x.x.x", "x.x.x.x"]	#填写上面查询到的ip地址                      
}
```
在这之后需要重启一下docker的进程
```bash
sudo service docker restart
```
有的时候新镜像无法进行`apt-get update`,会卡在中间,这是因为没有`apt-transport-https`这个包,到网上下载了deb文件后利用`dpkg -i`安装就可以了.


## Docker + ROS

```bash
docker run -it ros	#启动一个镜像
$ roscore
```

```bash
docker ps -l	#查看刚才启动镜像的ID
```

```bash
docker exec -it ID bash	#进入刚才的镜像
source /opt/ros/<distro>/setup.bash
```

经过以上操作就可以在一个容器中同时操作两个终端。

## container获取usb摄像头数据
一定要在container启动前将摄像头接到主机上,不然container无法读取,在启动参数里加上`--privileged`.


## 在docker中显示图形界面
在宿主机上:
```bash
xhost +
```
启动容器时：
```bash
docker run --runtime=nvidia --rm -it \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY=unix${DISPLAY} \
-e QT_X11_NO_MITSHM=1 \ #可以在做镜像的时候就设定好这一句
-e GDK_SCALE -e GDK_DPI_SCALE \ #这一行不写应该也没问题，不知道是干什么的
<image name>
```



## docker中安装cuda显示Can't locate Tie/File.pm in @INC

```bash
apt -y install libmodule-install-perl
```



## 将MATLAB挂载到docker容器中

```bash
docker run -it --rm \
-v /usr/local/MATLAB/R2018a:/usr/local/MATLAB/from-host \
-v ~/matlab-logs:/var/log/matlab \
--mac-address=10:7b:44:17:d7:8f
```

## RuntimeError: DataLoader worker (pid 1821) is killed by signal: Bus error.
This error is caused by lack of shared memory when using pytorch.
**solution**: When starting a container, add the argument `--shm-size 10G`. Or simply reduce the size of num_workers in dataloader.
