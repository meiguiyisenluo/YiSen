---
title: 查看Windows系统的激活密钥和密钥类型
date: 2024-11-18 11:17:33
tags:
---

查看本机的密钥
```
wmic path SoftwareLicensingService get OA3xOriginalProductKey
```

查看激活密钥类型
```
slmgr.vbs /dli
```
