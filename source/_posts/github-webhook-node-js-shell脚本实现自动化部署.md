---
title: github webhook + node.js + shell脚本实现自动化部署
date: 2024-04-08 15:39:03
tags:
---
## 处理GitHub webhook请求
``` js
// 设置 GIT_SSH_COMMAND 环境变量
process.env["GIT_SSH_COMMAND"] = "ssh -i /root/.ssh/id_rsa";

app.post("/webhook", upload.array(), (req, res) => {
  // 一些信息
  const event = req.headers["x-github-event"];
  const deliveryId = req.headers["x-github-delivery"];
  const payload = JSON.parse(req.body.payload);
  const repository = payload.repository;
  const repoName = repository.name;
  const branch = payload.ref.split("/").pop();

  console.log(`Received ${event} event with delivery id ${deliveryId}`);
  console.log(`From repository ${repoName} and branch ${branch}`);

  // 执行构建脚本
  exec(`/path/to/buildMission.sh`, {}, (error, stdout, stderr) => {
    if (error) {
      console.error("error:", error);
      return;
    }
    console.log("stdout: " + stdout);
    console.log("stderr: " + stderr);
  });
  res.sendStatus(200);
});
```

## buildMission.sh
```
#!/bin/bash
set -e

git pull
npm run build
mv build /var/www/html/xxx
```

## 最后
体验过后依旧存在问题，想把express服务用docker跑，但是express exec执行宿主机的shell脚本有点麻烦。

目前已经换为Jenkins。