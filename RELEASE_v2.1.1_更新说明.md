# V2.1.1 更新说明

发布日期：2026-02-14

## 版本目标
本版本聚焦 UI 观感与操作一致性，不改后端统计口径和核心计算精度。

## 主要更新

1. 大户榜与我的钱包持仓数值展示优化
- `累计估算花费V` / `累计花费V`：显示为 2 位小数。
- `累计买入代币` / `累计代币数量`：改为“万”为单位，显示 2 位小数。
- `FDV_USD`：改为“万”为单位，显示 2 位小数。
- 鼠标悬停在数字上可查看精确原始值。
- 说明：仅 UI 显示格式变化，实际计算与入库精度保持原样。

2. 分钟消耗柱状图数字展示优化
- 图中纵轴刻度和柱顶数值改为整数显示（四舍五入）。
- 说明：仅显示方式变化，不影响分钟聚合数据本身。

3. 时间输入持久化（防止刷新重置）
- 页眉回扫时间：`scanStartUtc8` / `scanEndUtc8`
- 分钟区间时间：`minuteStartUtc8` / `minuteEndUtc8`
- 页面刷新后保持刷新前的时间值；仅当用户手动修改时更新。

4. 版本标识同步
- UI 标题和副标题更新为 `v2.1.1`。
- CLI 帮助描述更新为 `v2.1.1`。
- README 顶部版本更新为 `v2.1.1`，并新增本版本说明入口。

## 全面检查结果

已执行并通过：
- Python 语法检查：`python -m py_compile virtuals_bot.py`
- Python 全量编译检查：`python -m compileall -q virtual`
- 启动参数检查：`python virtuals_bot.py --help`
- 前端脚本语法检查：`dashboard.html` 内嵌 JS 解析通过
- 配置文件检查：`config.example.json`、`config.lb_test.json` 可正常解析
- 冲突标记检查：无 `<<<<<<< / ======= / >>>>>>>`

## 变更文件
- `dashboard.html`
- `virtuals_bot.py`
- `README.md`
- `RELEASE_v2.1.1_更新说明.md`
