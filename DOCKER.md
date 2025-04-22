# DOCKER

# 虚拟机

虚拟机是模仿电脑硬件的软件，相当于一台装在电脑里的电脑，它有自己的操作系统，内存，磁盘空间。

虚拟机之间互不影响，可以运行不同的操作系统，可以在虚拟机中随机装软件而不用担心搞坏主系统。



# Docker

因为虚拟机本质上是一台电脑，而每台电脑都需要操作系统，操作系统的存在会占用大量内存同时启动会比较缓慢，对于部署应用程序的需求使用虚拟机过于浪费。

为了克服虚拟机的缺点(还需利用虚拟机独立的特性)，出现了容器技术——只隔离应用程序运行时的环境，容器之间可以共享一个操作系统。容器更加轻量级且占用资源更少。

docker是容器技术的实现

docker中有镜像和容器

+ 镜像：一个只读的模板，里面包含了运行某个程序或服务所需的所有内容
+ 容器：镜像的运行实例，运行起来的环境



Dockerfile是一个文本文件，包含了一系列指令，用于构建Docker镜像

<img src="./assets/image-20250421122911664.png" alt="image-20250421122911664" style="zoom:67%;" />



# Dockerfile

指定基础镜像 即新镜像基于谁来构建 

```dockerfile
FROM python:3.11.8-alpine、
```



python镜像有三种

| 镜像标签             | 体积大小（大概） | 特点                   |
| -------------------- | ---------------- | ---------------------- |
| `python:3.11`        | 900MB 左右       | 完整构建工具、调试支持 |
| `python:3.11-slim`   | 250MB 左右       | 去掉文档和编译工具     |
| `python:3.11-alpine` | 50MB 左右        | 极小，但可能依赖缺失多 |



设置环境变量

```python
ENV FLASK_APP=run.py
ENV FLASK_CONFIG=docker
```



一般不以root身份执行保证安全性，因此要创建新用户

```python
# python:3.11-alpine下
RUN adduser -D flasky

# python:3.11-slim下
RUN adduser --disabled-password --gecos "" flasky
```



接下来用户操作都以flasky用户身份执行

```python
USER flasky
```





设置容器内的工作目录，后续的所有指令(RUN CMD COPY都会基于这个文件工作)，该目录位于DOCKER容器内部的虚拟文件系统

```python
WORKDIR /app
```



COPY [源目录/文件] [目标位置]

```python
COPY requirements requirements

COPY app app 
COPY migrations migrations
# 复制多个到当前目录下
COPY run.py config.py boot.sh ./
```



RUN是构建镜像时执行的命令

```python
RUN python -m venv venv
RUN venv/bin/pip install -r requirements/docker.txt
```



声明要监听的端口

```python
EXPOSE 5000
```



建立容器时运行的脚本或命令

```python
# 容器启动时要运行的脚本
ENTRYPOINT [ "./boot.sh" ]

# 指定容器启动时的默认命令 可以设定参数
CMD ["python", "main.py"]
```



# 初始化

构建docker镜像，-t 镜像名字 . 在当前目录下寻找dockfile文件

```python
docker build -t 'testflask' .
```

想要更新镜像就要重新构建，重新构建会基于已有的镜像构建，速度较快



查看已有或已拉取的镜像

```python
docker images ls
```



# 推送

首先登录，似乎因为本地下载了docker desktop 直接就登上了

```python
docker login
```



对要推送的镜像要打一个标签

```dockerfile
docker tag 镜像名 nndjxh(docker hub用户名)/flask_web(子目录，也可以不加)/flask(本地镜像)
```



最后推送

```python
docker push nndjxh/flask_web/flask	
```



# 运行



```python
docker run --name flasky -d -p 8000:5000 \
 -e SECRET_KEY=57d40f677aff4d8d96df97223c74d217 \
 -e MAIL_USERNAME=<your-gmail-username> \
 -e MAIL_PASSWORD=<your-gmail-password> flasky:latest
```

+ --name 设置容器的名称
+ -d 设置后台运行
+ -p 设置端口映射，将运行机器的8000端口映射到5000，flask应用一般监听5000端口
+ -e设置环境变量

执行命令后终端会输出容器的ID，根据该ID可以停止容器

```python
docker stop 71357ee776ae
```



删除容器

```python
docker rm 71357ee776ae
```



停止并删除

```python
 docker rm -f 71357ee776ae
```



查看当前正在运行的容器

```python
docker ps
```

