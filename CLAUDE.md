# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server implementing a sophisticated Multi-Agent System (MAS) for sequential thinking. Built with Python using the Agno framework, it provides a `sequentialthinking` tool that coordinates multiple specialist AI agents to process complex thoughts through planning, research, analysis, critique, and synthesis.

## Core Architecture

- **Single-file Architecture**: All code is contained in `main.py` (no modules or separate files)
- **Multi-Agent Coordination**: Uses Agno's `Team` class in `coordinate` mode where:
  - Team object acts as the Coordinator
  - Specialist agents: Planner, Researcher, Analyzer, Critic, Synthesizer
  - Each thought triggers delegation to relevant specialists, then synthesis of responses
- **MCP Integration**: FastMCP server providing tools and prompts via stdio
- **State Management**: `AppContext` dataclass tracks thought history and branching
- **Data Validation**: Pydantic models with comprehensive validation (ThoughtData)

## Development Commands

```bash
# Install dependencies (using uv - recommended)
uv pip install -r requirements.txt

# Install with dev dependencies  
uv pip install -e ".[dev]"

# Run the server directly
python main.py

# Run via uv (as package)
uv run mcp-server-mas-sequential-thinking

# Run with specific directory path
uv --directory /path/to/mcp-server-mas-sequential-thinking run mcp-server-mas-sequential-thinking

# Development tools (from pyproject.toml dev dependencies)
pytest          # Run tests
black .         # Code formatting  
isort .         # Import sorting
mypy .          # Type checking

# Note: No requirements.txt file exists - all dependencies are in pyproject.toml
# Use: uv pip install . (or pip install .)
```

## Environment Configuration

Required environment variables based on LLM provider:

```bash
# Provider selection (defaults to "deepseek")
LLM_PROVIDER=deepseek  # or "groq", "openrouter", "azure"

# Provider-specific API keys (only set the one for your provider)
DEEPSEEK_API_KEY=your_key_here
GROQ_API_KEY=your_key_here
OPENROUTER_API_KEY=your_key_here
AZURE_OPENAI_API_KEY=your_key_here
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/

# Optional: Custom model IDs for different providers
DEEPSEEK_TEAM_MODEL_ID=deepseek-chat      # For coordinator
DEEPSEEK_AGENT_MODEL_ID=deepseek-chat     # For specialists
GROQ_TEAM_MODEL_ID=deepseek-r1-distill-llama-70b
GROQ_AGENT_MODEL_ID=qwen-2.5-32b
OPENROUTER_TEAM_MODEL_ID=deepseek/deepseek-chat-v3-0324
OPENROUTER_AGENT_MODEL_ID=deepseek/deepseek-r1
AZURE_TEAM_MODEL_ID=gpt-4                        # For coordinator (deployment name)
AZURE_AGENT_MODEL_ID=gpt-4                       # For specialists (deployment name)

# Optional: Base URL override (for custom endpoints)
DEEPSEEK_BASE_URL=your_base_url_if_needed

# Optional: External tools
EXA_API_KEY=your_key_here  # For Researcher agent web search
```

## Key Components

### ThoughtData Model (lines 72-200)
- Comprehensive Pydantic validation for thought inputs
- Handles revisions (`isRevision`, `revisesThought`)  
- Supports branching (`branchFromThought`, `branchId`)
- Minimum 5 thoughts enforced via validator

### Agent Team Setup (lines 304-486)
- `create_sequential_thinking_team()` creates the 5 specialist agents
- Each agent has specific tools and instructions for their domain
- Team uses `coordinate` mode for delegation and synthesis

### MCP Tool Implementation (lines 590-800)
- `sequentialthinking` tool processes each thought through the agent team
- Comprehensive error handling and logging
- Returns synthesized coordinator response as guidance for next steps

## Logging

- Logs to `~/.sequential_thinking/logs/sequential_thinking.log`
- Rotating file handler (10MB, 5 backups)
- Console output to stderr
- Structured logging of thought processing and agent interactions

## Important Architecture Details

### Token Consumption Warning
⚠️ **High Token Usage**: This MAS architecture consumes 3-6x more tokens than single-agent approaches due to parallel specialist processing. Each thought triggers multiple agent calls.

### Installation Methods
- **Smithery**: `npx -y @smithery/cli install @FradSer/mcp-server-mas-sequential-thinking --client claude`
- **Manual**: Clone repo and install dependencies via pyproject.toml

### MCP Client Configuration

#### DeepSeek Configuration
```json
{
  "mcpServers": {
    "mas-sequential-thinking": {
      "command": "uvx", 
      "args": ["mcp-server-mas-sequential-thinking"],
      "env": {
        "LLM_PROVIDER": "deepseek",
        "DEEPSEEK_API_KEY": "your_key_here"
      }
    }
  }
}
```

#### Azure OpenAI Configuration
```json
{
  "mcpServers": {
    "mas-sequential-thinking": {
      "command": "uvx", 
      "args": ["mcp-server-mas-sequential-thinking"],
      "env": {
        "LLM_PROVIDER": "azure",
        "AZURE_OPENAI_API_KEY": "your_azure_api_key_here",
        "AZURE_OPENAI_ENDPOINT": "https://your-resource.openai.azure.com/",
        "AZURE_TEAM_MODEL_ID": "gpt-4",
        "AZURE_AGENT_MODEL_ID": "gpt-4"
      }
    }
  }
}
```

## Development Notes

- All logic is in single `main.py` file - no separate modules
- Uses dataclasses and Pydantic for data structures  
- Agent instructions are embedded as string lists in agent definitions  
- Context lifespan managed through `app_lifespan()` asynccontextmanager
- Tool returns only the coordinator response string (not full JSON)
- No requirements.txt - dependencies managed via pyproject.toml
- Supports revision (`isRevision=True`) and branching (`branchFromThought`) workflows