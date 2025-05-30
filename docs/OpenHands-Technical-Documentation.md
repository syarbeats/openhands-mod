# OpenHands Technical Documentation

## Introduction

OpenHands is an open-source AI agent framework that enables users to build, deploy, and interact with AI agents. The framework provides a flexible and extensible architecture for developing AI-powered applications with various LLM providers. This document explains the system architecture, workflow, and how to extend the system with new model providers.

## System Architecture

OpenHands follows a modular architecture that separates concerns into distinct components, allowing for flexibility and extensibility.

```
+----------------------+     +----------------------+     +----------------------+
|                      |     |                      |     |                      |
|  Frontend UI (React) |<--->|  Backend Server      |<--->|  Agent Controller    |
|                      |     |  (FastAPI)           |     |                      |
+----------------------+     +----------------------+     +----------------------+
                                       ^                           ^
                                       |                           |
                                       v                           v
+----------------------+     +----------------------+     +----------------------+
|                      |     |                      |     |                      |
|  Storage System      |<--->|  Event Stream        |<--->|  LLM Interface       |
|                      |     |                      |     |                      |
+----------------------+     +----------------------+     +----------------------+
                                       ^                           ^
                                       |                           |
                                       v                           v
+----------------------+     +----------------------+
|                      |     |                      |
|  Runtime Environment |<--->|  Security Layer      |
|                      |     |                      |
+----------------------+     +----------------------+
```

### Core Components

1. **Frontend UI**: React-based user interface for interacting with agents
2. **Backend Server**: FastAPI server that handles requests and manages sessions
3. **Agent Controller**: Orchestrates the agent's actions and responses
4. **Event Stream**: Manages event flow between components
5. **LLM Interface**: Communicates with various LLM providers
6. **Runtime Environment**: Executes agent actions in a sandboxed environment
7. **Security Layer**: Ensures safe execution of agent actions
8. **Storage System**: Manages persistent data and conversation history

## Activity Diagram

The following activity diagram illustrates the typical workflow when a user interacts with an agent:

```
┌─────────┐          ┌─────────┐          ┌─────────┐          ┌─────────┐          ┌─────────┐
│  User   │          │Frontend │          │Backend  │          │ Agent   │          │  LLM    │
│         │          │         │          │         │          │Controller│          │Provider │
└────┬────┘          └────┬────┘          └────┬────┘          └────┬────┘          └────┬────┘
     │                     │                    │                    │                    │
     │  Send Message       │                    │                    │                    │
     │─────────────────────>                    │                    │                    │
     │                     │  Forward Message   │                    │                    │
     │                     │───────────────────>│                    │                    │
     │                     │                    │  Create Session    │                    │
     │                     │                    │───────────────────>│                    │
     │                     │                    │                    │  Process Message   │
     │                     │                    │                    │───────────────────>│
     │                     │                    │                    │                    │
     │                     │                    │                    │<───────────────────│
     │                     │                    │                    │  Response          │
     │                     │                    │                    │                    │
     │                     │                    │                    │  Generate Action   │
     │                     │                    │<───────────────────│                    │
     │                     │                    │  Action Result     │                    │
     │                     │                    │                    │                    │
     │                     │                    │  Execute Action    │                    │
     │                     │                    │───────────────────>│                    │
     │                     │                    │                    │                    │
     │                     │                    │<───────────────────│                    │
     │                     │                    │  Action Result     │                    │
     │                     │  Update UI         │                    │                    │
     │                     │<───────────────────│                    │                    │
     │  View Response      │                    │                    │                    │
     │<─────────────────────                    │                    │                    │
     │                     │                    │                    │                    │
```

## Sequence Diagram for Agent Interaction

```
┌─────┐          ┌─────────┐          ┌────────┐          ┌─────────┐          ┌────┐
│User │          │Frontend │          │Backend │          │Agent    │          │LLM │
│     │          │         │          │        │          │Controller│          │    │
└──┬──┘          └────┬────┘          └───┬────┘          └────┬────┘          └─┬──┘
   │                  │                    │                    │                 │
   │ Send Message     │                    │                    │                 │
   │─────────────────>│                    │                    │                 │
   │                  │ WebSocket Message  │                    │                 │
   │                  │───────────────────>│                    │                 │
   │                  │                    │                    │                 │
   │                  │                    │ Initialize Agent   │                 │
   │                  │                    │───────────────────>│                 │
   │                  │                    │                    │ Request Completion
   │                  │                    │                    │────────────────>│
   │                  │                    │                    │                 │
   │                  │                    │                    │ Response        │
   │                  │                    │                    │<────────────────│
   │                  │                    │                    │                 │
   │                  │                    │                    │ Process Response│
   │                  │                    │                    │─┐               │
   │                  │                    │                    │ │               │
   │                  │                    │                    │<┘               │
   │                  │                    │                    │                 │
   │                  │                    │ Action/Observation │                 │
   │                  │                    │<───────────────────│                 │
   │                  │                    │                    │                 │
   │                  │ WebSocket Update   │                    │                 │
   │                  │<───────────────────│                    │                 │
   │ UI Update        │                    │                    │                 │
   │<─────────────────│                    │                    │                 │
   │                  │                    │                    │                 │
```

## Component Details

### 1. Session Management

The Session class in `openhands/server/session/session.py` manages user interactions with agents. It:
- Handles WebSocket connections
- Initializes agents with appropriate configurations
- Manages the event stream for communication
- Handles errors and timeouts

### 2. Agent Controller

The AgentController in `openhands/controller/agent_controller.py` orchestrates agent behavior:
- Processes user input
- Communicates with LLM providers
- Generates actions based on LLM responses
- Manages agent state and handles errors

### 3. LLM Interface

The LLM class in `openhands/llm/llm.py` provides a unified interface to various LLM providers:
- Handles communication with different model providers
- Manages retries, error handling, and token counting
- Converts between different function calling formats
- Supports various model-specific features

### 4. Runtime Environment

The runtime implementations in `openhands/runtime/impl/` provide sandboxed environments for executing agent actions:
- Support for local and remote runtimes
- Secure execution of code
- Handling of file operations and command execution
- Integration with various plugins

## Adding a New Model Provider

OpenHands uses LiteLLM as its abstraction layer for LLM providers, making it relatively straightforward to add new model providers. Here's how to add a new provider:

### Step 1: Understand the LLM Integration

OpenHands primarily uses the `LLM` class in `openhands/llm/llm.py` to interact with language models. This class uses LiteLLM to handle most provider-specific details.

### Step 2: Check LiteLLM Support

First, check if LiteLLM already supports the model provider you want to add. If it does, most of the integration work is already done.

### Step 3: Update Configuration Support

1. Modify `openhands/core/config/llm_config.py` to add support for your new provider's configuration options:

```python
class LLMConfig(BaseModel):
    # Add provider-specific fields if needed
    your_provider_specific_field: str | None = None
```

### Step 4: Register Model Capabilities

Update the model capability lists in `openhands/llm/llm.py` to include your new model:

```python
# Add models that support function calling
FUNCTION_CALLING_SUPPORTED_MODELS = [
    # existing models...
    'your-new-model',
]

# Add models that support cache prompt
CACHE_PROMPT_SUPPORTED_MODELS = [
    # existing models...
    'your-new-model',
]

# Add models that support reasoning effort
REASONING_EFFORT_SUPPORTED_MODELS = [
    # existing models...
    'your-new-model',
]
```

### Step 5: Handle Provider-Specific Logic

If your provider requires special handling that LiteLLM doesn't cover, add provider-specific code to the LLM class in `openhands/llm/llm.py`:

```python
def _prepare_completion_kwargs(self, messages, tools, tool_choice, reasoning_effort):
    # ... existing code ...
    
    # Add provider-specific handling
    if self._is_provider_x_model():
        # Special handling for your provider
        kwargs["special_parameter"] = "provider-specific-value"
        
    return kwargs
```

### Step 6: Test Your Integration

1. Create a basic configuration that uses your new provider:
   
```toml
# config.toml
[core]
workspace_base = "./workspace"

[llm]
model = "your-new-model"
api_key = "your-api-key"
base_url = "https://your-provider-api-endpoint.com" # if needed
```

2. Run OpenHands with this configuration to test your integration.

### Step 7: Document Your Integration

Add documentation for your new provider in the appropriate documentation files, explaining:
- Required configuration
- Supported features
- Any limitations or special considerations
- Example configuration

## Advanced Topics

### Event System

OpenHands uses an event-based architecture to decouple components and enable flexible interaction patterns. The core event types are:

- **Actions**: Represent agent intents (e.g., running commands, sending messages)
- **Observations**: Represent the results of actions and environmental changes

Events flow through the `EventStream`, which distributes them to registered subscribers, forming a publish-subscribe pattern.

### Security Model

OpenHands implements a multi-layered security approach:
- Sandboxed execution environments (Docker containers)
- Action validation before execution
- Rate limiting and budget constraints
- Optional confirmation mode requiring user approval

### Memory and Context Management

Agents maintain context through:
- Short-term memory (conversation history)
- Long-term memory (vector database storage)
- File system access within their sandbox

## Common Issues and Troubleshooting

### Timeout on Session Creation

Issue: "Error creating agent_session: timed out when starting a conversation"

Possible causes:
1. Network connectivity issues with LLM provider
2. Docker/container setup problems
3. Resource constraints

Solutions:
1. Check network connectivity to LLM provider
2. Ensure Docker is running properly
3. Check system resources (memory, CPU)
4. Review logs for additional error details
5. Increase timeout values in configuration

### Model Provider Integration Issues

If you encounter problems with a specific model provider:

1. Check API credentials and permissions
2. Verify model naming convention matches provider requirements
3. Check for rate limiting or quota issues
4. Enable debug logging for more detailed error information

## Resources

- GitHub Repository: [OpenHands](https://github.com/openhands/openhands)
- Documentation: [OpenHands Documentation](https://docs.openhands.ai)
- LiteLLM Documentation: [LiteLLM](https://litellm.vercel.app)
