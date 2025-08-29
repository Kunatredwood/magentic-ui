# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Magentic-UI is a research prototype of a human-centered interface powered by a multi-agent system that can browse and perform actions on the web, generate and execute code, and analyze files. It's built using AutoGen and provides a platform for studying human-agent interaction.

The application consists of a Python backend (FastAPI) and a React frontend (Gatsby), with Docker containers for code execution and browser automation.

## Essential Commands

### Development Commands
```bash
# Install dependencies (development)
uv venv --python=3.12 .venv
uv sync --all-extras
source .venv/bin/activate

# Run the application
magentic-ui --port 8081

# Run without Docker (limited functionality)
magentic-ui --run-without-docker --port 8081

# CLI interface
magentic-cli --work-dir PATH/TO/STORE/DATA
```

### Code Quality and Testing
```bash
# Format code
poe fmt

# Lint code
poe lint

# Type checking (multiple options available)
poe pyright
poe mypy

# Run tests
poe test

# Run all checks (format, lint, type check, test)
poe check
```

### Frontend Development
```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install -g gatsby-cli
npm install --global yarn
yarn install

# Development mode (with hot reload)
npm run start
# Frontend dev server runs at http://localhost:8000

# Build for production
yarn build
# Copies built files to ../src/magentic_ui/backend/web/ui/

# Type checking
npm run typecheck
```

### Docker Management
```bash
# Build Docker images manually (if needed)
cd docker
sh build-all.sh

# Check Docker status
docker ps
```

## Architecture Overview

### Backend Structure (`src/magentic_ui/`)
- **CLI Entry**: `backend/cli.py` - Main CLI interface using Typer
- **Web API**: `backend/web/app.py` - FastAPI application with routing
- **Task Teams**: `task_team.py` - Creates multi-agent teams (GroupChat/RoundRobinGroupChat)
- **Agents**: `agents/` - Specialized agents (WebSurfer, CoderAgent, FileSurfer, McpAgent)
- **Teams**: `teams/orchestrator/` - Team orchestration and planning logic
- **Tools**: `tools/` - Browser automation (Playwright), search (Bing), MCP integrations
- **Configuration**: `magentic_ui_config.py` - Central configuration management

### Frontend Structure (`frontend/src/`)
- **Components**: `components/` - Reusable UI components
- **Views**: `views/` - Main application views (chat, manager, sidebar)
- **Store**: `store.tsx` - Global state management with Zustand
- **Settings**: `components/settings/` - Configuration UI for models, agents, MCP servers

### Key Agent Architecture
The system uses a multi-agent approach:
1. **Orchestrator**: Plans and coordinates tasks
2. **WebSurfer**: Handles web browsing and interaction
3. **CoderAgent**: Executes code in sandboxed containers
4. **FileSurfer**: File system operations
5. **McpAgent**: Integrates with Model Context Protocol servers
6. **UserProxy**: Handles human interaction and approvals

### Configuration System
- Main config: `MagenticUIConfig` class with model clients, approval policies, etc.
- Model clients: Supports OpenAI, Azure OpenAI, Ollama
- MCP integration: Configurable agents with stdio/SSE server connections
- Runtime config: YAML files can override UI settings

## Important Development Notes

### Testing
- Uses pytest with async support
- Playwright installation required: `playwright install`
- Tests exclude npx-dependent tests by default
- Run with coverage: `pytest --cov=src --cov-report=xml`

### Code Quality
- Strict type checking with both mypy and pyright
- Ruff for formatting and linting
- Exclude certain directories from type checking (assistantbench, some web routes)

### Docker Requirements
- Browser automation requires Docker containers
- Two main images: browser (VNC-enabled) and Python environment
- Without Docker: limited functionality, no code execution or live browser view
- Images pulled from GHCR automatically

### Model Client Configuration
Support for multiple LLM providers:
- OpenAI (default): `OpenAIChatCompletionClient`
- Azure OpenAI: `AzureOpenAIChatCompletionClient`
- Ollama: Local model support
- Each agent can use different model clients

### MCP (Model Context Protocol) Integration
- Agents can connect to MCP servers via stdio or SSE
- Configure in YAML with server parameters and agent descriptions
- Example: AirBnB server integration for travel-related tasks

### Security and Approval System
- Action guards for sensitive operations
- Configurable approval policies: "always", "never", "auto"
- Human-in-the-loop for critical actions
- URL allow/block lists for web browsing

### Frontend Development
- Gatsby-based React application
- Tailwind CSS for styling
- TypeScript throughout
- Real-time updates via WebSocket connections
- Monaco editor for code display
- VNC integration for browser viewing

## Common Development Workflows

### Adding New Agents
1. Create agent class in `src/magentic_ui/agents/`
2. Update `task_team.py` to include in team configuration
3. Add UI configuration in frontend settings
4. Update configuration schema in `magentic_ui_config.py`

### Modifying Web Interface
1. Work in `frontend/` directory
2. Use `npm run start` for development with hot reload
3. Build with `yarn build` to copy to backend
4. Frontend serves from `http://localhost:8000`, backend from configured port

### Configuration Changes
- Backend configs: Modify `MagenticUIConfig` class
- Frontend configs: Update settings components and validation schemas
- Runtime configs: YAML files with AutoGen model client specifications

### Troubleshooting
- Docker issues: Check TROUBLESHOOTING.md
- Port conflicts: Use `--port` flag to change default 8081
- WSL2 setup required for Windows users
- Check firewall settings for remote server deployments