# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Graphiti is a Python framework for building temporally-aware knowledge graphs designed for AI agents. It enables real-time incremental updates to knowledge graphs without batch recomputation, making it suitable for dynamic environments.

Key features:

- Bi-temporal data model with explicit tracking of event occurrence times
- Hybrid retrieval combining semantic embeddings, keyword search (BM25), and graph traversal
- Support for custom entity definitions via Pydantic models
- Integration with Neo4j, FalkorDB, Kuzu, and Amazon Neptune as graph storage backends
- Optional OpenTelemetry distributed tracing support
- High-concurrency ingestion pipelines with configurable rate limiting

## Development Commands

### Main Development Commands (run from project root)

```bash
# Install dependencies
uv sync --extra dev

# Format code (ruff import sorting + formatting)
make format

# Lint code (ruff + pyright type checking)
make lint

# Run tests
make test

# Run all checks (format, lint, test)
make check
```

### Server Development (run from server/ directory)

```bash
cd server/
# Install server dependencies
uv sync --extra dev

# Run server in development mode
uvicorn graph_service.main:app --reload

# Format, lint, test server code
make format
make lint
make test
```

### MCP Server Development (run from mcp_server/ directory)

```bash
cd mcp_server/
# Install MCP server dependencies
uv sync

# Run with Docker Compose
docker-compose up
```

## Code Architecture

### Core Library (`graphiti_core/`)

- **Main Entry Point**: `graphiti.py` - Contains the main `Graphiti` class that orchestrates all functionality
- **Graph Storage**: `driver/` - Database drivers for Neo4j and FalkorDB
- **LLM Integration**: `llm_client/` - Clients for OpenAI, Anthropic, Gemini, Groq
- **Embeddings**: `embedder/` - Embedding clients for various providers
- **Graph Elements**: `nodes.py`, `edges.py` - Core graph data structures
- **Search**: `search/` - Hybrid search implementation with configurable strategies
- **Prompts**: `prompts/` - LLM prompts for entity extraction, deduplication, summarization
- **Utilities**: `utils/` - Maintenance operations, bulk processing, datetime handling

### Server (`server/`)

- **FastAPI Service**: `graph_service/main.py` - REST API server
- **Routers**: `routers/` - API endpoints for ingestion and retrieval
- **DTOs**: `dto/` - Data transfer objects for API contracts

### MCP Server (`mcp_server/`)

- **MCP Implementation**: `graphiti_mcp_server.py` - Model Context Protocol server for AI assistants
- **Docker Support**: Containerized deployment with Neo4j

## Testing

- **Unit Tests**: `tests/` - Comprehensive test suite using pytest
- **Integration Tests**: Tests marked with `_int` suffix require database connections
- **Evaluation**: `tests/evals/` - End-to-end evaluation scripts

## Configuration

### Environment Variables

- `OPENAI_API_KEY` - Required for LLM inference and embeddings
- `USE_PARALLEL_RUNTIME` - Optional boolean for Neo4j parallel runtime (enterprise only)
- Provider-specific keys: `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY`, `GROQ_API_KEY`, `VOYAGE_API_KEY`

### Database Setup

- **Neo4j**: Version 5.26+ required, available via Neo4j Desktop
  - Database name defaults to `neo4j` (hardcoded in Neo4jDriver)
  - Override by passing `database` parameter to driver constructor
- **FalkorDB**: Version 1.1.2+ as alternative backend
  - Database name defaults to `default_db` (hardcoded in FalkorDriver)
  - Override by passing `database` parameter to driver constructor

## Development Guidelines

### Code Style

- Use Ruff for formatting and linting (configured in pyproject.toml)
- Line length: 100 characters
- Quote style: single quotes
- Type checking with Pyright is enforced
- Main project uses `typeCheckingMode = "basic"`, server uses `typeCheckingMode = "standard"`

### Testing Requirements

- Run tests with `make test` or `pytest`
- Integration tests require database connections and are marked with `_int` suffix
- Use `pytest-xdist` for parallel test execution
- Run specific test files: `pytest tests/test_specific_file.py`
- Run specific test methods: `pytest tests/test_file.py::test_method_name`
- Run only integration tests: `pytest tests/ -k "_int"`
- Run only unit tests: `pytest tests/ -k "not _int"`

### LLM Provider Support

The codebase supports multiple LLM providers but works best with services supporting structured output (OpenAI, Gemini). Other providers may cause schema validation issues, especially with smaller models.

#### Current LLM Models (as of November 2025)

**OpenAI Models:**
- **GPT-5 Family** (Reasoning models, require temperature=0):
  - `gpt-5-mini` - Fast reasoning model
  - `gpt-5-nano` - Smallest reasoning model
- **GPT-4.1 Family** (Standard models):
  - `gpt-4.1` - Full capability model
  - `gpt-4.1-mini` - Efficient model for most tasks
  - `gpt-4.1-nano` - Lightweight model
- **Legacy Models** (Still supported):
  - `gpt-4o` - Previous generation flagship
  - `gpt-4o-mini` - Previous generation efficient

**Anthropic Models:**
- **Claude 4.5 Family** (Latest):
  - `claude-sonnet-4-5-latest` - Flagship model, auto-updates
  - `claude-sonnet-4-5-20250929` - Pinned Sonnet version from September 2025
  - `claude-haiku-4-5-latest` - Fast model, auto-updates
- **Claude 3.7 Family**:
  - `claude-3-7-sonnet-latest` - Auto-updates
  - `claude-3-7-sonnet-20250219` - Pinned version from February 2025
- **Claude 3.5 Family**:
  - `claude-3-5-sonnet-latest` - Auto-updates
  - `claude-3-5-sonnet-20241022` - Pinned version from October 2024
  - `claude-3-5-haiku-latest` - Fast model

**Google Gemini Models:**
- **Gemini 2.5 Family** (Latest):
  - `gemini-2.5-pro` - Flagship reasoning and multimodal
  - `gemini-2.5-flash` - Fast, efficient
- **Gemini 2.0 Family**:
  - `gemini-2.0-flash` - Experimental fast model
- **Gemini 1.5 Family** (Stable):
  - `gemini-1.5-pro` - Production-stable flagship
  - `gemini-1.5-flash` - Production-stable efficient

**Note**: Model names like `gpt-5-mini`, `gpt-4.1`, and `gpt-4.1-mini` used in this codebase are valid OpenAI model identifiers. The GPT-5 family are reasoning models that require `temperature=0` (automatically handled in the code).

### MCP Server Usage Guidelines

When working with the MCP server, follow the patterns established in `mcp_server/cursor_rules.md`:

- Always search for existing knowledge before adding new information
- Use specific entity type filters (`Preference`, `Procedure`, `Requirement`)
- Store new information immediately using `add_memory`
- Follow discovered procedures and respect established preferences

## Git Workflow Requirements

### ⚠️ CRITICAL: Remote Repository Rules ⚠️

**THIS REPOSITORY IS A FORK OF getzep/graphiti**

```
origin = donbr/graphiti (YOUR fork - the ONLY remote)
```

**The upstream remote has been removed to prevent accidental pushes to getzep/graphiti.**

**BEFORE ANY `git push` or `gh pr create` COMMAND:**
1. Run `git remote -v` to verify only `origin` exists
2. If `upstream` or any other remote exists pointing to getzep, DO NOT proceed - ask user first

**BEFORE ANY `gh pr create` COMMAND:**
1. The `gh` CLI creates PRs under THE USER'S GitHub account, not Claude's
2. Any mistakes with PRs reflect on THE USER, not on Claude
3. ALWAYS specify `--repo donbr/graphiti` to ensure PR goes to the correct repo
4. NEVER create PRs against getzep/graphiti without EXPLICIT user permission
5. By default, `gh pr create` may try to create PRs against the parent fork (getzep/graphiti) - ALWAYS use `--repo donbr/graphiti` to prevent this

### Branch Strategy

- **NEVER push directly to `main` branch** - All changes require feature branches and pull requests
- **NEVER push to upstream (`getzep/graphiti`)** - Only push to origin (`donbr/graphiti`)
- **NEVER create PRs against upstream** - PRs stay within the fork unless explicitly requested
- Use descriptive branch names: `feature/description`, `fix/description`, `docs/description`

### Development Process

1. **Create a feature branch** before making any code changes:
   ```bash
   git checkout -b feature/my-feature-name
   ```

2. **Test-Driven Development (TDD)** is required for:
   - All new features
   - All bug fixes
   - Write tests first, then implement the code to pass them

3. **Pull Request workflow**:
   ```bash
   # FIRST: Verify remotes
   git remote -v

   # Push to YOUR fork (origin), never upstream
   git push -u origin feature/my-feature-name

   # Create PR ONLY within the fork (donbr/graphiti)
   # ALWAYS use --repo to be explicit
   gh pr create --repo donbr/graphiti --title "feat: Description" --body "Summary of changes"
   ```

4. **Verify before committing**:
   - Run `make format` to format code
   - Run `make lint` to check for issues
   - Run `make test` to ensure tests pass
   - Test locally with `fastmcp inspect` and `fastmcp dev` for MCP server changes

### What NOT to do

- ❌ Do NOT commit directly to main
- ❌ Do NOT push to upstream (getzep/graphiti)
- ❌ Do NOT create PRs against upstream without explicit permission
- ❌ Do NOT use `gh pr create` without `--repo donbr/graphiti`
- ❌ Do NOT skip tests for "quick fixes"
- ❌ Do NOT commit .env files with real API keys
- ❌ Do NOT forget: gh CLI actions appear as THE USER, not as Claude