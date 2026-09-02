---
title: 如何绑定域名到github pages
date: 2026-09-02 00:00:00
tags:
  - GitHub Pages
  - 域名
  - Cloudflare
---

教程以 `drshi.cn` 绑定 `sandyplus.github.io` 为例，新手可直接参考操作。

## 准备工作

- **已部署的 GitHub Pages 网站**：例如 `sandyplus.github.io`。
- **已购买的域名**：例如在腾讯云购买的 `drshi.cn`。
- **注册 Cloudflare 账号**：前往 Cloudflare 官网免费注册。

## 第一阶段：在 GitHub 声明自定义域名

在配置网络解析之前，必须先让 GitHub 知道你的域名。

1. 登录 GitHub，进入你的网站代码仓库。
2. 点击顶部导航栏最右侧的 **Settings**（齿轮图标）。
3. 在左侧菜单栏中找到并点击 **Pages**。
4. 页面向下滑动，找到 **Custom domain**（自定义域名）设置项。
5. 在空白输入框中填入你的主域名（例如 `drshi.cn`），点击 **Save**。
6. *注意：此时可能会出现黄色的 DNS 校验提示，且下方的 **`Enforce HTTPS`** 无法勾选，这是正常现象，请保持原样进入下一步。*

## 第二阶段：将域名接入 Cloudflare

利用 Cloudflare 的免费 CDN 解决 GitHub Pages 国内访问慢、易断连的问题。

1. 登录 [Cloudflare 控制台](https://dash.cloudflare.com/)，点击主页的**domains** --**添加域名**。
2. 输入你的域名 `drshi.cn`，点击 **继续**。
3. 页面向下滑动到底部，选择 **Free（免费）** 套餐，点击 **继续**。
4. **添加 DNS 记录**：在扫描结果页面，点击 **添加记录（Add record）**，添加以下两条关键记录：
   - **第一条（主域名）**：类型选 `CNAME`，名称填 `@`，目标填 `sandyplus.github.io`，确保代理状态为**橙色云朵（已代理）**，保存。
   - **第二条（www 前缀）**：类型选 `CNAME`，名称填 `www`，目标填 `sandyplus.github.io`，代理状态为**橙色云朵**，保存。
5. 点击页面底部的 **继续**。Cloudflare 会分配给你两个**名称服务器（Nameservers）**（例如 `etienova.ns.cloudflare.com` 等），请复制并保存这两个地址。

## 第三阶段：去腾讯云移交 DNS 解析权

将域名的控制权从腾讯云转交给 Cloudflare。

1. 登录 [腾讯云域名控制台](https://console.cloud.tencent.com/domain)。
2. 在“我的域名”列表中找到你的域名，点击右侧的 **管理**。
3. 在详情页找到 **DNS 服务器**（或“修改 DNS”），点击修改。
4. 选择 **非DNSPOD**，删掉原有的腾讯云服务器地址（如以 `dnspod.net` 结尾的地址）。
5. 将刚才在 Cloudflare 复制的两个名称服务器地址分别填入，点击保存（可能需要手机验证码或刷脸验证）。
6. 回到 Cloudflare 页面，点击底部的 **检查名称服务器**。等待 10~30 分钟，直到 Cloudflare 发送邮件通知，或控制台域名状态显示为带绿色/深灰色对勾的 **活动（Active）**。

## 第四阶段：终极排错与开启 HTTPS

当 Cloudflare 显示激活后，回到 GitHub Pages 设置页。如果你遇到红色的 `DNS check unsuccessful` 报错或 `Enforce HTTPS` 无法勾选，请使用以下“破局”操作：

1. **清除卡死的缓存**：在 GitHub Pages 设置的 Custom domain 下，点击红色的 **Remove** 按钮删掉你的域名。
2. **确认代理开启**：前往 Cloudflare 的 DNS 记录页面，确保 `@` 和 `www` 两条记录的代理状态都是开启的（**橙色云朵**）。注意，可能需要你等待10-30分钟。
3. **重新绑定**：回到 GitHub Pages 设置页，重新在 Custom domain 中输入 `drshi.cn` 并点击 **Save**。
4. **开启安全加密**：由于 Cloudflare 已经在前端套上了加密，此时 GitHub 会顺利通过检测。等待页面刷新后，直接勾选下方的 **Enforce HTTPS**。

至此，你在浏览器中输入域名即可秒开网站，且自带 HTTPS 安全小绿锁。
