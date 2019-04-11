# docker-note


## docker常用命令

- 查看所有image： ```docker image ls```

- 查看所有container：```docker container ls -a```

- 创建一个新的容器：```docker run  <arguments>  <image name>```

  - arguments：
    - -i -t / -it：配合使用可进入bash，如`docker run -it <image name> bash`
    - --rm：终止容器后容器自动销毁
    - --name：命名容器
    - --dns：容器无法联网时配置DNS
    - ~~--mount：挂载宿主机文件/文件夹~~
    - -v：挂载宿主机文件/文件夹
    - --network=host

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

- 打开一个已经存在的容器``docker exec <arguments> <container name>``(使用exit不会终止容器)；

  或者使用``docker attach <container name>``（使用exit会终止容器）；

  启动已终止容器之前需要``docker container start <container name>``,再配合`attach`进入容器.


## nvidia-docker启动命令

### 1.0

```bash
nvidia-docker run <arguments> <image name>
```

### 2.0

```bash
docker run --runtime=nvidia <arguments> <image name>
```



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



## 在docker中显示图形界面

```bash
xhost +
```

```bash
docker run --runtime=nvidia --rm -it \
-e DISPLAY=unix${DISPLAY} \
-e QT_X11_NO_MITSHM=1 \ #可以在做镜像的时候就设定好这一句
-e GDK_SCALE -e GDK_DPI_SCALE \ #这一行应该不需要
-v /tmp/.X11-unix:/tmp/.X11-unix \
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

## container获取usb摄像头数据
一定要在container启动前将摄像头接到主机上,不然container无法读取,在启动参数里加上`--privileged`.

## docker保存与加载镜像
保存镜像
```bash
docker save [image name]>image_name.tar
```
加载镜像
```bash
docker load<image_name.tar
```
