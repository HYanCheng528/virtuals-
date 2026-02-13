# v1.5.0 Release 使用说明（给下载者）

## 1. 获取代码

### 方式 A：下载 ZIP
1. 打开仓库：`https://github.com/HYanCheng528/virtual-`
2. 进入 `Tags`，选择 `v1.5.0`
3. 下载源码 ZIP 并解压

### 方式 B：Git（推荐）
```powershell
git clone https://github.com/HYanCheng528/virtual-.git
cd virtual-
git checkout v1.5.0
```

## 2. 安装依赖

```powershell
python -m pip install -r requirements.txt
```

## 3. 创建配置文件

```powershell
Copy-Item .\config.example.json .\config.json
```

## 4. 编辑 `config.json`

至少需要填写/确认以下配置：

- `WS_RPC_URL`
- `HTTP_RPC_URL`
- `LAUNCH_CONFIGS`（项目名和内盘地址）
- `MY_WALLETS`
- `VIRTUAL_USDC_PAIR_ADDR`（链上价格模式）

可选（推荐）：

- `BACKFILL_HTTP_RPC_URL`（把回扫请求分流到独立 HTTP 节点）

## 5. 启动程序

```powershell
python virtuals_bot.py --config .\config.json
```

## 6. 打开面板

- Dashboard：`http://127.0.0.1:8080/`
- 健康检查：`http://127.0.0.1:8080/health`

## 常见问题

### 1) 端口 8080 被占用
修改 `config.json`：

- `API_PORT` 改为其它端口（如 `8081`）

然后重启程序。

### 2) 程序关闭方式
在运行窗口按：

- `Ctrl + C`

### 3) 为什么仓库里没有 `config.json`
`config.json` 被 `.gitignore` 忽略（避免泄露私有配置和节点信息），请按上面步骤由 `config.example.json` 复制生成。
