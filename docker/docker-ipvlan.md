# docker 使用 ipvlan 网络

- Archlinux 
- docker 1.13

基于 archlinux 系统的 ipvlan 安装配置，ipvlan 属于 Linux 的一个内核模块, 所以要求 Linux 内核版本要大于 4.2 因为 ipvlan 的特性，容器默认无法与容器所在宿主机的 enp0s3 网卡通信，但可以跨主机与其他主机或者其他主机上的容器通信

## 配置 docker 网络使用 ipvlan

docker 的 ipvlan 网络驱动属于实验性功能，必须要开启 docker 的 Experimental 功能

```
$ emacs /etc/docker/daemon.json
{
    "experimental": true
}
```

宿主机 IP 地址

```
$ ip addr show enp0s3
enp0s3 inet 192.168.1.100/24 scope global enp0s3
```

创建 docker 网络
```
$ docker network  create -d ipvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=enp0s3 ipvlan
```

查看 docker 的网络驱动情况

```
$ docker network ls
```

启动测试容器

```
$ docker  run --net=ipvlan --name=ipv1 --ip 192.168.1.2 -itd alpine /bin/sh
$ docker  run --net=ipvlan --name=ipv2 --ip 192.168.1.3 -it --rm alpine /bin/sh
```

在容器内部 ping 测试

```
$ ping 192.168.1.2
```

----------------------------------------------------------------------------------------

By typefo typefo@qq.com Update: 2017-04-29 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
