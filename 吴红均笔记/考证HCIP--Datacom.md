[TOC]
# 认识网络设备
## 设备构成
## 交换机、路由器和防火墙
## 框式设备
- 主控板 MPU：提供整个系统控制和管理平面
- 交换网板 SFU：系统的数据平台
- 接口板 LPU：提供不同类型 光口、电口，不同速率的接入 接口
	- 光进铜退
- 接口板和主控板通过交换网板完成通信
## 盒式设备
- 各个业务模块不是独立的硬件模式，集成在一个框内
## 逻辑架构
- 控制管理平面，路由
	- 主控板和接口板的管理单元组成
- 转发平面，数据
	- 交换网面板和接口板组成 FPE 转发引擎
- 监控平面
	- 主控版和接口版的监控单元构成
## 设备处理
- 业务报文
- 协议报文
# IP路由基础
## 概述
### 收到IP报文，目的地址匹配
- 如果匹配到了，出接口或下一跳IP地址转发
- 如果没有匹配到，丢弃
### 路由表 RIB
- 本地核心路由表，全局路由表
- 协议路由表
- 路由器的控制平面
### 转发表 FIB
- 选择一个掩码最长的 FIB 表项转发报文
- 路由器的数据平面
## 路由来源
### 直连路由
- 直接连接所在的网段的路由，设备自动生成
### 静态路由
- 网管手工配置的路由
### 动态路由
#### 通过动态路由协议学习到的路由
#### IGP 内部网关协议
- 距离矢量协议
	- RIPv1，RIPv2，100
- 链路状态协议
	- OSPF 10 ，150
	- IS-IS 中间系统到中间系统，15
#### EGP外部网关协议
- BGP，255
## 路由迭代
计算出一个直连的下一跳

## 路由引入
-  从一种路由协议发布到另一种路由协议操作 import-routr
- 具有方向性
- 注意点
	- 路由优先级
	- 路由度量值
	- 路由回灌
# OSPF基础
## 动态路由协议分类
- 距离矢量协议：RIP
- 链路状态协议：OSPF IS-IS
## RIP 协议
### 基本知识
- 优先级100，度量值 跳数，端口号UDP 520，RIPv2 组播 224.0.0.9
- 计时器：更新30s，老化180s，垃圾收集120s
- 报文：request，response
- 防环：水平分割，毒性反转，最大跳数，触发更新
### 协议配置
```
rip
version 2
net 1.0.0.0 A类地址
net 172.16.0.0 B类地址
net 192.168.12.0 C类地址
```
### 查看3张表
邻居表
```
dis rip 1 neighbor
```
数据库
```
dis rip 1 database
```
路由表
```
dis rip route
```
## OSPF HCIA知识
- 优先级 10，150 
- 度量值 cost
- 组播 224.0.0.5 224.0.06
- 2种认证 接口，区域（simple，md5）
- 3张表
- 4种网络类型
- 5种报文
- 7种状态（不包含attempt）
- hello，dead interval，cost修改，DR选举，router-id选举，silent-interface
- OSPF建立过程
	- 建立邻居关系
	- 交互LSA同步的LSDB
	- spf 优选路径 best
	- 生成路由表项加载到路由表

# OSPF 介绍
## OSPF基础术语
### router-id
- 唯一标识一台运行OSPF协议的路由器，不能相同
- 一个32位无符号整数
- 选举规则
	- 推荐：手工配置ospf router-id 1.1.1.1
	- loopback 接口最大的IP地址
	- 物理接口中最大的IP地址
	- 0.0.0.0
### 区域 area
- 标识OSPF 区域，点分十进制
- 从逻辑上把设备分成不同的组，每个组用区域ID来标识

### 度量值 Cost
- 累计cost 为开销值
- 计算公示：参考带宽 100M/接口实际带宽
- 修改cost：1.接口cost2.参考带宽
## 三大表项
### OSPF 邻居表
```
dis ospf peer 
```
### LSDB
```
dis ospf lsdb
```
### OSPF 路由表
```
dis ospf routing
```

## OSPF报文
### Hello 报文
- 发现和维护邻居关系
### DD
- 交互链路状态摘要
### LSR
- 请求特定的 LSA
### LSU
- 发送详细的 LSA
### LSAck
- 确认 LSA
## OSPF工作过程
- Hello 报文发现直连链路上的邻居
- 协商主从 master / slave
- 相互交互各在的 LSDB（摘要）
- 更新 LSA，同步的 LSDB
- 计算路由
## 建立邻居关系

![[Pasted image 20240425204811.png]]
### Down
- 初始状态，没有从邻居收到任何消息
### lnit
- 从邻居收到了hello，但自己的 router-id 不在所收到的 hello 报文中
### 2-way
- 从邻居收到了hello，自己的router-id 存在于 hello 报文的邻居列表中
- DR选举（MA网络）
### Exstart
- 路由器开始向邻居发送DD报文（不包含摘要报文）
- i 第一个报文  M 后面还有更多报文  MS 为啥Master
- router-id 大的为 master
### Exchange
- 发送包含摘要消息的DD 报文
### Loading
- 互相发送 LSR，LSU，LSAck 报文
### Full
- 路由器已经完成了邻居的 LSDB 同步
## DR 指定路由器
### 负责在 MA 网络建立和维护邻接关系，同步 LSA
### 选举
- 接口 DR 优先级，越大越优先，默认为  1 ，0放弃
- router-id 越大越优
- DR 不抢占，基于接口
## OSPF 网络类型
### P2P 
- 10秒Hello    无DR
### broadcast
- 10秒Hello    DR
### NBMA
- 30秒Hello    DR
### P2MP
- 30秒Hello    无DR

# OSPF 路由计算
## 邻居建立不了的原因
- hello dead 间隔 10 ， 30
- 网络类型要相同
- 区域号 area 要一致
- 认证要相同
- router-id 要不同
- mtu 要相同， ospf mtu-enable
- MA 的子网掩码要相同
- MA 无DR
- 接口未设置 silent-interface
- 未工标识位





































