# Secs

## 0.写在前面

​	最近学习内容繁杂，不再单一学习，索性开个新仓库记录下一些各种各样的片段学习，没什么逻辑性方便自己查


## 杂七杂八的Trick

- LD_PRELOAD相关
  - [Linux Preload Hook原理与实践](https://github.com/Y4tacker/Secs/blob/main/%E6%9D%82%E4%B8%83%E6%9D%82%E5%85%AB%E7%9A%84Trick/Linux%20Preload%20Hook%E5%8E%9F%E7%90%86%E4%B8%8E%E5%AE%9E%E8%B7%B5/index.md)
  - [linux ld.so.preload机制解析](https://zhuanlan.zhihu.com/p/575574549)
  
- 提权相关
  - [简谈SUID提权](https://zhuanlan.zhihu.com/p/373352345)
  - [Linux提权姿势二：利用NFS提权](https://cloud.tencent.com/developer/inventory/6764/article/1708369)
  - [sudo配置文件/etc/sudoers详解及实战用法](https://blog.csdn.net/field_yang/article/details/51547804)
  - [最全Potato家族提权总结](https://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247490866&idx=1&sn=00fda0a83a521ab4773e024f627bafeb&chksm=cea8f672f9df7f64e45acb7902297f51aec88edb54110fde35d9dde60f50664365af9b107981&mpshare=1&scene=1&srcid=1207CVGEd4PgMadxWUFK2xUB&sharer_sharetime=1670373132287&sharer_shareid=52185258fb750059aa42ee7e5101e6f6#rd)
  - [如何在只能控制命令不能控制参数情况下实现RCE(windows)](https://github.com/Y4tacker/Secs/blob/main/%E6%9D%82%E4%B8%83%E6%9D%82%E5%85%AB%E7%9A%84Trick/smb/%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E5%8F%AF%E6%8E%A7%E5%88%B6%E5%91%BD%E4%BB%A4/index.md)
  
- SSRF
  - [php-curl的一个案例](https://github.com/Y4tacker/Secs/blob/main/%E6%9D%82%E4%B8%83%E6%9D%82%E5%85%AB%E7%9A%84Trick/SSRF/php_curl/index.md)
  - [如何在侦查阶段快速发现SSRF](https://mp.weixin.qq.com/s?__biz=MjM5Mzc4MzUzMQ==&mid=2650255830&idx=1&sn=b5a7764d9994aa930dfa8aed1d93879a&chksm=be92da5289e553446f9a49a08e5dbfcb890b697c83144e6bbb2e8c2590515dc332aac2cb6567#rd)
  
- SQLI
  
  - [当addslashes遇到pre_replace](https://github.com/Y4tacker/Secs/blob/main/%E6%9D%82%E4%B8%83%E6%9D%82%E5%85%AB%E7%9A%84Trick/SQLI/addslashes%E4%B8%8Epreg_replace/index.md)
  
- 信息收集相关
  - [【JSRC小课堂】Web安全专题（一）](https://mp.weixin.qq.com/s?__biz=MjM5OTk2MTMxOQ==&mid=2727832089&idx=1&sn=fb15a15bad60a42fe8ae11345a0e328c&chksm=8050bd91b72734877045c2b1e5437a81c09e4e21fa8e43db09dd90a7e14e3f45cec34850cc14&scene=21#wechat_redirect)
  
- 🎣？
  

## 靶场学习

- Vulnhub
  - [Evilbox](https://github.com/Y4tacker/Secs/tree/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/EvilboxOne)
  - [Node](https://github.com/Y4tacker/Secs/blob/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/Node/node.md)
  - [Raven](https://github.com/Y4tacker/Secs/blob/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/Raven/raven.md)
  - [Hackme1](https://github.com/Y4tacker/Secs/blob/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/Hackme1/index.md)
  - [Hackme2](https://github.com/Y4tacker/Secs/blob/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/Hackme2/index.md)
  - [LinESC](https://github.com/Y4tacker/Secs/blob/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/LinESC/index.md)
  - [Callme](https://github.com/Y4tacker/Secs/blob/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/Callme/index.md)
  - [School](https://github.com/Y4tacker/Secs/blob/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/School/index.md)
  - [Ragnar-lothbrok](https://github.com/Y4tacker/Secs/blob/main/%E9%9D%B6%E5%9C%BA%E5%AD%A6%E4%B9%A0/Vulnhub/Ragnar-lothbrok/index.md)

## 好文分享
- [【JSRC小课堂】Web安全专题（四）SRC漏洞挖掘技术之命令执行漏洞](https://mp.weixin.qq.com/s/9cXGR7tlN_OAP-dMyZyHEw)
- [一次从jmx到rce](https://mp.weixin.qq.com/s?__biz=MzIwMzIyMjYzNA==&mid=2247506824&idx=1&sn=1bff6060290c0fdb7fe059cff2c61153&chksm=96d0208da1a7a99b6e61c8e3c332d324c0296bbccf1163cb8a10760e57cd17e150cb23a0e36a&mpshare=1&scene=1&srcid=1220PA2K5MY7dM3gWTr06z4r&sharer_sharetime=1671532238935&sharer_shareid=19374164c9d8647c6159e09a97bb1208#rd)
  

## 未授权访问
- [二十八种未授权访问漏洞合集](https://mp.weixin.qq.com/s?__biz=MzI1NzI5NDM4Mw==&mid=2247494207&idx=1&sn=0e834aa50ee9b3d7d87033b02050349c&chksm=ea1b037fdd6c8a69eabdb991af32e15e8a50c32ba838e1b970af5a988921c55f49970c470893#rd)

## 免杀
- [GoPass系列免杀基础（一）](https://mp.weixin.qq.com/s/WzRZs9uieqrkdp7JuwhtPw)

## API安全
- [API安全学习笔记（写的很杂有很多不太相关的东西，自己挑重点看）](https://xz.aliyun.com/t/11977)
- [玩转graphQL(讲了一些和graphql相关的安全问题)](https://mp.weixin.qq.com/s/gp2jGrLPllsh5xn7vn9BwQ)

## 论文学习

- 机器学习为何难以应用于真实的网络入侵检测系统
  - [机器学习为何难以应用于真实的网络入侵检测系统？（上）](https://github.com/Y4tacker/Secs/blob/main/%E8%AE%BA%E6%96%87%E5%AD%A6%E4%B9%A0/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E4%B8%BA%E4%BD%95%E9%9A%BE%E4%BB%A5%E5%BA%94%E7%94%A8%E4%BA%8E%E7%9C%9F%E5%AE%9E%E7%9A%84%E7%BD%91%E7%BB%9C%E5%85%A5%E4%BE%B5%E6%A3%80%E6%B5%8B%E7%B3%BB%E7%BB%9F%EF%BC%9F%EF%BC%88%E4%B8%8A%EF%BC%89/index.md)
  - [机器学习为何难以应用于真实的网络入侵检测系统？（下）](https://mp.weixin.qq.com/s?__biz=MzA4ODYzMjU0NQ==&mid=2652311846&idx=1&sn=5d93c068dab470679e172715ca12ebb1&chksm=8bc48ca8bcb305be5e133b9d644471a7cfb619b5601befa0e5b9e3895894354c4ee415bf99e4&mpshare=1&scene=23&srcid=111285yJ4ALkxxhNslt7IQY4&sharer_sharetime=1668237367354&sharer_shareid=58505d0e4294aa4fa3fc97f46db18590%23rd)



## 其他

- [PayloadsAllTheThings(好用的Payload仓库)](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [Galaxy-Bugbounty-Checklist](https://github.com/0xmaximus/Galaxy-Bugbounty-Checklist)
- Code Review
  - [vulnerable-code-snippets](https://github.com/yeswehack/vulnerable-code-snippets)
- PHP源码调试
  - [PHP源码调试ARM MAC](https://j7ur8.github.io/WebBook/PHP/M1_VSCODE%E8%B0%83%E8%AF%95php%E6%BA%90%E7%A0%81.html)
  - [Docker远程调试PHP源码](https://github.com/rust17/gdb-in-docker)
