# V-Pulse 盘面雷达（V3.0.0）

面向 Base 链的实时监控与回扫分析工具。支持三进程并行（`writer` / `realtime` / `backfill`），主打低延迟看盘、分钟消耗统计、大户榜、我的钱包持仓追踪。

## 1. 核心能力
- 实时监听：WebSocket 监听链上新块，快速捕获交易日志。
- 自动补漏：后台按区块窗口循环回扫，补齐 WS 漏单。
- 手动区间回扫：按 UTC+8 时间区间发起回扫任务。
- 数据持久化：SQLite 本地落库（无 Supabase 热路径）。
- 多项目管理：UI 支持项目切换、项目配置保存、删除。
- 钱包监控：支持在 UI 动态增删监控钱包，并保留历史。
- 可视化：分钟消耗柱状图、大户榜、我的钱包、事件录入延迟视图。

## 2. 三进程架构
- `writer`：唯一写库进程，提供 API + UI。
- `realtime`：WS 实时监听 + 回执解析，将事件写入事件总线。
- `backfill`：自动回扫 + 手动回扫任务执行，将事件写入事件总线。

说明：`realtime` 与 `backfill` 不直接写主库，统一由 `writer` 落库，减少并发写冲突。

## 3. 仓库文件（V3.0.0 最小发布集）
- `virtuals_bot.py`：主程序
- `dashboard.html`：前端 UI
- `favicon-vpulse.svg`：浏览器标签图标
- `config.example.json`：脱敏配置模板（API 已替换为占位符）
- `requirements.txt`：依赖
- `start_3roles.ps1`：一键启动三进程
- `stop_3roles.ps1`：一键停止三进程
- `RELEASE_v3.0.0_更新说明.md`：版本更新点
- `RELEASE_v3.0.0_使用说明.md`：详细使用教程
- `需求文档_v3.0.0.md`：完整需求说明（可给 Codex 生成同类程序）

## 4. 环境要求
- Windows 10/11（PowerShell）
- Python 3.10+
- 可用 Base RPC：
  - 1 个 WS（实时）
  - 1 个 HTTP（实时补充调用）
  - 1 个 HTTP（回扫，建议独立）

## 5. 快速启动（推荐）

### 5.1 安装依赖
```powershell
cd C:\Users\你的用户名\...\virtual
python -m pip install -r requirements.txt
```

### 5.2 准备配置
```powershell
copy .\config.example.json .\config.json
```

只需先改 3 项：
- `WS_RPC_URL`
- `HTTP_RPC_URL`
- `BACKFILL_HTTP_RPC_URL`

### 5.3 启动三进程
```powershell
.\start_3roles.ps1
```

### 5.4 打开 UI
- `http://127.0.0.1:8080/`

### 5.5 停止三进程
```powershell
.\stop_3roles.ps1
```

## 6. 手动启动（便于排障）
```powershell
python virtuals_bot.py --config .\config.json --role writer
python virtuals_bot.py --config .\config.json --role realtime
python virtuals_bot.py --config .\config.json --role backfill
```

## 7. 配置说明（重点参数）

### 链接参数
- `WS_RPC_URL`：实时链路 WS。
- `HTTP_RPC_URL`：实时链路 HTTP（回执、eth_call）。
- `BACKFILL_HTTP_RPC_URL`：回扫专用 HTTP。

### 项目与钱包
- `LAUNCH_CONFIGS`：项目列表（名称、内盘地址、fee/tax 地址等）。
- `MY_WALLETS`：我的钱包地址列表。

### 延迟/吞吐参数
- `CONFIRMATIONS`：回扫确认块数。`0` 更快，`1+` 更稳。
- `BACKFILL_CHUNK_BLOCKS`：单次回扫区块跨度。
- `BACKFILL_INTERVAL_SEC`：自动回扫轮询间隔。
- `RECEIPT_WORKERS_REALTIME`：实时解析并发。
- `RECEIPT_WORKERS_BACKFILL`：回扫解析并发。
- `DB_BATCH_SIZE`：writer 每批写入条数。
- `DB_FLUSH_MS`：writer 定时 flush 间隔。

## 8. 主要 API
- `GET /`：Dashboard 页面
- `GET /health`：健康状态
- `GET /meta`：项目、钱包、运行参数
- `GET /leaderboard?project=...`：大户榜
- `GET /minutes?project=...&from=...&to=...`：分钟消耗
- `GET /mywallets?project=...`：我的钱包
- `POST /scan-range`：手动区间回扫
- `POST /scan-jobs/{job_id}/cancel`：取消回扫任务

## 9. 常见问题

### Q1：UI 只有地球图标或旧标题
- 重启 `writer` 进程。
- 浏览器 `Ctrl + F5` 强刷。

### Q2：回扫失败，提示 `eth_getLogs is limited`
- 把 `BACKFILL_CHUNK_BLOCKS` 调小（例如 `10 -> 5`）。
- 回扫专用节点尽量独立。

### Q3：为什么没新数据
- 确认 `realtime` 进程在运行。
- 看 `/health` 是否 `ws_connected=true`。
- 检查项目内盘地址是否正确。

### Q4：新增钱包后没历史数据
- 默认只统计新增后录入的数据。
- 可使用“按项目 + 单钱包重算”功能补历史。

## 10. 发布文档
- `RELEASE_v3.0.0_更新说明.md`
- `RELEASE_v3.0.0_使用说明.md`
- `需求文档_v3.0.0.md`