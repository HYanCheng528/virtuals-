# V3.0.0 使用说明（从 0 到可用）

本文面向首次使用者，按步骤执行即可跑起来。

## 1. 你将得到什么
- 一个本地 Web 面板：`http://127.0.0.1:8080/`
- 实时 + 回扫双链路数据
- 大户榜、分钟消耗、我的钱包持仓、录入延迟视图

## 2. 前置准备
- 操作系统：Windows 10/11
- Python：3.10 及以上
- 网络：可访问 Base RPC
- 节点建议：
  - 节点 A（实时）：`WS + HTTP`
  - 节点 B（回扫）：`HTTP`

## 3. 下载与安装

### 3.1 下载仓库
- 方式 1：Git 克隆
```powershell
git clone <你的仓库地址>
cd virtual
```
- 方式 2：GitHub 下载 ZIP，解压后进入 `virtual` 文件夹

### 3.2 安装依赖
```powershell
python -m pip install -r requirements.txt
```

## 4. 配置文件

### 4.1 复制模板
```powershell
copy .\config.example.json .\config.json
```

### 4.2 必填项（先改这 3 个）
- `WS_RPC_URL`：改成你的 WS API
- `HTTP_RPC_URL`：改成你的 HTTP API
- `BACKFILL_HTTP_RPC_URL`：改成你的回扫 HTTP API

注意：仓库内不会提供真实 API Key，你必须填写自己的。

### 4.3 可选项（按需）
- `LAUNCH_CONFIGS`：项目列表
- `MY_WALLETS`：你要监控的钱包
- `API_PORT`：面板端口，默认 8080

## 5. 启动程序

### 5.1 一键启动（推荐）
```powershell
.\start_3roles.ps1
```

脚本会启动：
- writer
- realtime
- backfill

### 5.2 手动启动（用于排障）
分别开 3 个 PowerShell 窗口：
```powershell
python virtuals_bot.py --config .\config.json --role writer
python virtuals_bot.py --config .\config.json --role realtime
python virtuals_bot.py --config .\config.json --role backfill
```

## 6. 打开与验证

### 6.1 打开 UI
- 浏览器访问：`http://127.0.0.1:8080/`

### 6.2 检查健康状态
- 打开：`http://127.0.0.1:8080/health`
- 关注字段：
  - `ws_connected` 是否 `true`
  - `rpc_errors` 是否持续增长
  - `queueSize` 是否长期堆积

## 7. UI 使用流程（推荐）
1. 在页面顶部确认当前项目（name / internal_pool_addr）。
2. 若要补历史，选择 UTC+8 时间区间并点击区间回扫。
3. 看分钟消耗图是否开始有柱状数据。
4. 看大户榜和我的钱包是否开始滚动更新。
5. 如需新增监控钱包，在钱包区添加地址并保存。

## 8. 停止程序

### 8.1 一键停止
```powershell
.\stop_3roles.ps1
```

### 8.2 手动停止
- 在每个运行窗口按 `Ctrl + C`

## 9. 参数建议（你们可直接抄）

### 9.1 低延迟优先
- `CONFIRMATIONS = 0`
- `BACKFILL_CHUNK_BLOCKS = 5`
- `BACKFILL_INTERVAL_SEC = 5`
- `RECEIPT_WORKERS_REALTIME = 8`
- `RECEIPT_WORKERS_BACKFILL = 2`
- `DB_BATCH_SIZE = 1`
- `DB_FLUSH_MS = 150`

### 9.2 节点压力大时
- 先把 `BACKFILL_CHUNK_BLOCKS` 再降小
- 再把 `BACKFILL_INTERVAL_SEC` 拉大
- 必要时减少 `RECEIPT_WORKERS_BACKFILL`

## 10. 常见故障与处理

### 10.1 回扫失败：`eth_getLogs is limited`
- 原因：节点限制单次 logs 范围。
- 处理：降低 `BACKFILL_CHUNK_BLOCKS`。

### 10.2 没有新数据
- 检查 realtime 进程是否运行。
- 检查 `WS_RPC_URL` 是否可用。
- 检查内盘地址是否填错。

### 10.3 页面显示旧图标/旧标题
- 重启 writer。
- 浏览器 `Ctrl + F5` 强刷。

### 10.4 端口被占用
- 改 `API_PORT`（如 8081），重启 writer。

## 11. 安全与隐私
- 仓库不提交 `config.json`（已在 `.gitignore`）。
- 请勿在公开仓库提交真实 API Key。
- 钱包地址、内盘地址可按需要公开。

## 12. 升级建议
- 升级前先备份 `data/*.db`。
- 升级后先用 10 分钟观察 `health`。
- 无异常再进入正式使用。