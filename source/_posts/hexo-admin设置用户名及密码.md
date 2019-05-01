title: hexo-admin设置用户名及密码
author: Nico
tags:
  - Hexo
categories:
  - Hexo
date: 2019-05-01 16:13:00
---
## 安装hexo插件
##### 安装并使用hexo-admin
```
npm install --save hexo-admin
hexo server -d
open http://localhost:4000/admin/
```
#### 设置后台密码
修改站点配置文件`_config.yml`:
```
# admin
admin:
  username: ****
  password_hash: ZwrRbx0gZl8myLbI9/oA4T4TxgSxE.
  secret: *****
```
- username是用户名
- password_hash是密码的哈希映射值，由于不同版本的node.js的哈希算法是不一样的，所有用以下方法来产生有效的密码哈希值
```
$ node
> const bcrypt = require('bcrypt-nodejs')
> bcrypt.hashSync('your password secret here!')
//=> '$2a$10$8f0CO288aEgpb0BQk0mAEOIDwPS.s6nl703xL6PLTVzM.758x8xsi'
```
