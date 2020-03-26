---
title: 你我您商城缓存机制实现
draft: false
date: 2018-03-28
categories:
- 项目
---

#### 常驻缓存

- 所有会员(100万大概100M): 
    - 数据：perm:member:uid -> json:info(会员表)
    - 强制同步机制: perm:member:mtime为空
    - 修改机制：后台会员数据库的时候强制更新缓存，并更新perm:member:mtime

- 所有团长: 
    - 相关数据：
        - perm:buddy:uid -> json:info(团长表)
    - 强制同步机制: perm:buddy:mtime为空
    - 修改机制：后台团长数据库的时候强制更新缓存，并更新perm:buddy:mtime

- 所有小区信息: perm:area:id -> json:info(小区信息)

    - 强制同步机制: perm:area:mtime为空
    - 修改机制：后台小区数据库修改的时候强制更新缓存，并更新perm:area:mtime

- 所有收货地址信息: perm:addr:id -> json:info(收货地址信息)

    - 强制同步机制: perm:addr:mtime为空
    - 修改机制：后台收货地址数据库修改的时候强制更新缓存，并更新perm:addr:mtime

- 所有商品分类: perm:catalogue:id -> (商品分类表)
    
    - 强制同步机制: perm:catalogue:mtime为空
    - 修改机制：后台商品分类数据库修改的时候强制更新缓存，并更新perm:catalogue:mtime

#### 周期缓存

每天临晨会清空并强制更新,后续按后台数据库修改监控binlog同步

- 客户只读数据+后台读写

    - 所有商品信息：cycle:goods:goods_id -> (商品表)
    
    - 所有上架商品分类集合zset：cycle:onshelf:[catid] -> 上架商品zset

 
- 客户读写数据+后台读写

    - 商品库存: cycle:goods:cat_id:remain -> (商品库存)


#### 临时缓存: 

每天临晨清空临时数据

- 购物车数据: temp:card:uid

#### 原理

1. 常驻缓存有一个强制同步机制，第一次redis启动时会尝试强制同步数据库数据到缓存
2. 后台数据库在更改常驻数据的时候会强制同步缓存
3. 

#### 预热缓存内容






#### 缓存预热

1. redis记录预热开始时间
2. 从数据库中读取缓存内容，并以缓存格式保存到redis中(每个key一个thread/routine)




- 第一次做一次warm up取当前数据主要采用mysql binlog解析的方式同步主数据库里面的数据

同步的内容:

- redis