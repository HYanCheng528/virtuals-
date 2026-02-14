# v2.0.0 三进程运行说明

本版本新增 Split Role 运行模式，可将程序拆分为 3 个进程：

1. `writer`：唯一写库进程 + API/UI 服务进程（默认 8080）
2. `realtime`：实时 WS 监听 + receipt 解析 + 写入事件总线
3. `backfill`：自动回扫/手动回扫执行 + receipt 解析 + 写入事件总线

## 关键点

- 3 进程共享同一个 `config.json`。
- 新增配置：`EVENT_BUS_SQLITE_PATH`（默认 `./data/virtuals_bus.db`）。
- UI 只连接 `writer` 进程，地址仍是 `http://127.0.0.1:8080/`。
- 手动回扫接口由 `writer` 创建任务，`backfill` 进程执行。

## 启动方式（3 个终端）

```powershell
cd C:\Users\hyc\Desktop\Codex\virtual
python virtuals_bot.py --config .\config.json --role writer
```

```powershell
cd C:\Users\hyc\Desktop\Codex\virtual
python virtuals_bot.py --config .\config.json --role realtime
```

```powershell
cd C:\Users\hyc\Desktop\Codex\virtual
python virtuals_bot.py --config .\config.json --role backfill
```

## 兼容模式（单进程）

仍可使用旧模式：

```powershell
python virtuals_bot.py --config .\config.json --role all
```

## 停止方式

每个终端按一次 `Ctrl + C` 即可。

## 故障恢复说明

- `writer` 重启后会继续消费 `EVENT_BUS_SQLITE_PATH` 中未处理事件。
- 手动回扫任务若在 `running` 状态异常中断，重启后会自动回到 `queued` 再执行。

