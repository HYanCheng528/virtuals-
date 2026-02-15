# V3.0.0 更新说明

## 一、版本定位
V3.0.0 是对外分发版本，目标是“朋友下载后可直接使用”，同时确保仓库不泄露 API Key。

## 二、核心变更
- 品牌升级：`V-Pulse 盘面雷达`
- 浏览器页签标题更新为 `V-Pulse 盘面雷达 v3.0.0`
- 新增自定义图标：`favicon-vpulse.svg`
- 后端增加图标路由：`/favicon-vpulse.svg`
- `datetime-local` 日历按钮改为白色可见样式（暗色主题下更清晰）

## 三、发布工程化整理
- 配置模板脱敏：`config.example.json` 中 API 全部替换为占位符
- 保留钱包/业务地址字段，便于团队直接复用
- 文档重构：
  - `README.md`（详细版）
  - `RELEASE_v3.0.0_使用说明.md`（上手教程）
  - `需求文档_v3.0.0.md`（可复现需求）

## 四、兼容性
- 不改变核心数据逻辑与统计口径
- 原有三进程运行方式保持兼容
- 原有 `config.json` 可继续使用

## 五、升级建议
1. 重启 writer/realtime/backfill 三进程。
2. 浏览器执行 `Ctrl + F5` 强制刷新。
3. 检查 `health` 页面确认 `ws_connected=true`。