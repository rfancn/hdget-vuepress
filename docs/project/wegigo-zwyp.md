---
title: Wegigo系统设计思路
draft: false
date: 2018-03-27
categories:
- 项目
---


#### 场景一: 用户浏览商城

                                                                                    canal sync
    user browse -> Service Load Balance -> wegigo servers <------> redis cluster <---------------> mysql

用户浏览的数据都是从后台mysql同步到redis的数据, 数据从mysql数据库同步到redis允许一定延迟

任务：

- 调通mysql的数据到redis的同步
- redis中数据存储的格式

#### 场景二: 用户操作购物车

    user add/edit cart -> Service Load Balance -> wegigo servers <-------> redis cluster

用户购物车的数据全保存在redis中, 用户购物车的数据添加删除和编辑都在redis一层实现

任务：

- 需要把shop++中购物车逻辑实现到wegigo server中


#### 场景三: 用户提交订单

    user submit order -> Service Load Balance -> wegigo servers <---> redis cluster
                                                       |
                                                  mysql cluster

用户提交订单, wegigo server从redis中提取数据并调用shop++的订单提交业务逻辑来写入mysql数据库

任务：

- 修改shop++的订单提交业务逻辑来适配wegigo server


#### 任务分解

- 修改shop++代码适配团长业务逻辑
- 实现购物车在redis中的业务逻辑
- 实现shop++和微信小程序适配

- 实现mysql到redis数据的同步
- 设计mysql到redis数据的映射关系




- 评估服务器数量和性能
    - 一期：20w用户+2万订单/天, 并发2000
        - redis + rabbitmq + mysql

        - redis: 每天的订单10kx2w=200M 购物车10kx5w=500M，商品详情50kx100=5M，大概2G(1650/3年)

        - 评估公网带宽: 50K每次请求， 并发2000， 10M
        - 评估存储容量: 假设20万条100M文件大小，估计一年最少5G, 预留20G, 做10年的考虑，数据库200G空间(6000/3年)
            - 会员20w
            - 商品100条/天 * 365 = 4w * 10 = 40w
            - 订单20000条/天 * 365 = 730w
            - 其他：200w

    - 二期：50w用户+5万订单/天, 并发5000s
        - SLB + redis cluster + rabbitmq cluster + mysql cluster
    

- 压力测试和问题解决


#### 问题

- Cannot assign requested address 

    > echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
    > echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle

- Error 1040: Too many connections

    > mysql> set global max_connections = 500;

- canal cannot authenticate while connection to mysql

    > mysql> use mysql 
    > mysql> delete from user where user=''; 
    > mysql> flush privileges;

参考项目:

- [阿里巴巴canal](https://github.com/alibaba/canal)
- [canal2nosql](https://github.com/liukelin/canal_mysql_nosql_sync)

