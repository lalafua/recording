# Process Recording

## 5.23 Fri (Last modified on 5.26)

- [x] **飞控与 RPI 的通信设置**

CUAV 7-Nano 飞控板上有一个 ETH 接口，还附带一根 RJ45--GH1.25-4P 线，于是很自然的用它与 RPI 通信。

![](./assets/picture_01.png)

飞控上：

飞控的默认 IP 为 192.168.144.14/24，设置：

```sh
NET_DHCP=0
NET_ENABLE=1 
NET_P1_TYPE=1       # UDP Client
NET_P1_PROTOCAL=2   # MAVLink2
NET_P1_IP0=192
NET_P1_IP1=168
NET_P1_IP2=144
NET_P1_IP3=15
NET_P1_PORT=15001
```

`NET_P1_XXXX` 这里含义的是飞控作为客户端，向 `udp://192.168.144.15:15001` （RPI IP）发送数据包。关于这些参数的详细解释和其他选项参考 [Complete Parameter List](https://ardupilot.org/rover/docs/parameters.html#)。

RPI 上：

前面提到设置的 RPI IP 地址为 192.168.144.15，于是用 `netplan` 对 `eth0` 进行设置。`netplan` 旨在简化和统一 `Linux` 网络配置的工具，通过声明式的 `ymal` 文件，让管理员能够更轻松、更安全地管理复杂的网络设置。编写完 `yaml` 文件之后 `netplan` 会将这些 `ymal` 配置转换成特定后端网络管理服务（主要是 `systemd-networkd` 或 `NetworkManager` ）能够理解的配置。

```sh
$ vim /etc/netplan/25-eth0-static.ymal
```

填入：

```ymal
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      dhcp6: no
      addresses: [192.168.144.15/24] # 飞控板上设置的目标 IP
```

之后应用配置：

```sh
$ sudo netplan apply
```

查看网卡设置： 

```sh
$ ip addr show

...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 2c:cf:67:37:10:28 brd ff:ff:ff:ff:ff:ff
    inet 192.168.144.15/24 brd 192.168.144.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::2ecf:67ff:fe37:1028/64 scope link
       valid_lft forever preferred_lft forever
...
```

飞控和 RPI 都设置无误之后，验证：

```sh
$ ping 192.168.144.14

PING 192.168.144.14 (192.168.144.14) 56(84) bytes of data.
64 bytes from 192.168.144.14: icmp_seq=1 ttl=255 time=0.197 ms
64 bytes from 192.168.144.14: icmp_seq=2 ttl=255 time=0.178 ms
64 bytes from 192.168.144.14: icmp_seq=3 ttl=255 time=0.209 ms
64 bytes from 192.168.144.14: icmp_seq=4 ttl=255 time=0.787 ms
64 bytes from 192.168.144.14: icmp_seq=5 ttl=255 time=0.181 ms
^C
--- 192.168.144.14 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4080ms
rtt min/avg/max/mdev = 0.178/0.310/0.787/0.238 ms

$ dig 192.168.144.14

; <<>> DiG 9.18.30-0ubuntu0.22.04.2-Ubuntu <<>> 192.168.144.14
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42387
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;192.168.144.14.                        IN      A

;; ANSWER SECTION:
192.168.144.14.         0       IN      A       192.168.144.14

;; Query time: 7 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Mon May 26 11:59:14 UTC 2025
;; MSG SIZE  rcvd: 59
```

启动 `mavros client` :

```sh
$ ros2 launch mavros apm.launch fcu_url:=udp://:15001@

...
[mavros_node-1] [INFO] [1748260217.290740452] [mavros.sys]: VER: 1.1: Capabilities         0x000000000000f1ef
[mavros_node-1] [INFO] [1748260217.290928250] [mavros.sys]: VER: 1.1: Flight software:     040600ff (3564396232636339)
[mavros_node-1] [INFO] [1748260217.290984399] [mavros.sys]: VER: 1.1: Middleware software: 00000000 (0000000000000000)
[mavros_node-1] [INFO] [1748260217.291033696] [mavros.sys]: VER: 1.1: OS software:         00000000 (3030363438623838)
[mavros_node-1] [INFO] [1748260217.291201124] [mavros.sys]: VER: 1.1: Board hardware:      1b580000
[mavros_node-1] [INFO] [1748260217.291264124] [mavros.sys]: VER: 1.1: VID/PID:             1209:5740
[mavros_node-1] [INFO] [1748260217.291306069] [mavros.sys]: VER: 1.1: UID:                 0000000000000000
[mavros_node-1] [INFO] [1748260217.810181549] [mavros.mavros]: CON: Got HEARTBEAT, connected. FCU: ArduPilot
[mavros_node-1] [INFO] [1748260217.810860445] [mavros.mission]: WP: detected enable_partial_push: 1
[mavros_node-1] [INFO] [1748260217.814026254] [mavros.rc]: RC_CHANNELS message detected!
[mavros_node-1] [INFO] [1748260217.814519055] [mavros.imu]: IMU: Raw IMU message used.
...

...
[mavros_node-1] [INFO] [1748260218.830760126] [mavros.sys]: VER: 1.1: Capabilities         0x000000000000f1ef
[mavros_node-1] [INFO] [1748260218.830953405] [mavros.sys]: VER: 1.1: Flight software:     040600ff (9cc2b9d5)
[mavros_node-1] [INFO] [1748260218.831009795] [mavros.sys]: VER: 1.1: Middleware software: 00000000 (        )
[mavros_node-1] [INFO] [1748260218.831059314] [mavros.sys]: VER: 1.1: OS software:         00000000 (88b84600   @W��)
[mavros_node-1] [INFO] [1748260218.831104777] [mavros.sys]: VER: 1.1: Board hardware:      1b580000
[mavros_node-1] [INFO] [1748260218.831146981] [mavros.sys]: VER: 1.1: VID/PID:             1209:5740
[mavros_node-1] [INFO] [1748260218.831190445] [mavros.sys]: VER: 1.1: UID:                 0000000000000000
...

...
[mavros_node-1] [INFO] [1748260227.830730840] [mavros.sys]: FCU: ArduRover V4.6.0 (9cc2b9d5)
[mavros_node-1] [INFO] [1748260227.831353401] [mavros.sys]: FCU: ChibiOS: 88b84600
[mavros_node-1] [INFO] [1748260227.832218372] [mavros.sys]: FCU: CUAV-7-Nano 003C0030 30335115 33383936
[mavros_node-1] [INFO] [1748260227.832828229] [mavros.sys]: FCU: RCOut: PWM:1-14
[mavros_node-1] [INFO] [1748260227.833713867] [mavros.sys]: FCU: IMU0: fast sampling 2.0kHz/2.0kHz
...
```

看看 `socket` ：

```sh
$ ss -alu # 启动 mavros client 之后

...
UNCONN            0                 0                                    0.0.0.0:15001                               0.0.0.0:*
...
```

看看 `mavros` 提供的 `topic` ， `node` 和 `interface`：

```sh
$ ros2 topic list

...
/mavros/state
/mavros/status_event
/mavros/statustext/recv
/mavros/statustext/send
/mavros/sys_status
...

$ ros2 node list

...
/mavros/guided_target
/mavros/home_position
/mavros/imu
/mavros/landing_target
/mavros/local_position
...

$ ros2 interface list

Messages:
    ...
    mavros_msgs/msg/State
    mavros_msgs/msg/StatusEvent
    mavros_msgs/msg/StatusText
    mavros_msgs/msg/SysStatus
    mavros_msgs/msg/TerrainReport
    ...
Services:
    ...
    mavros_msgs/srv/SetMode
    mavros_msgs/srv/StreamRate
    mavros_msgs/srv/WaypointClear
    mavros_msgs/srv/WaypointPull
    mavros_msgs/srv/WaypointPush
    ...
Actions:
    ...
    control_msgs/action/FollowJointTrajectory
    control_msgs/action/GripperCommand
    control_msgs/action/JointTrajectory
    control_msgs/action/ParallelGripperCommand
    control_msgs/action/PointHead
    control_msgs/action/SingleJointPosition
    ...
```

参考：
- [mavros_tutorial](https://masoudir.github.io/mavros_tutorial/) （虽然是 ROS Melodic 的 tutorial，但是仍然适用于当前的 mavros）
- [Ardupilot Copter - Ethernet / Network Setup](https://ardupilot.org/copter/docs/common-network.html#common-network)

PS. 地面站的选择


## 5.25 Sun

- [x] **在 RPI (Ubuntu Server 22.04.5) 上启用 X11 转发**

为了避免 desktop 占用大量资源（对 RPI 4B），我选择使用 server 版 OS，但是某些应用比如 rqt 对实际开发还是很有帮助的，CLI 调用 `ros2 echo[...]` 的效率并不高，不能快速找到自己需要的东西。

STFW 后发现可以启用 ssh 的 X11 Forwading 转发 gui 应用到宿主机上显示。

在 RPI 上：

```sh
$ sudo apt update
$ sudo apt install -y xauth xorg openbox
```

可能还需要编辑 `/etc/ssh/sshd_config`，启用：

```
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes
```

```
# 并重启 sshd 服务
$ sudo systemctl restart sshd
```

- [x] **rqt 插件加载问题**

树莓派上使用 `sudo apt install ros-$ROS_DISTRO-rqt*` 安装`rqt` 之后发现：

```sh
$ rqt --list-plugins

ros-humble-rqt-mocap4r2-control-dbgsym/jammy,now 0.0.7-1jammy.20250430.070001 arm64 [installed]
ros-humble-rqt-mocap4r2-control/jammy,now 0.0.7-1jammy.20250430.070001 arm64 [installed]
```

但是检查插件是全部安装好了的：

```sh
$ apt list --installed | grep "ros-$ROS_DISTRO-rqt"

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

ros-humble-rqt-action/jammy,now 2.0.1-3jammy.20250430.085713 arm64 [installed]
ros-humble-rqt-bag-plugins/jammy,now 1.1.5-1jammy.20250513.024448 arm64 [installed]
ros-humble-rqt-bag/jammy,now 1.1.5-1jammy.20250430.120846 arm64 [installed]
ros-humble-rqt-common-plugins/jammy,now 1.2.0-1jammy.20250513.025002 arm64 [installed]
ros-humble-rqt-console/jammy,now 2.0.3-1jammy.20250430.084731 arm64 [installed]
ros-humble-rqt-controller-manager/jammy,now 2.50.0-1jammy.20250430.084735 arm64 [installed]
ros-humble-rqt-dotgraph/jammy,now 0.0.4-1jammy.20250430.085903 arm64 [installed]
ros-humble-rqt-gauges/jammy,now 0.0.3-1jammy.20250430.084738 arm64 [installed]
ros-humble-rqt-graph/jammy,now 1.3.1-1jammy.20250430.085402 arm64 [installed]
ros-humble-rqt-gui-cpp-dbgsym/jammy,now 1.1.7-1jammy.20250430.065222 arm64 [installed]                                                                  [10/247]
ros-humble-rqt-gui-cpp/jammy,now 1.1.7-1jammy.20250430.065222 arm64 [installed]
ros-humble-rqt-gui-py/jammy,now 1.1.7-1jammy.20250430.084018 arm64 [installed]
ros-humble-rqt-gui/jammy,now 1.1.7-1jammy.20250430.024539 arm64 [installed]
ros-humble-rqt-image-overlay-dbgsym/jammy,now 0.1.3-1jammy.20250430.092010 arm64 [installed]
ros-humble-rqt-image-overlay-layer/jammy,now 0.1.3-1jammy.20250430.034157 arm64 [installed]
ros-humble-rqt-image-overlay/jammy,now 0.1.3-1jammy.20250430.092010 arm64 [installed]
ros-humble-rqt-image-view-dbgsym/jammy,now 1.2.0-2jammy.20250430.070002 arm64 [installed]
ros-humble-rqt-image-view/jammy,now 1.2.0-2jammy.20250430.070002 arm64 [installed]
ros-humble-rqt-joint-trajectory-controller/jammy,now 2.45.0-1jammy.20250430.084732 arm64 [installed]
ros-humble-rqt-mocap4r2-control-dbgsym/jammy,now 0.0.7-1jammy.20250430.070001 arm64 [installed]
ros-humble-rqt-mocap4r2-control/jammy,now 0.0.7-1jammy.20250430.070001 arm64 [installed]
ros-humble-rqt-moveit/jammy,now 1.0.1-3jammy.20250430.090331 arm64 [installed]
ros-humble-rqt-msg/jammy,now 1.2.0-1jammy.20250430.085224 arm64 [installed]
ros-humble-rqt-plot/jammy,now 1.1.5-1jammy.20250513.003458 arm64 [installed]
ros-humble-rqt-publisher/jammy,now 1.5.0-1jammy.20250430.085503 arm64 [installed]
ros-humble-rqt-py-common/jammy,now 1.1.7-1jammy.20250430.024617 arm64 [installed]
ros-humble-rqt-py-console/jammy,now 1.0.2-3jammy.20250430.085015 arm64 [installed]
ros-humble-rqt-reconfigure/jammy,now 1.1.2-1jammy.20250430.085248 arm64 [installed]
ros-humble-rqt-robot-dashboard/jammy,now 0.6.1-3jammy.20250430.085837 arm64 [installed]
ros-humble-rqt-robot-monitor/jammy,now 1.0.6-1jammy.20250430.085252 arm64 [installed]
ros-humble-rqt-robot-steering/jammy,now 1.0.1-1jammy.20250430.085248 arm64 [installed]
ros-humble-rqt-runtime-monitor/jammy,now 1.0.0-3jammy.20250430.085245 arm64 [installed]
ros-humble-rqt-service-caller/jammy,now 1.0.5-3jammy.20250430.085546 arm64 [installed]
ros-humble-rqt-shell/jammy,now 1.0.2-3jammy.20250430.085812 arm64 [installed]
ros-humble-rqt-srv/jammy,now 1.0.3-3jammy.20250430.085746 arm64 [installed]
ros-humble-rqt-tf-tree/jammy,now 1.0.5-1jammy.20250430.085851 arm64 [installed]
ros-humble-rqt-topic/jammy,now 1.5.0-1jammy.20250430.085816 arm64 [installed]
ros-humble-rqt/jammy,now 1.1.7-1jammy.20250430.085405 arm64 [installed]
```

使用 `rqt_graph --force-discover` 发现可以正常加载 node graph 插件，因此猜测是缓存问题，ATFA (Ask The Fuck AI) 后找到缓存位置，删除 rqt_gui 的缓存：
```sh
rm -f .config/ros.org/rqt_gui.ini 
```
之后，rqt_gui 可以正常加载已安装的插件。