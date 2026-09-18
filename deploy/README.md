# 部署说明

线上地址：<https://bordy.cn/reachy-mini/>

## 拓扑

```
GitHub minze131313-cpu/reachy-mini-guide
        │  git clone --depth 1（部署容器内）
        ▼
Hostinger VPS 187.77.25.70 (KVM 2 · Ubuntu 24.04)
        │  容器把 public/ 同步到：
        └─ /var/www/spark-toys/reachy-mini/  ← 站点根目录下的独立目录
                       │
        nginx 站点 bordy.cn（配置来自 sites-available/spark-toys.conf）
          root /var/www/spark-toys;
          location / { try_files $uri $uri/ /index.html; }
          ↓
        https://bordy.cn/reachy-mini/
```

## 为什么不用改 nginx

`bordy.cn` 的站点根目录是 `/var/www/spark-toys`，并带 SPA 回退规则
`try_files $uri $uri/ /index.html`。**真实存在的文件或目录优先于回退**，所以把站点
放进 `root` 下的 `reachy-mini/` 子目录即可被直接命中，不需要新增 location，
也不需要 reload nginx。

> 曾经尝试过 `location ^~ /reachy-mini/ { alias /var/www/reachy-mini/current/; }`
> 指向软链目录。`nginx -t` 通过、reload 成功，但实际请求仍落到
> `root /var/www/spark-toys` 解析，图片全部 404。因此最终方案是**把真实文件同步到
> root 下**，不依赖 alias 与软链。

## 更新部署

```bash
# 1. 本地改内容后推仓库
git add -A && git commit -m "更新" && git push

# 2. 触发 VPS 侧同步（Hostinger API，token 在本机 ~/.hostinger_token）
python3 - <<'PY'
import json, os, urllib.request
token = open(os.path.expanduser('~/.hostinger_token')).read().strip()
vps = '1369858'
compose = open('deploy/compose.yml').read()
req = urllib.request.Request(
    f'https://developers.hostinger.com/api/vps/v1/virtual-machines/{vps}/docker',
    data=json.dumps({'project_name': 'reachy-mini-deploy', 'content': compose}).encode(),
    headers={'Authorization': f'Bearer {token}', 'content-type': 'application/json'},
    method='POST')
print(urllib.request.urlopen(req).read().decode()[:200])
PY

# 3. 验证
curl -s -o /dev/null -w '%{http_code}\n' https://bordy.cn/reachy-mini/
```

## 注意事项

- **compose 会展开 `$VAR`**：文件里的 shell 变量必须写成 `$$VAR`，否则会被替换成空字符串
  （本项目在这一点上踩过两次坑，症状是 `mkdir: can't create directory ''` 或变量静默变空）。
- 目标机 **SSH 不可用**（22 端口开放，但本机密钥未获授权），维护一律走
  Hostinger API + 一次性容器。
- 需要访问宿主文件时，容器要 `volumes: /:/host`；若还要操作 nginx，需再加
  `pid: host` + `privileged: true` + `cap_add: [SYS_ADMIN, SYS_PTRACE]`，
  然后 `apk add util-linux`，用
  `nsenter --target 1 --mount --uts --ipc --net --pid -- nginx -t` 校验、
  `nginx -s reload` 热载（**不要重启整机**）。
  注意容器内 `/host/usr/sbin/nginx` 因动态库路径问题无法直接执行，必须走 `nsenter`。
- 本部署**没有**修改 `spark-toys.conf`。若将来确实需要改，请先备份为
  `*.bak-<日期>`，并用 `nginx -t` 通过后才 reload。
- 缓存策略来自站点既有的 `location ~* \.(css|js|jpg|png|gif|webp|svg|ico)$`
  （7 天）。因此**替换同名图片后，访客可能最长 7 天看不到更新**；更新图片时建议改文件名。
