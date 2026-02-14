# V2.1.0 更新说明

发布日期：2026-02-14

## 本次版本目标
在 V2.0.0（三进程）基础上，继续优化高峰期稳定性与可调优能力，重点解决“实时与回扫并发时，参数不够细分”和“入库吞吐调参不便”两个问题。

## 主要更新

1. 回执并发参数拆分（实时/回扫独立）
- 新增配置：
  - `RECEIPT_WORKERS_REALTIME`
  - `RECEIPT_WORKERS_BACKFILL`
- 作用：实时链路与回扫链路可以分别设置 worker 并发，互不绑定。
- 兼容性：若未填写上述新字段，会自动回退到原有 `RECEIPT_WORKERS`。

2. 新增运行时 DB 批量调节（UI 可直接改）
- Dashboard 新增“入库批量调节（DB_BATCH_SIZE）”区域：
  - 预设档位：`1 / 5 / 10 / 20 / 50`
  - 支持自定义输入：`1~100`
  - 支持一键应用和状态反馈
- 作用：无需重启进程即可在线调节 writer 每轮入库批量。

3. 新增运行时调参 API
- `GET /runtime/db-batch-size`：读取当前生效值
- `POST /runtime/db-batch-size`：设置并立即生效
- 同时在 `/meta` 返回 `runtimeTuning.db_batch_size`，前端可直接同步展示。

4. DB 批量参数持久化
- 当前生效的 `DB_BATCH_SIZE` 会写入 `system_state(runtime_db_batch_size)`。
- 进程重启后自动恢复最近一次在 UI/API 设置的值。

5. Writer 循环改为动态读取批量
- `bus_writer_loop` 每轮读取当前 `db_batch_size`，调参后即时生效，不需重启。

6. 文档与示例配置更新
- `README.md` 补充新参数说明与回退行为。
- `config.example.json`、`config.lb_test.json` 增加新参数示例。

## 升级兼容说明
- 旧配置可直接运行，不会因为缺少新字段报错。
- 单进程 `--role all` 仍按 `RECEIPT_WORKERS` 工作。
- 三进程模式下建议按链路压力分别配置 realtime/backfill worker。

## 变更文件
- `virtuals_bot.py`
- `dashboard.html`
- `README.md`
- `config.example.json`
- `config.lb_test.json`
- `RELEASE_v2.1.0_更新说明.md`
