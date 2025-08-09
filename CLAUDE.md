# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About Swarm

Swarm is OpenAI's experimental, educational multi-agent orchestration framework. It focuses on making agent coordination and execution lightweight, highly controllable, and easily testable through two primitive abstractions: `Agent`s and **handoffs**.

**Important Note**: Swarm has been replaced by the [OpenAI Agents SDK](https://github.com/openai/openai-agents-python) for production use. This repository remains as an educational resource.

## Development Commands

### Installation
```bash
pip install git+https://github.com/openai/swarm.git
```

### Running Tests
```bash
python -m pytest tests/
```

### Running Examples
```bash
# Basic examples
python examples/basic/bare_minimum.py
python examples/basic/function_calling.py
python examples/basic/agent_handoff.py

# Complex examples (require additional setup)
python examples/airline/main.py
python examples/support_bot/main.py
python examples/personal_shopper/main.py
```

### Code Quality
The project uses autopep8 for formatting with these settings:
```bash
autopep8 --max-line-length=120 --ignore=E501,W6 --in-place --recursive --aggressive=3
```

## Architecture Overview

### Core Components

1. **Swarm Client (`swarm/core.py`)**: Main orchestration engine that handles agent execution loops
   - Manages conversation flow between agents
   - Handles tool execution and agent handoffs
   - Supports both streaming and non-streaming modes

2. **Agent (`swarm/types.py`)**: Core abstraction representing an AI agent
   - Contains instructions (system prompt)
   - Has associated functions/tools
   - Can transfer control to other agents

3. **Response & Result**: Data structures for managing execution results
   - `Response`: Contains messages, final agent, and context variables
   - `Result`: Encapsulates function return values with optional agent handoff

### Key Patterns

**Agent Definition**:
```python
from swarm import Swarm, Agent

agent = Agent(
    name="Agent Name",
    instructions="System prompt for the agent",
    functions=[function1, function2],  # Tools the agent can call
    model="gpt-4o"  # OpenAI model to use
)
```

**Agent Handoffs**:
```python
def transfer_to_other_agent():
    """Transfer control to another agent"""
    return other_agent  # Return agent instance

# Or with context variables:
def transfer_with_context():
    return Result(
        value="Transfer complete",
        agent=other_agent,
        context_variables={"key": "value"}
    )
```

**Context Variables**: Shared state passed between agents and functions
- Available to functions via `context_variables` parameter
- Automatically injected if function signature includes it
- Used for maintaining conversation state across handoffs

### Execution Flow

The `client.run()` method implements this loop:
1. Get completion from current agent
2. Execute tool calls and append results  
3. Switch agent if necessary (via handoff)
4. Update context variables
5. Continue until no more function calls

## Project Structure

- `swarm/`: Core framework code
  - `core.py`: Main Swarm client and execution logic
  - `types.py`: Agent, Response, and Result classes
  - `util.py`: Helper functions for JSON schema generation
  - `repl/`: Interactive demo utilities
- `examples/`: Educational examples demonstrating patterns
  - `basic/`: Fundamental concepts (handoffs, functions, context)
  - `airline/`: Multi-agent customer service system
  - `support_bot/`: Help desk with knowledge retrieval
  - `personal_shopper/`: E-commerce agent with database integration
- `tests/`: Unit tests with mock OpenAI client

## Testing Approach

Tests use a mock OpenAI client (`tests/mock_client.py`) to avoid API calls during testing. Key test files:
- `test_core.py`: Core Swarm functionality
- `test_util.py`: Utility functions

Run evaluations in example directories to test multi-agent scenarios:
```bash
cd examples/airline
python evals/function_evals.py
```

## Dependencies

Core dependencies (from `setup.cfg`):
- `openai>=1.33.0`: OpenAI API client
- `numpy`: Numerical operations
- `requests`: HTTP requests
- `tqdm`: Progress bars
- `instructor`: Structured LLM outputs
- `pytest`: Testing framework
- `pre-commit`: Git hooks

## Development Notes

- Framework is stateless between `client.run()` calls
- Uses OpenAI Chat Completions API (not Assistants API)
- Automatically converts Python functions to JSON schemas for tool calling
- Supports dynamic instructions via callable functions
- Context variables enable state sharing without global variables
- Agent handoffs enable complex multi-step workflows