---
title: Docker上折腾了Nginx + AFFiNE + AI 作为写作平台
date: 2024-10-15 00:17:06
updated: 2024-10-15 00:17:06
tags:
- AFFiNE
- Blog
- Tools
- Writing
- AI Tools
- Docker
categories: 
- ['折腾','AFFiNE']
keywords:
---
# Docker上折腾了Nginx + AFFiNE + AI 作为写作平台

## 0x01 Background



这个软件我在4月份的时候尝试过，那个时候功能比较单一，对docker的支持不是很友好（我的homelab还基于arm，而它不支持），bug也挺多，自定义程度不高，用了一段时间发现了trilium，就放弃AFFiNE转用trilium了。

最近刷Github trending的时候看到了AFFiNE，发现加入了AI功能，主要是短短的一个月就从十几k变成了四十几k star，这让我很好奇它到底有什么样的魔力，就搭了一套环境配了AI、改了quota，打算拿来作为写一些presentation的文档工具。

当然，我自己的日常写作还是在trilium上。



## 0x02 Implementation

本次部署是考虑文档编辑可能涉及多人协同，没有部署在homelab，而是部署在vps上，当然网络环境和防火墙不在本次部署的考虑范围内

### 01 Dependencies

我的环境是 CentOS Stream release 9, x86\_64 架构，在公网的vps，网络畅通，2c 3G。AFFiNE使用起来完全不卡顿，不知道最低配置需要多少。

* [ ] linux x86主机
* [ ] docker
* [ ] git





### 02 Deployment



1. 域名解析，我通过cloudflare带 proxy的A记录走到了我的主机，做一个基本防护和访问加速 
2. 主机上配置了nginx proxy manager 做nginx 管理和证书签发，进81端口初始化nginx proxy manager

```
docker run -d --restart=always -p 80:80 -p 443:443 -p 81:81 \ 
-v /root/nginx_proxy_manager/data:/data \ 
-v /root/nginx_proxy_manager/letsencrypt:/etc/letsencrypt \ 
 docker.io/jc21/nginx-proxy-manager:latest
```

3. 初始化AFFiNE

```
git clone https://github.com/toeverything/AFFiNE.git --branch stable
cd AFFiNE/
vim ./.github/deployment/self-host/compose.yaml
#compose 这个文件没啥改的，但是建议改一下postgres的默认账号密码
docker compose -f ./.github/deployment/self-host/compose.yaml up -d

```

4. 顺利的话现在affine的纯demo版本已经起来了，如果可以正常访问3010端口的话，先配一下admin账号，然后去nginx-proxy-manager配置一下域名，指向3010端口
   1. 关于nginx代理时访问affine的后端的网址，我是通过 `docker network connect affine_network ngm` 打通网卡，让nginx走内部地址http://affine\_selfhosted:3010 访问，然后在主机上通过一些配置把docker 的端口给拦了（关于docker的网络隔离本文不赘述），避免不必要的端口暴露
   2. BTW，nginx-proxy-manager配置了cert之后（我是通过dns challenge配的，去cloudflare domain页面创建个token就可以了），通过CF 的 proxy可能会遇到多次重定向的问题，是因为CF通过http访问https端口导致的，在CF的ssl/tls 里配一下` `**`Current encryption mode:`**`Full `就可以了
5. 配置AFFiNE

```javascript
vim ~/.affine/self-host/config/.env
把AFFINE_SERVER_HOST配置为你的域名就可以了

vim ~/.affine/self-host/config/affine.js
//这个js的倒数几行是被注释的，取消注释配置你的AI接口
//截止2024.10.12，affine还没有自定义AImodel的能力，根据测试，应该是默认会调用gpt-4o模型
// /* Copilot Plugin */
AFFiNE.use('copilot', {
   openai: {
     baseURL:'https://host/v1',
//这是某个大佬在代码里加上的会采用这个配置项来指定URL，但是默认配置文件里没有
     apiKey: 'your-key'
   },
//   fal: {
//     apiKey: 'yourkey',
//   },
//   unsplashKey: 'your-key',
//   storage: {
//     provider: 'cloudflare-r2',
//     bucket: 'copilot',
//   }
 })


docker compose -f ./.github/deployment/self-host/compose.yaml restart 
```

6. 不出意外的话，现在你的affine已经工作正常了，但是你的AI功能仍然面临着每天10次的quota limit，不过幸好这个配置项是存在数据库里可以改的，接下来让我们修改它

```
docker exec -it affine_postgres /bin/bash
psql -U affine
\d
#你可以看到数据库的tables
#features 这个表是配置不同的plan的能力的
#user_features 这个表是配置不同user的plan的

UPDATE features
SET configs = jsonb_set(configs::jsonb, '{copilotActionLimit}', '1000'::jsonb, false)
WHERE id = 13;
#在我这个时候 user的默认plan id 是13，也就是free_plan_v4，对应的limit 是10次每天
#我直接修改了free plan而不是把我的plan改成pro或者其他的，就是懒得搞
.exit
exit

docker compose -f ./.github/deployment/self-host/compose.yaml restart
```

7. 不出意外的话，你现在已经不仅可以通过域名访问AFFiNE，使用AI能力，还基本不限次数了（AI API的次数取决于你自己的配置嗷，不想付费而是想用本地AI的，可以部署一个oneapi做gateway，我不需要所以不赘述了）

## 0x03 Reflections



加上AI，修了bug的AFFiNE感觉确实比年初好用很多，当然不排除这40+k stars 产生了一些心理作用。

当然，opening issues数量只有不到200个，也可以看得出来官方至少是有在高强度维护这个项目的。

AFFine的开发人员也说正在开发完全self host的AI的版本，不过我现在这个配置也能用，就没那么期待了，反而挺期待他们实现诸如table的普通功能什么的。

这是第一次通过直接改数据库配置来绕过quota，感觉蛮奇妙的，希望接下来的使用可以满足我对它的期待。





## 0x99 References



1. https://docs.affine.pro/docs/self-host-affine
2. https://github.com/toeverything/AFFiNE/discussions/7030
3. https://github.com/toeverything/AFFiNE/issues/6156#issuecomment-2084366898
4. https://github.com/NginxProxyManager/nginx-proxy-manager?tab=readme-ov-file#quick-setup
5. https://community.cloudflare.com/t/dns-err-too-many-redirects/584654/3
