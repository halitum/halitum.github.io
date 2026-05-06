---
title: "容器与宿主机交换数据方式"
date: 2026-03-25
description: "Docker 容器与宿主机之间交换数据的五种方式对比与选择建议"
tags: ["Docker", "容器"]
categories: ["实用技术"]
---

## 方式对比

| 方式 | 命令示例 | 适用场景 |
|------|---------|---------|
| **Volume 挂载** | `docker run -v /host/path:/container/path` | 持久化数据、实时双向共享文件 |
| **Named Volume** | `docker run -v myvolume:/container/path` | 多容器共享数据，由 Docker 管理存储位置 |
| **docker cp** | `docker cp container:/app/data.csv ./` | 临时取出/放入单个文件 |
| **暴露端口** | `docker run -p 8080:80` | 容器内有网络服务（Web/DB/API） |
| **docker exec** | `docker exec container cat /file > local.txt` | 临时读取容器内命令输出 |

## 选择建议

- **需要持久化 / 实时共享文件** → Volume 挂载（最常用）
- **多个容器共享同一份数据** → Named Volume
- **临时拷贝文件** → `docker cp`
- **访问容器内的网络服务** → 暴露端口
- **容器停止后数据消失** → 默认行为（不挂载 volume）

## 注意

暴露端口解决的是"网络层面的服务访问"，而非文件数据交换。需要拿到文件本身时应优先考虑 volume 挂载或 `docker cp`。
