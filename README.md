# LoggingAnalysis

[English](./README_EN.md)

AI驱动的日志分析工具，通过MCP服务器接口为LLM提供日志分析能力。

## 功能特性

- **智能分块处理**：自动将大型日志文件分割成可管理的块
- **两阶段AI分析**：
  - 使用经济高效的AI模型从每个块中提取关键信息（异常、库引用、问题行为）
  - 使用强大的AI模型整合所有提取结果，生成综合分析报告
- **结构化输出**：所有分析结果都使用Pydantic进行数据验证
- **可选网页搜索**：集成zai-sdk，可搜索额外上下文信息
- **MCP服务器接口**：可作为MCP服务器运行，便于LLM集成

## 安装

```bash
# 使用 uv 安装依赖
uv sync

# 或安装开发依赖
uv sync --extra dev
```

## 配置

1. 复制 `.env.example` 到 `.env`：

```bash
cp .env.example .env
```

2. 编辑 `.env` 文件，配置你的API密钥：

```bash
OPENAI_API_KEY=your_openai_api_key_here
EXTRACTION_MODEL=gpt-4o-mini
INTEGRATION_MODEL=gpt-4o
```

## 使用

### 作为MCP服务器

```bash
uv run logginganalysis-mcp
```

### 直接使用Python API

```python
import asyncio
from logginganalysis import LogAnalyzer

async def main():
    analyzer = LogAnalyzer()
    with open("application.log", "r") as f:
        log_content = f.read()
    report = await analyzer.analyze(log_content)
    print(report.analysis.overall_summary)

asyncio.run(main())
```

## 项目结构

```
logginganalysis/
├── config/          # 配置管理
├── models/          # 数据模型
├── chunking/        # 日志分块
├── extraction/      # 信息提取
├── integration/     # 信息集成
├── reporting/       # 报告生成
├── mcp/            # MCP服务器
└── utils/          # 工具函数
```

## 开发

### 运行测试

```bash
# 运行所有测试
pytest

# 运行特定测试文件
pytest tests/test_specific_file.py

# 查看测试覆盖率
pytest --cov=logginganalysis
```

### 项目架构

该项目采用**管道架构**处理日志：

1. **日志分块模块** (`chunking/`) - 将大型日志分割成可管理的块
   - 支持多种分块策略（时间戳、错误边界等）
   - 使用 LangChain 实现灵活的分块逻辑

2. **AI 提取模块** (`extraction/`) - 使用经济高效的 AI 模型提取：
   - 异常信息
   - 引用的库
   - 潜在问题（数据库故障、网络超时等）
   - 内置流控功能，支持 TPM/RPM 限制

3. **集成模块** (`integration/`) - 使用强大的 AI 模型整合提取的信息
   - 综合所有块的分析结果
   - 可选的网页搜索功能（zai-sdk）
   - 内置流控功能

4. **报告生成模块** (`reporting/`) - 生成综合分析报告
   - 支持 Markdown、JSON、Text 等多种输出格式

5. **MCP 服务器模块** (`mcp/`) - 提供 MCP 协议接口
   - 标准化的工具定义和验证
   - 易于与 LLM 工作流集成

### 核心功能

- **流控 (Rate Limiting)**：支持 TPM（每分钟 Token 数）和 RPM（每分钟请求数）限制
  - 令牌桶算法（Token Bucket）用于 RPM
  - 滑动窗口算法（Sliding Window）用于 TPM
  - 可通过环境变量配置启用/禁用

### 环境变量

在 `.env` 文件中可配置以下变量：

```bash
# API 密钥
OPENAI_API_KEY=your_openai_api_key_here

# AI 模型配置
EXTRACTION_MODEL=gpt-4o-mini    # 用于提取阶段的经济模型
INTEGRATION_MODEL=gpt-4o         # 用于集成阶段的强大模型

# 流控配置（可选）
ENABLE_RATE_LIMIT=true           # 启用流控
TPM_LIMIT=100000                 # TPM 限制（每分钟 Token 数）
RPM_LIMIT=500                    # RPM 限制（每分钟请求数）
RATE_LIMIT_BURST=10              # 突发容量
```

## 许可证

MIT License
