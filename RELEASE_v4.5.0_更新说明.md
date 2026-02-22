# V4.5.0 更新说明

发布日期：2026-02-22

## 本次目标
在不关闭自动回扫、不放宽交易判定规则的前提下，降低高峰期实时延迟：
- 实现写库“实时优先”
- 保证回扫“持续推进不过饿”
- 在实时压力过高时自动降速回扫，压力回落后自动恢复

## 主要更新

### 1. Writer 写库优先级（Realtime > Backfill）
- 新增事件总线按来源拉取能力（`realtime` / `backfill`）
- Writer 按配额优先消费实时事件，同时保留回扫保底配额
- 默认配额：
  - `PRIORITY_WRITE_REALTIME_QUOTA = 8`
  - `PRIORITY_WRITE_BACKFILL_QUOTA = 2`

效果：
- 高峰时实时事件更快入库
- 回扫事件不会被长期饿死

### 2. 自动回扫高压降载（不关闭回扫）
- 回扫进程根据实时压力动态切换“正常/降速”模式
- 触发降速（满足任一）：
  - `realtime_pending_txs >= BACKFILL_RT_PENDING_HIGH`（默认 200）
  - `realtime_queue_size >= BACKFILL_RT_QUEUE_HIGH`（默认 400）
- 恢复正常（同时满足）：
  - `realtime_pending_txs <= BACKFILL_RT_PENDING_LOW`（默认 60）
  - `realtime_queue_size <= BACKFILL_RT_QUEUE_LOW`（默认 120）
- 降速参数默认：
  - `BACKFILL_THROTTLE_CHUNK_BLOCKS = 2`
  - `BACKFILL_THROTTLE_INTERVAL_SEC = 8`

效果：
- 实时压力大时，回扫自动让路
- 压力回落后，回扫自动恢复速度
- 回扫不断线，仍持续补漏

### 3. 监控字段补充
- 角色心跳与健康状态增加实时压力/回扫降速相关指标：
  - `realtime_pending_txs`
  - `realtime_queue_size`
  - `backfill_throttled`

用途：
- 便于在 UI 和排障中判断是否触发了降速策略

### 4. 大户榜卖出附页（本版已纳入）
- 大户榜主表保留核心字段（含 `FDV_USD(万)`）
- 卖出交易明细放在可展开附页中展示
- 在榜期间出现卖出的地址，显示差异标记并持续跟踪

## 新增配置项（已在 `config.example.json` 给出）

```json
"PRIORITY_WRITE_ENABLED": true,
"PRIORITY_WRITE_REALTIME_QUOTA": 8,
"PRIORITY_WRITE_BACKFILL_QUOTA": 2,
"BACKFILL_THROTTLE_ENABLED": true,
"BACKFILL_RT_PENDING_HIGH": 200,
"BACKFILL_RT_PENDING_LOW": 60,
"BACKFILL_RT_QUEUE_HIGH": 400,
"BACKFILL_RT_QUEUE_LOW": 120,
"BACKFILL_THROTTLE_CHUNK_BLOCKS": 2,
"BACKFILL_THROTTLE_INTERVAL_SEC": 8
```

## 兼容性与影响
- 不改交易有效性判定规则
- 不改数据库业务含义，仅新增调度策略与状态指标
- 回扫仍会写库；高压期只是更慢，不会停写

## 升级建议
1. 拉取 `v4.5.0` 后，先检查你的 `config.json` 是否包含新增项
2. 按三进程方式重启：`writer` / `realtime` / `backfill`
3. 观察高峰期：
   - 实时延迟是否下降
   - 回扫是否持续推进
   - `backfill_throttled` 是否按压力自动切换

## 回滚说明
如需临时关闭本策略，可在配置中设置：
- `PRIORITY_WRITE_ENABLED = false`
- `BACKFILL_THROTTLE_ENABLED = false`

即可回到原先“无优先级、无自动降速”的行为。
