1. **什么是Docker**

   + Docker是一个开源的应用容器引擎，基于LXC（Linux Container）内核虚拟化技术实现，提供一系列更强的功能，比如镜像、Dockerfile等；
     Docker理念是将应用及依赖包打包到一个可移植的容器中，可发布到任意Linux发行版Docker引擎上。使用沙箱机制运行程序，程序之间相互隔离；Docker使用Go语言开发 

2. **Docker架构与内部组件**

   + Docker daemon作为服务端接收来自客户端的请求，并处理这些请求，比如创建，运行容器等。客户端提供一系列指令与Docker daemon交互。
   + LXC：Linux容器技术，共享内核，容器共享宿主机资源，使用namespace和cgroup对资源限制和隔离。
   + Cgroups：（control groups）Linxu内核提供的一种限制单进程或者多进程资源的限制，比如cpu 内存等资源的使用限制
   + Namespace：命名空间，也称名字空间，Linux内核提供的一种限制单进程或者多进程资源隔离的机制，一个进程可以属于多个命名空间，Linux内核提供了6种namespace：UTS IPS PID Network Mount User
   + UFS：（UnionFS）联合文件系统，支持将不同位置的目录挂载到同一虚拟文件系统，形成一种分层的模型，成员目录称为虚拟文件系统的一个分支branch
   + AUFS（advanced multi layered unification filesystem）：高级多层统一文件系统，是UFS的一种，每个branch可以指定readonly（ro只读）、 readwrite（读写）和whiteout-able（wo隐藏）权限；一般情况下， aufs只有最上层的branch才有读写权限，其他branch均为只读权限。 《ubuntu默认的存储驱动》

   ![Docker内部结构](E:\DarkhorseRoad\github\SpingCloud\springboot\Docker\images\Docker内部结构.png)

3. **Docker优点**

   + 持续集成
     + 在项目快速打包的情况下，轻量级容器对项目快速构建、环境打包、发布流程就能提高工作效率。
   + 版本控制
     + 每个镜像就是一个版本，在一个项目多个版本时可以很方便的管理。
   + 可移植性
     + 容器可以移植到任意一台Docker主机上。
   + 标准化
     + 应用程序环境及依赖，操作系统等问题，增加了生产环境故障率，容器保证了所有配置，依赖始终不变。
   + 隔离性与安全
     + 容器之间的进程是相互隔离的，一个容器出现问题不会影响其他容器。

4. **虚拟机与容器的区别**

   ![虚拟机与容器的去区别](E:\DarkhorseRoad\github\SpingCloud\springboot\Docker\images\虚拟机与容器的区别.png)

   - 以KVM举例
     - 启动时间
       - Docker秒级，KVM分钟级
     - 轻量级
       - 容器镜像大小通常以M为单位，虚拟机以G为单位
       - 容器资源占用小，要比虚拟机部署更快速
     - 性能
       - 容器共享宿主机内核。系统级虚拟化，占用资源少，没有Hypervisor层开销，容器性能基本接近物理机
       - 虚拟机需要Hypervisor层支持，虚拟化一些设备，具有完整的GuestOS，虚拟化开销大，因而降低性能，没有容器性能好。
     - 安全性
       - 由于共享宿主机内核，只是进程隔离，因此隔离性和稳定性不如虚拟机，容器具有一定权限访问宿主机内核，存在一定安全隐患
     - 使用要求
       - KVM是基于硬件的完全虚拟化，需要硬件CPU虚拟化技术支持，
       - 容器共享宿主机内核，可运行在主流的linux发行版，不考虑CPU是否支持虚拟化技术。

5. 应用场景

   + 应用打包与部署自动化
     + 构建标准化的运行环境：现在大多数方案是在物理机和虚拟机上部署运行环境，面临问题是环境杂乱，完整性迁移难度高等问题。容器即开即用。
   + 自动化测试和持续集成/部署
     + 自动化构建镜像和良好的RESTAPI。能够很好的集成到持续集成/部署环境来。
   + 部署与弹性扩展
     + 由于容器是应用即的，资源占用小，弹性扩展部署速度要更快、
   + 微服务
     + Docker这种容器华隔离技术，正式应对了微服务理念，将业务模块放置到容器中运行，容器的可复用性大大增加了业务模块扩展性。

##### 镜像

1. **什么是镜像**

   + 镜像是一个不包含Linux内核而又精简的Linux操作系统

2. **镜像从哪里来**

   + Docker Hub是由Docker公司维护的的公共注册中心，包含大量的容器镜像，Docker默认从这个公共镜像库下载镜像，https://hub.docker.com/explore 下载慢，可以从国内的源提高下载速度curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://04be47cf.m.daocloud.io 
   + 工作原理

     + 当我们启动一个新的容器时， Docker会加载只读镜像，并在其之上添加一个读写层，并将镜像中的目录复制一份到
       /var/lib/docker/aufs/mnt/容器ID为目录下，我们可以使用chroot进入此目录。**如果运行中的容器修改一个已经存在的文件，
       那么会将该文件从下面的只读层复制到读写层，只读层的这个文件就会覆盖，但还存在，只是隐藏了（ufs）这就实现了文件系统隔离，当删除容
       器后，读写层的数据将会删除，只读镜像不变。 **
   + 镜像文件存储结构

     + docker相关文件存放在： /var/lib/docker目录下
     + /var/lib/docker/aufs/diff # 每层与其父层之间的文件差异
     + /var/lib/docker/aufs/layers/   # 每层一个文件，记录其父层一直到根层之间的ID，大部分文件的最后一行都已，表示继  承来自同一层
     + /var/lib/docker/aufs/mnt    # 联合挂载点，从只读层复制到最上层可读写层的文件系统数据.在建立镜像时，每次写操作，都被视作一种增量操作，即在原有的数据层上添加一个新层；所以一个镜像会有若干个层组成。每次commit提交就会对产生一个ID，就相当于在上一层有加了一层，可以通过这个ID对镜像回滚.

3. 常用操作

   + 检索 docker search 关键字
   + 拉取 docker pull 镜像名：tag（标签)
   + 列表 docker image 
   + 删除 docker rmi image-id
   + export
   + import
   + save
   + load

##### 容器

1. 创建容器常用选项

   + docker create [options] 
     + 创建新的容器不启动
   + docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
     + 创建一个新的容器并且启动
     + 参数
       + -i, --interactive                    Keep STDIN open even if not attached
       + -t, --tty                            Allocate a pseudo-TTY
       + -d, --detach                         Run container in background and print container ID

2. 容器基本操作

   + ps
   + attach
   + rm
   + start
   + stop
   + kill
   + pause/unpause
   + rename

3. 容器更多操作

   + inspect
   + exec
   + top
   + port
   + top
   + cp
   + diffs
   + logs
   + stats
   + update
   + events

4. 容器数据持久化

   + 数据卷

     + 将宿主机的数据目录挂载到容器的目录

   + 数据卷的特点

     + 在容器启动初始化时，如果容器使用的宿主机挂载点有数据，就会将数据拷贝到容器中
     + 数据卷可以直接在容器中共享和重用
     + 可以直接对数据卷里的内容进行修改
     + 数据卷的变化不会影响镜像的变化
     + 卷会一直存在，即使挂载卷的容器已经删除

     ```shell
     root@fupingstar-VirtualBox:/home/hadoop# docker run -itd --name web01 -v /container_web:/data ubuntu
     /container_web是宿主机的目录  /data是容器的目录
     ```

     

   + 容器数据卷

     + 将一个运行的容器作为数据卷，让其他容器通过挂载这个容器实现数据的共享。

       

       

5. 搭建LNMP

   + 创建mysql数据库容器

   

```shell

	
```





##### Docker启动安装停止

1. 安装

   + 检查内核版本

   ```shell
   uname -r
   ```

   + 由于 `apt` 源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 HTTPS 传输的软件包以及 CA 证书。

   ```shell
   $ sudo apt-get update
   
   $ sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       software-properties-common
   ```

   + 鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

     为了确认所下载软件包的合法性，需要添加软件源的 `GPG` 密钥。

     ```shell
     $ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
     
     
     # 官方源
     # $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
     向sources.list中添加软件源
     $ sudo add-apt-repository \
         "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
         $(lsb_release -cs) \
         stable"
     
     
     # 官方源
     # $ sudo add-apt-repository \
     #    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
     #    $(lsb_release -cs) \
     #    stable"
     ```

   + 更新Apt软件包缓存，安装docker

   ```shell
   $ sudo apt-get update
   
   $ sudo apt-get install docker-ce
   ```

   + 启动docker

   ```shell
   $ sudo service docker start
   $ sudo systemctl start docker
   ```

   + 设置开启启动docker

   ```shell
   $ sudo systemctl enable docker
   ```

   + 停止docker

   ```shell
   systemctl stop docker
   service docker stop
   ```

   + 建立docker用户组

   ```shell
   建立docker组
   $ sudo groupadd docker
   
   将当前用户添加到docker组
   ```

   ```bash
   $ sudo usermod -aG docker $USER
   ```



##### 网络

1. 网络模式
2. 容器网络访问原理
3. 桥接宿主机和配置固定IP
4. 容器SSH连接

















