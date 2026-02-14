# Virtuals 实时监控机器人（v2.0.0）

## 功能概览
- 监听 Base 链 `Transfer` 日志，解析打新/内盘买入事件
- 持久化存储：SQLite + JSONL
- 指标输出：
  - 分钟消耗（SpentV）
  - 大户榜
  - 我的钱包持仓
  - 交易录入延迟
- 提供 Web UI 与 API

## 运行模式
支持 4 种角色：
- `--role writer`：API/UI + 主库写入
- `--role realtime`：实时 WS 监听与解析
- `--role backfill`：自动/手动回扫与解析
- `--role all`：单进程兼容模式

## 安装
```bash
cd virtual
python -m pip install -r requirements.txt
```

## 配置
1. 复制 `config.example.json` 为 `config.json`
2. 按需填写：
- `WS_RPC_URL`
- `HTTP_RPC_URL`
- `BACKFILL_HTTP_RPC_URL`（建议单独给回扫）
- `LAUNCH_CONFIGS`
- `MY_WALLETS`
- `VIRTUAL_USDC_PAIR_ADDR`（链上价格模式时）
- `EVENT_BUS_SQLITE_PATH`（三进程事件总线库）
- `RECEIPT_WORKERS_REALTIME`（可选，实时进程回执并发）
- `RECEIPT_WORKERS_BACKFILL`（可选，回扫进程回执并发）

说明：
- 若不配置 `RECEIPT_WORKERS_REALTIME`/`RECEIPT_WORKERS_BACKFILL`，将回退到 `RECEIPT_WORKERS`。
- `--role all` 单进程模式仍使用 `RECEIPT_WORKERS`。

## 启动
### 三进程（推荐）
```powershell
cd C:\Users\hyc\Desktop\Codex\virtual
python virtuals_bot.py --config .\config.json --role writer
python virtuals_bot.py --config .\config.json --role realtime
python virtuals_bot.py --config .\config.json --role backfill
```

### 一键启动/停止
```powershell
.\start_3roles.ps1
.\stop_3roles.ps1
```

### 单进程兼容
```powershell
python virtuals_bot.py --config .\config.json --role all
```

## 访问地址
- Dashboard: `http://127.0.0.1:8080/`
- Health: `http://127.0.0.1:8080/health`

## v2.0.0 文档
- 三进程运行说明：`RELEASE_v2.0.0_三进程运行说明.md`
- 更新说明：`RELEASE_v2.0.0_更新说明.md`

## 注意事项
- 不签名、不发交易，仅做链上读与分析。
- 若页面未更新，先确认旧进程未占用 8080 端口。
- `config.json` 默认不提交到 Git（避免泄露私密配置）。
