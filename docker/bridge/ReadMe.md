# Docker bridge

[use-the-default-bridge-network-tutorial](https://docs.docker.com/network/network-tutorial-standalone/#use-the-default-bridge-network)

```sh
Welcome to Alibaba Cloud Elastic Compute Service !

Last login: Sat Jul 24 13:41:08 2021 from 101.229.88.131

# docker network ls - 查看当前所有的docker network

root@iZbp18ily7toc6ikp5rtktZ:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
c7129c314b52   bridge    bridge    local
28d2f808c15f   host      host      local
8d6a3649cdfb   jenkins   bridge    local
f435464dd567   none      null      local

# 拉取镜像alpine，命名其为alpine1，ash是alpine中特有的shell，不同于bin/bash
# -dit: detached interactive TTY(see input and output)
# the container id will print in the console
root@iZbp18ily7toc6ikp5rtktZ:~# docker run -dit  --name alpine1 alpine ash
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
5843afab3874: Pull complete
Digest: sha256:234cb88d3020898631af0ccbbcca9a66ae7306ecd30c9720690858c1b007d2a0
Status: Downloaded newer image for alpine:latest
d1d602ae7795494c8ffa28cc643fd7a7b08afd7623dd9a6c205657c7c032e62d

# start another container 
root@iZbp18ily7toc6ikp5rtktZ:~# docker run -dit --name alpine2 alpine ash
1676e76513e592ef2c3979613d38950c0170985f0249a4b89244bee5ab66aad1
root@iZbp18ily7toc6ikp5rtktZ:~# docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
1676e76513e5   alpine    "ash"     8 seconds ago    Up 7 seconds              alpine2
d1d602ae7795   alpine    "ash"     18 seconds ago   Up 17 seconds             alpine1

# list all the bridge ips within default bridge
root@iZbp18ily7toc6ikp5rtktZ:~# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "c7129c314b526ee5edea7f66a8049e2281661cb634e1949b8e1ba4d7b5a63e30",
        "Created": "2021-07-24T11:23:02.54861788+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "1676e76513e592ef2c3979613d38950c0170985f0249a4b89244bee5ab66aad1": {
                "Name": "alpine2",
                "EndpointID": "4538758a3c11c71fd132b0aef9639a500443d6253becfded754c3ab021ab7410",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16", # This is the container alpine2's ip address
                "IPv6Address": ""
            },
            "d1d602ae7795494c8ffa28cc643fd7a7b08afd7623dd9a6c205657c7c032e62d": {
                "Name": "alpine1",
                "EndpointID": "c61ce32e5d2aaf5bacc5e07b507777919c07ea8c389d392b12f254ec05d225eb",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16", # This is the container alpine1's ip address
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
root@iZbp18ily7toc6ikp5rtktZ:~#


# attach to the container
root@iZbp18ily7toc6ikp5rtktZ:~# docker attach alpine1

# print all the ips
/ # ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
5: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# pint two times to www.baidu.com
/ # ping -c 2 www.baidu.com
PING www.baidu.com (180.101.49.12): 56 data bytes
64 bytes from 180.101.49.12: seq=0 ttl=49 time=13.026 ms
64 bytes from 180.101.49.12: seq=1 ttl=49 time=13.048 ms

--- www.baidu.com ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 13.026/13.037/13.048 ms
/ #


# ping container alpine2 using ip address

/ # ping 172.17.0.3 -c 2
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.149 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.098 ms

--- 172.17.0.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.098/0.123/0.149 ms


# ping container alpine2 using container name
# 可见使用container name是ping不通的
/ # ping -c 2 alpine2
ping: bad address 'alpine2'
/ #

# stop container alpine1 alpine2
root@iZbp18ily7toc6ikp5rtktZ:~# docker container stop alpine1 alpine2
alpine1
alpine2
# remove container alpine1 alpine2
root@iZbp18ily7toc6ikp5rtktZ:~# docker container rm alpine1 alpine2
alpine1
alpine2
root@iZbp18ily7toc6ikp5rtktZ:~#

```

由此得知：

- 默认的docker bridge，创建的container默认会被加入到default docker bridge network中
- container和container之间是可以通过ip地址互相连通的
- container和container之间无法通过container name连通
- 默认docker bridge network是不推荐使用在产品环境中的

## user-defined bridge network

- 我们即将创建4个container，两个连通在自定义bridge network，第三个连通到bridge network，第四个既连通到default bridge network，也连通到自定义bridge network

```sh

# create user-defined bridge network
root@iZbp18ily7toc6ikp5rtktZ:~# docker network create --driver bridge alpine-net
e09fc482b4e5b34353cd562750251ae96772751da9afc2bce3c40a4eec45aeb0

# query all networks
root@iZbp18ily7toc6ikp5rtktZ:~# docker network ls
NETWORK ID     NAME         DRIVER    SCOPE
e09fc482b4e5   alpine-net   bridge    local
c7129c314b52   bridge       bridge    local # 这个是默认的bridge network
28d2f808c15f   host         host      local
8d6a3649cdfb   jenkins      bridge    local
f435464dd567   none         null      local

# 查看创建的bridge network
root@iZbp18ily7toc6ikp5rtktZ:~# docker inspect network alpine-net
[
    {
        "Name": "alpine-net",
        "Id": "e09fc482b4e5b34353cd562750251ae96772751da9afc2bce3c40a4eec45aeb0",
        "Created": "2021-07-24T16:30:48.784024455+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16", # 子网掩码
                    "Gateway": "172.20.0.1" # 网关
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
Error: No such object: network
root@iZbp18ily7toc6ikp5rtktZ:~#


## 创建两个使用自定义bridge network的container
## 通过--network选项指定bridge network

# 创建alpine1, alpine2使用自定义bridge network
root@iZbp18ily7toc6ikp5rtktZ:~# docker run -dit --name alpine1 --network alpine-net alpine ash
e920e089c05ce704330909813e8849be980d71d0134150f0b0f76abbb2d647cb
root@iZbp18ily7toc6ikp5rtktZ:~# docker run -dit --name alpine2 --network alpine-net alpine ash
4a9af0d374f436cf0f6b0eec44d3d1d0d1cbc1de046cb2892f54373eebbf78d6

# 创建alpine3，使用默认bridge network
root@iZbp18ily7toc6ikp5rtktZ:~# docker run -dit --name alpine3 alpine ash
ded1d376740d5c44238da9d1675c9d6fe59c0a69fd3a7828b83974eb13fd95a3

# 创建alpine4, 既连接default bridge network, 也连接自定义network
root@iZbp18ily7toc6ikp5rtktZ:~# docker run -dit --name alpine4 --network alpine-net alpine ash
5c63668ab4a17a2dd3351f6b51f0256b1d3662eccc2bdadc0c7f5e0e5348b29c
root@iZbp18ily7toc6ikp5rtktZ:~# docker network connect bridge alpine4
root@iZbp18ily7toc6ikp5rtktZ:~#


# 查看自定义bridge network
root@iZbp18ily7toc6ikp5rtktZ:~# docker  network inspect alpine-net
[
    {
        "Name": "alpine-net",
        "Id": "e09fc482b4e5b34353cd562750251ae96772751da9afc2bce3c40a4eec45aeb0",
        "Created": "2021-07-24T16:30:48.784024455+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4a9af0d374f436cf0f6b0eec44d3d1d0d1cbc1de046cb2892f54373eebbf78d6": {
                "Name": "alpine2",
                "EndpointID": "3fed220af0763a7be92babaeff2766f0a2705da9ed07bd2f9d03f9606532c648",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            },
            "5c63668ab4a17a2dd3351f6b51f0256b1d3662eccc2bdadc0c7f5e0e5348b29c": {
                "Name": "alpine4",
                "EndpointID": "804bab319b3d7820efdd3a1b19ac864f572c7b553294b6cbbe99490d7c424c83",
                "MacAddress": "02:42:ac:14:00:04",
                "IPv4Address": "172.20.0.4/16",
                "IPv6Address": ""
            },
            "e920e089c05ce704330909813e8849be980d71d0134150f0b0f76abbb2d647cb": {
                "Name": "alpine1",
                "EndpointID": "7e41934190922865a9ebefad6cc9d55dffee8d668108ca5730bc7c7000f26f2e",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
root@iZbp18ily7toc6ikp5rtktZ:~#



# 进入container alpine1，通过ip和container name来ping alpine2

root@iZbp18ily7toc6ikp5rtktZ:~# docker attach alpine1
/ # ping -c 2 alpine2
PING alpine2 (172.20.0.3): 56 data bytes
64 bytes from 172.20.0.3: seq=0 ttl=64 time=0.104 ms
64 bytes from 172.20.0.3: seq=1 ttl=64 time=0.099 ms

--- alpine2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.099/0.101/0.104 ms
/ # ping -c 2 172.20.0.3
PING 172.20.0.3 (172.20.0.3): 56 data bytes
64 bytes from 172.20.0.3: seq=0 ttl=64 time=0.089 ms
64 bytes from 172.20.0.3: seq=1 ttl=64 time=0.130 ms

--- 172.20.0.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.089/0.109/0.130 ms
/ #


# 尝试在alpine1中ping alpine3，发现失败

/ # ping -c 2 alpine3
ping: bad address 'alpine3'
/ # ping -c 3 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
/ #


# alpine4 也是可以ping通的
/ # ping -c 3 alpine4
PING alpine4 (172.20.0.4): 56 data bytes
64 bytes from 172.20.0.4: seq=0 ttl=64 time=0.098 ms
64 bytes from 172.20.0.4: seq=1 ttl=64 time=0.101 ms
64 bytes from 172.20.0.4: seq=2 ttl=64 time=0.096 ms

--- alpine4 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.096/0.098/0.101 ms
/ #




# 尝试从alpine4中ping alpine1,2,3

# ping alpine3，可以ping通，但是要通过ip
/ # ping -c 2 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.117 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.096 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.096/0.106/0.117 ms
/ # ping -c 2 alpine3
ping: bad address 'alpine3'
/ #

# 通过container name alpine2，可以连通
/ # ping -c 2 alpine2
PING alpine2 (172.20.0.3): 56 data bytes
64 bytes from 172.20.0.3: seq=0 ttl=64 time=0.088 ms
64 bytes from 172.20.0.3: seq=1 ttl=64 time=0.103 ms

--- alpine2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.088/0.095/0.103 ms

# 通过alpine2 ip address，也可以ping通
/ # ping -c 2 172.20.0.3
PING 172.20.0.3 (172.20.0.3): 56 data bytes
64 bytes from 172.20.0.3: seq=0 ttl=64 time=0.089 ms
64 bytes from 172.20.0.3: seq=1 ttl=64 time=0.101 ms

--- 172.20.0.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.089/0.095/0.101 ms
/ #


root@iZbp18ily7toc6ikp5rtktZ:~# docker attach alpine4
You cannot attach to a stopped container, start it first
root@iZbp18ily7toc6ikp5rtktZ:~# docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
ded1d376740d   alpine    "ash"     14 minutes ago   Up 13 minutes             alpine3
root@iZbp18ily7toc6ikp5rtktZ:~# docker container stop alpine3
alpine3
root@iZbp18ily7toc6ikp5rtktZ:~#

```

由此可知：

- 对于自定义bridge network，既可以通过ip通信，也可以通过container name通信，都可以ping通
- 不在同一个bridge network的，比如alpine1和alpine3，是无法互相通信的
- 既在default bridge network，也在自定义bridge network的alpine4，既可以ping通自定义network的，也可以ping通默认bridge network的
