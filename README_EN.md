# LoggingAnalysis

[Chinese](./README.md)

An AI-driven log analysis tool with MCP server interface for LLM integration.

## Features

- **Intelligent Chunking**: Automatically splits large log files into manageable chunks
- **Two-Stage AI Analysis**:
  - Uses cost-effective AI models to extract key information from each chunk (exceptions, library references, problematic behaviors)
  - Uses powerful AI models to synthesize all extractions into comprehensive analysis reports
- **Structured Output**: All analysis results validated with Pydantic
- **Optional Web Search**: Integrates zai-sdk for additional context
- **MCP Server Interface**: Can run as an MCP server for easy LLM integration

## Installation

```bash
# Install dependencies with uv
uv sync

# Or install with development dependencies
uv sync --extra dev
```

## Configuration

1. Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

2. Edit `.env` and configure your API keys:

```bash
OPENAI_API_KEY=your_openai_api_key_here
EXTRACTION_MODEL=gpt-4o-mini
INTEGRATION_MODEL=gpt-4o
```

## Usage

### As MCP Server

```bash
uv run logginganalysis-mcp
```

### Direct Python API

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

## Project Structure

```
logginganalysis/
├── config/          # Configuration management
├── models/          # Data models
├── chunking/        # Log chunking
├── extraction/      # Information extraction
├── integration/     # Information integration
├── reporting/       # Report generation
├── mcp/            # MCP server
└── utils/          # Utility functions
```

## Development

### Running Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_specific_file.py

# View test coverage
pytest --cov=logginganalysis
```

### Architecture

The project follows a **pipeline architecture** for log processing:

1. **Log Chunking Module** (`chunking/`) - Splits large logs into manageable pieces
   - Supports multiple chunking strategies (timestamp, error boundaries, etc.)
   - Uses LangChain for flexible chunking logic

2. **AI Extraction Module** (`extraction/`) - Uses cost-effective AI models to extract:
   - Exception information
   - Referenced libraries
   - Potential issues (database failures, network timeouts, etc.)
   - Built-in rate limiting with TPM/RPM support

3. **Integration Module** (`integration/`) - Uses powerful AI models to synthesize extracted information
   - Combines analysis results from all chunks
   - Optional web search capabilities (zai-sdk)
   - Built-in rate limiting

4. **Report Generation Module** (`reporting/`) - Creates comprehensive analysis reports
   - Supports multiple output formats (Markdown, JSON, Text)

5. **MCP Server Module** (`mcp/`) - Provides MCP protocol interface
   - Standardized tool definitions and validation
   - Easy integration with LLM workflows

### Core Features

- **Rate Limiting**: Supports TPM (Tokens Per Minute) and RPM (Requests Per Minute) limits
  - Token Bucket algorithm for RPM
  - Sliding Window algorithm for TPM
  - Configurable via environment variables

### Environment Variables

Configure the following variables in your `.env` file:

```bash
# API Keys
OPENAI_API_KEY=your_openai_api_key_here

# AI Model Configuration
EXTRACTION_MODEL=gpt-4o-mini    # Cost-effective model for extraction stage
INTEGRATION_MODEL=gpt-4o         # Powerful model for integration stage

# Rate Limiting Configuration (Optional)
ENABLE_RATE_LIMIT=true           # Enable rate limiting
TPM_LIMIT=100000                 # TPM limit (tokens per minute)
RPM_LIMIT=500                    # RPM limit (requests per minute)
RATE_LIMIT_BURST=10              # Burst capacity
```

## License

MIT License
