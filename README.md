# indie-finance-plugin

独立投资者金融分析插件 — 基于 Anthropic [financial-services-plugins](https://github.com/anthropics/financial-services-plugins) 架构，替换为免费数据源，新增 Crypto 模块。

为独立投资者提供机构级金融分析能力，覆盖传统金融（美股/港股）和加密市场（代币/DeFi/空投）。

## 快速开始

### 1. 安装插件

```bash
claude plugin add /path/to/indie-finance-plugin
```

### 2. 配置 API Key（可选，增强功能）

MCP server 需要的 API Key（免费注册）：

| 服务 | 注册地址 | 用途 |
|------|---------|------|
| Alpha Vantage | alphavantage.co/support | 技术指标、电话会议（官方 MCP） |
| Dune Analytics | dune.com/settings/api | 链上查询（官方 MCP） |

CoinGecko Demo 模式无需 Key。Yahoo Finance、DefiLlama、FRED 通过 Web Search 访问，无需 Key。

设置方式：
```bash
export ALPHA_VANTAGE_API_KEY="your-key"
export DUNE_API_KEY="your-key"
```

## 子插件

### tradfi — 传统金融分析

Fork 自 Anthropic 官方 `financial-analysis` 和 `equity-research` 插件，保留分析框架，替换数据源为免费 MCP。

| 命令 | 用途 | 输出 |
|------|------|------|
| `/tradfi:comps [ticker]` | 可比公司分析 | Excel + Markdown |
| `/tradfi:dcf [ticker]` | DCF 估值模型 | Excel + Markdown |
| `/tradfi:earnings [ticker] [Q]` | 财报分析 | Markdown |
| `/tradfi:screen [条件]` | 股票筛选 | Markdown |
| `/tradfi:thesis [ticker]` | 投资论文追踪 | Markdown |
| `/tradfi:model-update [ticker]` | 模型更新（财报后） | Markdown + Excel |
| `/tradfi:debug-model` | 电子表格审计 | Markdown |

自动触发 skill（无需命令）：competitive-analysis、idea-generation、audit-xls

数据源：Alpha Vantage (MCP) → Yahoo Finance (Web Search) → Chrome

### crypto — 加密市场分析

全新模块，覆盖代币基本面、DeFi 协议、空投评估、链上查询。

| 命令 | 用途 | 输出 |
|------|------|------|
| `/crypto:token [symbol]` | 代币全面分析 | Markdown |
| `/crypto:defi [protocol]` | DeFi 协议分析 | Markdown |
| `/crypto:airdrop [project]` | 空投项目评估（六维度） | Markdown |
| `/crypto:onchain [query]` | 链上数据查询 | 对话内表格 |

数据源：CoinGecko (MCP) → Dune (MCP) → DefiLlama (Web Search) → Chrome

### macro — 宏观经济

横跨传统和加密市场的宏观经济看板。

| 命令 | 用途 | 输出 |
|------|------|------|
| `/macro:dashboard` | 宏观经济看板 | Markdown |
| `/macro:morning` | 晨间市场笔记 | Markdown |
| `/macro:catalyst` | 催化剂日历 | Markdown |

数据源：CoinGecko (MCP) → FRED (Web Search) → DefiLlama (Web Search) → Chrome

### portfolio — 投资组合管理

管理多账户投资组合，税务优化。

| 命令 | 用途 | 输出 |
|------|------|------|
| `/portfolio:rebalance` | 税务感知再平衡 | Excel + Markdown |
| `/portfolio:tlh` | 税收损失收获 | Excel + Markdown |

数据源：Yahoo Finance (Web Search) → Chrome

## 数据架构

### 三层 Fallback

所有命令遵循统一的数据获取策略：

1. **MCP 数据源** — 首选，通过 MCP 协议直接查询
2. **Web Search** — MCP 不可用时，搜索权威来源
3. **Chrome CDP** — 需要登录或动态渲染的页面

### MCP 数据源总览

仅启用通过安全审计的官方 MCP server（详见 [安全审计报告](docs/mcp-security-audit.md)）。

| MCP Server | 类型 | 官方 | 限制 | 子插件 |
|-----------|------|------|------|--------|
| CoinGecko | 官方 MCP | 是 | 30次/分, 10000次/月 | crypto, macro |
| Alpha Vantage | 官方 MCP | 是 | 25次/天 | tradfi |
| Dune Analytics | 官方 MCP | 是 | 55次/分 | crypto |

以下数据源通过 Web Search fallback 访问（无官方 MCP）：

| 数据源 | 限制 | 覆盖场景 |
|--------|------|---------|
| Yahoo Finance | 无限制 | 股票行情、财报、组合管理 |
| DefiLlama | 无限制 | DeFi TVL、协议数据 |
| FRED | 120次/分 | 宏观经济指标 |
| FMP | 250次/天 | SEC filing、分析师数据 |

## 目录结构

```
indie-finance-plugin/
├── .claude-plugin/marketplace.json    # 插件市场入口
├── .gitignore
├── CLAUDE.md                          # Claude 使用指引
├── README.md
├── tradfi/                            # 传统金融子插件
│   ├── .claude-plugin/plugin.json
│   ├── .mcp.json
│   ├── commands/
│   ├── skills/
│   └── hooks/hooks.json
├── crypto/                            # 加密市场子插件
│   ├── .claude-plugin/plugin.json
│   ├── .mcp.json
│   ├── commands/
│   ├── skills/
│   └── hooks/hooks.json
├── macro/                             # 宏观经济子插件
│   ├── .claude-plugin/plugin.json
│   ├── .mcp.json
│   ├── commands/
│   ├── skills/
│   └── hooks/hooks.json
├── portfolio/                         # 投资组合管理子插件
│   ├── .claude-plugin/plugin.json
│   ├── .mcp.json
│   ├── commands/
│   ├── skills/
│   └── hooks/hooks.json
└── _reference/                        # 官方机构 skill（不激活，供参考）
```

## 开发计划

- **Phase 1**: 基础骨架 + 数据层配置 + MCP 连接验证
- **Phase 2**: TradFi 模块（Fork 9 个官方 skill + 7 个命令 + 改写数据源）
- **Phase 3**: Crypto 模块（新建 skill）— 可与 Phase 2 并行
- **Phase 4**: 宏观模块 + 集成测试
- **Phase 5**: Portfolio 模块（再平衡 + 税损收获）

## 致谢

- 分析框架 Fork 自 [anthropics/financial-services-plugins](https://github.com/anthropics/financial-services-plugins)
- 免费数据源：Yahoo Finance, CoinGecko, DefiLlama, FRED, Alpha Vantage, FMP, Dune Analytics
