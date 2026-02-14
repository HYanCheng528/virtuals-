# RELEASE v2.0.0 更新说明

发布日期：2026-02-14

## 一、版本目标
本版本将原有单进程架构升级为三进程分工架构，减少实时与回扫互相抢占资源导致的延迟波动，并增强 UI 的配置管理能力。

## 二、核心更新内容
1. 三进程运行架构（Split Role）
- 新增角色：`writer` / `realtime` / `backfill`
- `writer` 负责 API/UI 与主库写入
- `realtime` 负责 WS 实时监听与解析
- `backfill` 负责自动回扫与手动区间回扫执行

2. 事件总线与任务解耦
- 新增事件总线 SQLite：`EVENT_BUS_SQLITE_PATH`
- 手动回扫改为 `writer` 建任务，`backfill` 执行任务
- 支持任务状态查询、取消、异常后重排队恢复

3. UI 功能增强
- 项目配置表单与当前项目选择联动
- 我的钱包区域支持：
  - 添加监控钱包（持久化）
  - 删除监控钱包（持久化）
  - 单钱包“重算当前项目”

4. 钱包重算能力
- 新增按“当前项目 + 单钱包”重算接口：`POST /wallet-recalc`
- 用已有 `events` 重建该钱包在该项目下的 `wallet_positions`
- 不放宽 swap 识别规则，不改解析口径

5. 工程与运维
- 新增一键脚本：`start_3roles.ps1` / `stop_3roles.ps1`
- 文档更新：三进程运行说明、配置样例新增事件总线路径

## 三、使用说明（快速）
1. 安装依赖
```bash
cd virtual
python -m pip install -r requirements.txt
```

2. 准备配置
```powershell
Copy-Item .\config.example.json .\config.json
```
按实际填写 RPC、项目和钱包。

3. 启动（推荐三进程）
```powershell
python virtuals_bot.py --config .\config.json --role writer
python virtuals_bot.py --config .\config.json --role realtime
python virtuals_bot.py --config .\config.json --role backfill
```

4. 打开页面
- `http://127.0.0.1:8080/`

5. 钱包补算（晚添加钱包时）
- 在“我的钱包持仓”区域先添加钱包
- 点击该钱包后方“重算当前项目”按钮
- 系统仅重算该项目下该钱包，不影响其他钱包

## 四、配置建议（低延迟）
- `CONFIRMATIONS = 0`
- `BACKFILL_INTERVAL_SEC = 3`
- `BACKFILL_CHUNK_BLOCKS = 10`（若节点限制更严可调到 5）
- `DB_FLUSH_MS = 300 ~ 500`
- 实时与回扫建议分开节点

## 五、兼容说明
- 保留 `--role all` 单进程兼容模式
- 旧配置可继续使用；若使用三进程，需配置 `EVENT_BUS_SQLITE_PATH`

## 六、已知注意点
- 高峰期不建议做大范围重算，建议按“当前项目 + 单钱包”进行。
- 仍建议在项目开始前提前添加监控钱包，以获得最完整实时持仓。
