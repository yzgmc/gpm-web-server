# gpm-web-server

Game Push Manager **网页服务端**。功能与 `gpm-server`（Windows 服务端）**完全一致**，仅部署位置与默认配置不同：本仓库面向 Web 服务器环境（Linux/容器/Nginx 反代等），默认监听 `8001`，`server_kind=web-server`，便于 web-admin 区分监测。

## 与 gpm-server 的关系

- **API 契约完全相同**：均来自 `gpm-common` 的协议定义
- **代码结构相同**：相同路由、相同存储方式
- **区别仅在配置**：`server_kind` / `server_name` / 默认端口

这样设计使得：
- 客户端可同时配置多个服务端地址，自动选择最近的可用源下载
- web-admin 能分别监测 Windows 服务端与 Web 服务端的运行状态
- 后续如需差异化能力，可各自演进而不影响对方

## 安装与运行

```bash
# 1. 先安装 gpm-common
pip install -e ../gpm-common

# 2. 安装依赖
pip install -r requirements.txt

# 3. 运行（默认 0.0.0.0:8001）
python run.py
```

环境变量与 `gpm-server` 一致：`GPM_HOST` / `GPM_PORT` / `GPM_DATA_DIR` / `GPM_SERVER_NAME` / `GPM_MAX_UPLOAD_MB` / `GPM_ADMIN_URL` / `GPM_PUBLIC_BASE_URL` / `GPM_REPORTER_INTERVAL` / `GPM_REPORTER_ID` / `GPM_LIGHT_DISK_YELLOW` / `GPM_LIGHT_DISK_RED` / `GPM_LIGHT_ERROR_RED`。

## Push 模型上报

与 `gpm-server` 一样，配置 `GPM_ADMIN_URL` 后会启动后台 Reporter 线程，定期向 web-admin 主动上报心跳（在线状态、整合包/模组数量、推送条目、状态指示灯等）。web-admin 不再轮询。灯色计算与阈值与 `gpm-server` 一致。

## API

与 `gpm-server` 完全一致，详见 `gpm-server` 仓库的 README。所有路由前缀 `/api/v1`：
`/sync` `/games` `/status` `/modpacks` `/mods`。

## 生产部署建议

- 使用 `gunicorn -k uvicorn.workers.UvicornWorker app.main:app` 多进程部署
- 通过 Nginx 反代，对 `/api/v1/modpacks/*/download` 等大文件路由开启 `proxy_buffering` 与限速
- 数据目录 `./data` 挂载到持久化卷
