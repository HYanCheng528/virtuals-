# V4.0.0 使用说明

## 1. 安装与启动
1. 安装依赖
```powershell
cd virtual
python -m pip install -r requirements.txt
```

2. 复制配置模板
```powershell
copy .\config.example.json .\config.json
```

3. 至少填写以下 RPC
- `WS_RPC_URL`
- `HTTP_RPC_URL`
- `BACKFILL_HTTP_RPC_URL`

4. 启动三进程
```powershell
.\start_3roles.ps1
```

5. 打开页面
- `http://127.0.0.1:8080/`

## 2. V4.0.0 新增配置项
在 `config.json` 中新增/确认：
```json
"AUTO_IDLE_PAUSE": true,
"UI_HEARTBEAT_TIMEOUT_SEC": 20
```

含义：
- `AUTO_IDLE_PAUSE=true`：无人访问页面时自动暂停后台链上请求。
- `UI_HEARTBEAT_TIMEOUT_SEC=20`：最后一次心跳后，超过 20 秒进入空闲暂停。

## 3. 心跳与自动休眠机制
### 页面打开时
- 前端会立即发送一次 `/heartbeat`，之后周期心跳。
- 后端收到心跳后，恢复 realtime/backfill/价格轮询。

### 页面关闭时
- 超过 `UI_HEARTBEAT_TIMEOUT_SEC` 未收到心跳，自动暂停：
  - 实时 WS 监听
  - 自动回扫循环
  - 价格轮询

### 再次打开页面
- 自动恢复实时处理。
- 不自动追平历史；需要历史请手动区间回扫。

## 4. 延迟指标解释（V4.0.0）
### 顶部三项
1. `链上->云端`：交易上链到入库耗时。
2. `云端->界面`：交易入库后，到页面首次看到该交易的耗时样本。
3. `端到端`：交易上链到页面首次看到该交易的耗时样本。

单位规则：
- `< 1s` 显示 `ms`
- `>= 1s` 显示 `s`

### 底部“交易录入延迟”表
- 显示的是秒级录入延迟数值（无单位字符）。
- 因源数据为秒级时间戳，`0` 代表“< 1 秒”量级。

## 5. 常用运行命令
### 手动分别启动
```powershell
python virtuals_bot.py --config .\config.json --role writer
python virtuals_bot.py --config .\config.json --role realtime
python virtuals_bot.py --config .\config.json --role backfill
```

### 一键停止
```powershell
.\stop_3roles.ps1
```

## 6. 升级到 V4.0.0 的操作步骤
1. 拉取 V4.0.0 代码。
2. 对照 `config.example.json` 补齐新配置项。
3. 停掉旧进程。
4. 重新启动三进程。
5. 浏览器 `Ctrl+F5` 强制刷新页面缓存。

## 7. 常见问题
1. `OSError [10048]` 端口占用
- 说明 8080 已被占用。先停止旧 writer，或修改 `API_PORT`。

2. 页面关闭后仍在请求节点
- 检查是否真的关闭了所有打开该页面的浏览器标签页。
- 确认 `AUTO_IDLE_PAUSE=true` 且 writer 已重启。

3. 新项目与旧项目同内盘地址时延迟指标不显示
- V4.0.0 已按项目隔离延迟统计；请强刷页面并等待新交易样本进入。
