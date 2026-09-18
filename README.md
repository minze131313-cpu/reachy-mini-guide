# Reachy Mini 社区应用图鉴

Hugging Face × Pollen Robotics × Seeed Studio 的开源桌面机器人 **Reachy Mini** 社区生态整理：
74 个精选应用、14 个代码项目、硬件规格、SDK 上手、中国区网络实战与生态观察。

## 线上地址

- <https://bordy.cn/reachy-mini/>

## 仓库结构

| 路径 | 说明 |
|---|---|
| `public/index.html` | 单页文章（部署时作为站点入口） |
| `public/img/` | 页面引用的全部图片（已压缩，约 3.4MB） |

## 部署

线上由 Hostinger VPS 上的 nginx 直接托管静态文件：仓库 `public/` 目录同步到
`/var/www/bordy.cn/reachy-mini/`。nginx 站点 `bordy.cn` 的配置为 `root /var/www/bordy.cn;`，
无额外 location 规则，因此新增子目录无需改动 nginx 配置。

## 数据说明

- 点赞数 / 星标数抓取于 2026-09-18，会随时间变化。
- 74 个应用是从 547 个带 `reachy_mini` 标签的 Space 中人工筛选的代表作，非全量清单。
- 硬件规格以官方数据手册为准；官方未公布续航小时数与销量，文中据此留白。
- 应用卡片图是 Hugging Face 自动生成的社交预览图，非应用截图。

## 主要来源

官方 SDK <https://github.com/pollen-robotics/reachy_mini> ·
官方文档 <https://huggingface.co/docs/reachy_mini/> ·
应用商店 <https://huggingface.co/spaces?q=reachy+mini> ·
发布博客 <https://huggingface.co/blog/reachy-mini>

> 内容为社区研究整理，非官方文档。
