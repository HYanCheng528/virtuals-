# Virtuals 实时扫描机器人（v1.1）

## 功能
- Base 链 `Transfer` 实时监听（WebSocket）。
- 基于 `txHash + receipt.logs` 做项目归因与买入解析。
- 输出单笔事件（JSONL）与 SQLite 持久化。
- 实时维护：
  - `mywallets` 累计成本与回本 FDV
  - 分钟级 `spentV` 聚合
  - 大户榜
- 内置接口：
  - `GET /`（Dashboard UI）
  - `GET /meta`
  - `GET /health`
  - `GET /mywallets`
  - `GET /mywallets/{addr}`
  - `GET /minutes?project=...&from=...&to=...`
  - `GET /leaderboard?project=...&top=N`

## 安装
```bash
cd virtual
python -m pip install -r requirements.txt
```

## 配置
1. 复制 `config.example.json` 为 `config.json`
2. 填写：
   - `WS_RPC_URL`
   - `HTTP_RPC_URL`
   - `BACKFILL_HTTP_RPC_URL`（可选，给自动/手动回扫独立 HTTP 节点）
   - `LAUNCH_CONFIGS`（多个项目）
   - `MY_WALLETS`
   - `VIRTUAL_USDC_PAIR_ADDR`（若开启链上价格）

## 启动
```bash
cd virtual
python virtuals_bot.py --config ./config.json
```

## 启动后查看
- Dashboard: `http://127.0.0.1:8080/`
- 健康检查: `http://127.0.0.1:8080/health`

## 输出位置
- SQLite: `SQLITE_PATH`
- JSONL: `JSONL_PATH`

## 注意
- v1.1 禁止 Supabase 配置（检测到会直接报错退出）。
- 仅链上只读分析，不签名、不发交易、不做 approve。
- 如果访问 `/` 仍是旧结果，通常是旧进程还占着 `8080`，先停止旧进程再重启。
