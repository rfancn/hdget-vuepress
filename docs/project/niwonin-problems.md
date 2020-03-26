---
title: 你我您商城实现功能列表
draft: false
date: 2018-03-28
categories:
- 项目
---

#### (1) 非互斥数据的同步

主要依赖canal来实现主数据库到redis的同步

- 后台更新会员数据库记录的时候强制更新缓存，并更新perm:member:mtime

- 团长信息管理

    - 团长管理功能
    - 强制同步所有团长信息到缓存功能
    - 后台修改团长数据库的时候强制更新缓存，并更新perm:buddy:mtime

- 小区信息管理

    - 团长管理功能
    - 强制同步所有小区信息到缓存功能
    - 后台修改小区数据库的时候强制更新缓存，并更新perm:area:mtime

- 收货地址信息管理

    - 收货地址管理功能
    - 强制同步所有收货地址到缓存功能
    - 后台修改收货地址数据库的时候强制更新缓存，并更新perm:addr:mtime

- 商品分类管理同步

    - 强制同步所有商品分类到缓存功能
    - 后台修改商品分类数据库的时候强制更新缓存，并更新perm:catalogue:mtime

#### (2) 互斥数据的同步

锁定缓存中相应数据，获取并显示缓存数据到后台，后台数据库更新后并同步到缓存

- 商品库存的同步


#### 参考

- [阿里巴巴canal](https://github.com/alibaba/canal)

- [Slowly changing dimension](https://en.wikipedia.org/wiki/Slowly_changing_dimension)