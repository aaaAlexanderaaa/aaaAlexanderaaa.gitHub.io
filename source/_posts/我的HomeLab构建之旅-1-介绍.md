---
title: 我的HomeLab构建之旅(1)-介绍篇
date: 2024-11-03 09:22:05
updated: 2024-11-03 09:22:05
tags:
- tools
- homelab
- docker
categories:
- ['折腾','homelab']
keywords:
---
# 我的HomeLab构建之旅


## 0x00 Preface

截至2024.11.3，我的HomeLab架构已经基本稳定，但是家庭组网仍在研究和纠结。所以本文也只能分享用了哪些工具、怎么用的、怎么选型的、工具之间的关系，至于工具搭建、踩坑指南，还要后面一一补足。



## 0x01 background

由于今年才养成记日记的习惯，而新冠之后记忆力有明显下降，促使我搭建Homelab的契机和背景已经不可详考了。但或许我能根据我的购物记录和购物车回想一二。

### &#x20;从朝南卧室到墨水屏开发

一切还要从朝南的超大的采光巨好的卧室说起……

因为卧室空间太大，而客厅没有空调，所以我把卧室的部分空间改造成了工作空间，放了个160\*80的电脑桌，加上显示器和工学院，非常完美。

但是，采光太好了，冬季的早晨阳光斜照，以至于我的电脑桌很大一部分要沐浴在阳光下，这对电子设备来说是不健康的。

然后我买了个屏风，调换桌子方向朝向窗台，用屏风挡住桌子，大概就是下面这样。再一次完美地解决了白天采光太好晒设备、没有光照又浪费的困境。

![](assets/6qBza4qejC1I9oS6r-oHlbISJ95TtoOoztHJlCfKJFA=.png)

记得是年初刚结束元旦的时候，我给自己创造需求了，既然电脑桌有背板（屏风），这屏风摆着就只是为了遮光是不是有点浪费？或许该整个洞洞板在后面把空间利用起来？接着折腾着给屏风上加了两块洞洞板。

然后发现我的东西还是有点少了，洞洞板装上了也没啥可以摆的。

又继续给自己创造需求了，或许可以加一个备忘录摆件架在洞洞板上，可以显眼地提醒自己todolist？

这个需求就复杂了，首先排除手写，要电子的；能远程上传的，得物联网；没有亮度，不然晚上影响睡眠，得墨水屏；不能小了，不然看不清。

这一来二去就变成了需要一个大墨水屏，看了下市场价格，决定自己搞裸屏开发，也不需要高刷，可以用彩色屏。

### &#x20;从香橙派到n100

但是我全系都是mac m系列芯片，没了解到有什么生态，所以还得买个开发版。最开始没搞清楚，在树莓派和香橙派之间选择了香橙派5b，心想这能有什么差别，肯定挑性价比高的。然后装了ubuntu和开发环境之后，发现可以开发但是它接线和树莓派不一样，怼了半天都搞不好，可是已经不能退了。

墨水屏最后买了微雪官方的esp32板子宝宝式编程给它起起来了。但是orange pi就这么空着。

于是开始整点小工具托管什么的，终于开始了我的homelab搭建之路。

中间发现这玩意儿是arm架构的很多docker都跑不了，但是也能将就用。

折腾到五月的时候这和香橙派宕机了一次起不来，给sd卡烧固件，通过sd卡启动把数据恢复之后。索性买了个n100 4c16g把数据倒腾上去，作为homelab的主力了。

此后一路发展和调整，终于形成了现在的homelab架构。



## 0x02 Overview

![](assets/AVFYx1Bjot9_ZuFOA0vnPm45VZrDPhSNpk3Y0cWbTjI=.png)



### Base env

从左到右依次是

* DNS：修改/etc/resolv.conf 指向我的dnsmasq，其后是dnscrypt-proxy
* [firewall-cmd](https://firewalld.org/)：网络的基本隔离（docker环境隔离需要额外配置）
* [tailscale](https://tailscale.com/)：网络穿透，组网
* Docker：不必多说
* Git：不必多说
* Python：不必多说
* venv：Python 依赖隔离，不必多说
* pip：Python 包管理，不必多说



### Docker App

#### Support

底下三个是

* [Nginx proxy manager](https://github.com/NginxProxyManager/nginx-proxy-manager)：非常方便地帮助你管理nginx配置和[certbot](https://certbot.eff.org/)证书更新
* [Portainer](https://github.com/portainer/portainer)：非常方便地帮你管理容器，包括远端容器
* [Sync thing](https://github.com/syncthing/syncthing)：数据同步，用于将数据备份更新到其他机器（香橙派出了一次岔子之后怕得要死）



上面三个是

* [Rustdesk server](https://github.com/rustdesk/rustdesk-server)：rustdesk的中继服务器
* [Bark](https://github.com/Finb/Bark)：非常方便的IOS通知推送，比ntfy好使（ntfy在IOS上不弹框）
* [Glances](https://github.com/nicolargo/glances)：监控





#### Docker Apps with features

下面四个大的是

* [Home assistant](https://github.com/home-assistant/core)：智能家居管理，我主要是用它把小米桥接到Homekit
* [NextCloud](https://github.com/nextcloud/server)：一个功能很全的东西，数据存储、在线编辑、日历、看板、账单跟踪啥的。我用它主要是看板和日历跟踪，偶尔放点小文件
* [Tiny Tiny RSS](https://github.com/HenryQW/Awesome-TTRSS)：RSS管理器和阅读器，一直在用，但是最近换到了follow
* [TriliumNext](https://github.com/TriliumNext/Notes)：笔记软件，日记笔记啥的都放上面，是[trilium](https://github.com/zadam/trilium/issues/4620)的重构+优化版本



上面的都是些小工具

* [Wallabag](https://github.com/wallabag/wallabag)：稍后读
* [alist](https://github.com/AlistGo/alist)：类似nextcloud但是只作存储，我用来做webdav存singleFile 的文件
* [singleFile2trilium](https://github.com/nil0x42/singlefile2trilium)：把这个py打包成了一个小容器，把singleFile的HTML存储到trilium里面去
* [json crack](https://github.com/AykutSarac/jsoncrack.com)：一个纯前端的json在线编辑和可视化，改了点代码去除了限制然后打包的docker，在考虑换成[json4u](https://github.com/loggerhead/json4u)但是那个项目还不成熟（docker起不来）
* [one-API](https://github.com/songquanpeng/one-api)：AI的API管理
* [dashy](https://github.com/Lissy93/dashy)：dashboard，把我所有这些东西放在一个dashboard里面访问
* [pdfx](https://github.com/Stirling-Tools/Stirling-PDF)：pdf工具包，其实用不太到了，但是放着也没撤掉
* [excalidraw](https://github.com/excalidraw/excalidraw)：在线编辑，试了用[excalidraw-collaboration](https://github.com/alswl/excalidraw-collaboration)起的可以存储文件和共同编辑的room（实际上感觉不如纯前端、文件导下来玩），有一定的数据丢失，索性就还是用纯前端了
* [dbgate](https://hub.docker.com/r/dbgate/dbgate)：web端的数据库客户端，挺好用的





#### Unused Docker App

* [omnivore](https://github.com/omnivore-app/omnivore)：和Wallabag一样，但是docker环境配不好，就不玩了
* [AFFiNE](https://github.com/toeverything/AFFiNE)：被trilium替代了，现在在homelab以外起了一个作为写作平台，还挺好
* [astral](https://github.com/astralapp/astral)：github stars 管理，用了很久但是因为一个很奇怪的bug崩了，作者也不维护了，就弃用转用[gitstars](https://github.com/cfour-hi/gitstars)了（纯前端搭在vecal上）
* [planka](https://github.com/plankanban/planka)：看板，能用但是UI不优雅，现在用的nextcloud的deck功能
* [maybe](https://github.com/maybe-finance/maybe)：财务管理，UI挺好的，但是，试用的时候还是开发状态，功能还不算好使，不知道现在咋样，现在用的nextcloud的cospend做的订阅管理
* [gpt4free](https://github.com/xtekky/gpt4free)：项目是好项目，但是好多接口都超出quota，找接口这个过程带来的折磨远超出白嫖的快乐，我还是选择了[GPT\_API\_Free](https://github.com/chatanywhere/GPT_API_free)
* [focalboard](https://github.com/mattermost-community/focalboard)：看板，能用且优雅，但是`This repository is currently not maintained` 
* [ntfy](https://github.com/binwiederhier/ntfy)：跨端通知，但是试用的时候，IOS根本不弹通知，还得打开APP等它自己加载，被Bark替代了





### Services

* [Rustdesk](https://github.com/rustdesk/rustdesk)：Rustdesk的app，装在ubuntu上，
* [Electerm-sync-server](https://github.com/electerm/electerm-sync-server-python)：electerm（一个开源终端）配置同步的
* [n8n](https://github.com/n8n-io/n8n)：低代码自动化工具，我在上面的自动化包括 定时备份、订阅到期监控、glances挂了就重启


## 0x03 Reflections


现在让我来回顾过去这大半年做的这么些折腾，包括还有很多没列出的，试用过但是淘汰、弃用了的工具，如今一一列下来还颇有一种成就感。

关于不在homelab上的工具、homelab中的各种工具的选型与搭建，之后再来分享
