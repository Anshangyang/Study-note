# 个人网站初站点docker部署

## 项目目标：

- [x] 个人网站初站点搭建。 ✅ 2026-08-06
- [x] nginx配置文件。 ✅ 2026-08-06
- [x] ssl证书。 ✅ 2026-08-06
- [x] docker镜像构建。 ✅ 2026-08-06
- [x] docker私有仓库管理。 ✅ 2026-08-06
- [x] docker部署。 ✅ 2026-08-06

## 技术栈：

| 组件         | 技术                   | 备注         |
| ---------- | -------------------- | ---------- |
| 个人网站       | HTML、CSS             | 初次尝试，比较简单  |
| docker | Dockerfile           | 主要联系docker |
| docker私有仓库 | registry+registry-ui | 因镜像包含私密信息  |
| 网站部署       | nginx镜像              | 比较简单       |

## 关键决策：

| 决策    | 选择           | 原因              |
| ----- | ------------ | --------------- |
| 部署技术栈 | docker+nginx | 简单的同时还能学习docker |

## 进度日志：

- 2026-08-06：个人网站完成。
- 2026-08-06：Dockerfile文件连同ssl证书、nginx配置完成。
- 2026-08-06：docker私有仓库及ui完成。
- 2026-08-06：部署完成。

## 任务清单：

- [x] 个人网站 ✅ 2026-08-06
- [x] Dockerfile文件 ✅ 2026-08-06
- [x] 私有仓库+nginx ✅ 2026-08-06

## 目录结构：

项目根/
├── website/ 
├── ssl/ 
├── nginx.conf
└── Dockerfile

## 重要代码：

```Dockerfile
FROM nginx:alpine

# 复制网站文件
COPY website/ /usr/share/nginx/html/

# 复制 SSL 证书
COPY ssl/ /etc/nginx/ssl/

# 复制自定义 Nginx 配置文件（覆盖默认配置）
COPY nginx.conf /etc/nginx/nginx.conf

# 暴露 443（HTTPS）和 80（HTTP）
EXPOSE 80 443
```

```nginx
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        listen 80;
        server_name 域名;

        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl;
        server_name 域名;

        ssl_certificate /etc/nginx/ssl/域名_bundle.crt;
        ssl_certificate_key /etc/nginx/ssl/域名.key;

        ssl_protocols TLSv1.2 TLSv1.3;

        root /usr/share/nginx/html;
        index index.html index.htm;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

```json:
{
  "insecure-registries": [
    "私有仓库所在服务器IP及端口"
  ],
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://mirror.ccs.tencentyun.com"
  ]
}
```

>推送镜像到私有仓库需先打上标签，命令为：docker tag 镜像名 私有仓库IP及端口/镜像名:版本号。

## 遇到的问题与解决方案：

### 问题1：nginx缺少events无法打开

- 现象：所创容器无法启动。
- 原因：自定义nginx配置文件缺少events。
- 解决：添加之后重新上传部署。

### 问题2：部署完成后CSS样式未生效

- 现象：网页只显示HTML，无任何样式。
- 原因：缺少include /etc/nginx/mime.types;。
- 解决：添加之后重新上传。

### 问题3：修改镜像后拉取部署网页依旧未修改

- 现象：新镜像拉取部署之后与旧镜像一样。
- 原因：浏览器缓存。
- 解决：强制浏览器刷新。

## 参考资料

- [git](https://git-scm.com/book/zh/v2)
- [docker](https://docs.docker.com/)

---

# *"日复一日，必有精进！"*