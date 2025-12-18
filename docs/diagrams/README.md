# Architecture Diagram Quick Reference

This document provides a quick index of all architecture diagrams available for the MCP Python SDK.

## Main Architecture Documents

### 1. [Architecture Overview](../architecture.md)

Comprehensive overview of the MCP Python SDK architecture including:

- **High-Level Architecture Diagram**: Shows all major components and their relationships
  - Client Applications (LLM Apps, Claude Desktop, Custom Clients)
  - Transport Layer (stdio, SSE, HTTP, WebSocket)
  - Protocol Layer (Types, Session Management, Auth)
  - Server Implementation (FastMCP, Low-Level Server)
  - Client Implementation (ClientSession, OAuth)
  - Shared Components

- **Server Request Flow**: Sequence diagram showing how requests flow through the server
- **Client Connection Flow**: Sequence diagram showing client initialization and tool calls

### 2. [Protocol Primitives](protocol-primitives.md)

Detailed diagrams of the three core MCP primitives:

- **The Three Primitives**: Tools, Resources, and Prompts with their control flows
- **Primitive Interactions**: Sequence diagram showing how primitives interact
- **Data Flow Through SDK**: How requests move through SDK layers
- **Tool Execution Detail**: State machine for tool lifecycle
- **Resource Lifecycle**: State machine for resource operations
- **Prompt Processing**: Flowchart for prompt handling
- **Context Injection System**: How context is created and injected
- **Structured Output Flow**: Type-based output formatting logic
- **Authentication Flow**: OAuth 2.1 authorization sequence

## Diagram Types

### Component Diagrams
Show the static structure and relationships between components:
- High-Level Architecture (architecture.md)
- The Three Primitives (protocol-primitives.md)
- Context Injection System (protocol-primitives.md)

### Sequence Diagrams
Show interactions over time:
- Server Request Flow (architecture.md)
- Client Connection Flow (architecture.md)
- Primitive Interactions (protocol-primitives.md)
- Authentication Flow (protocol-primitives.md)

### State Machines
Show state transitions:
- Tool Execution Detail (protocol-primitives.md)
- Resource Lifecycle (protocol-primitives.md)

### Flowcharts
Show decision logic and processing:
- Data Flow Through SDK (protocol-primitives.md)
- Prompt Processing (protocol-primitives.md)
- Structured Output Flow (protocol-primitives.md)

## Key Concepts Illustrated

### Transport Layer
- Multiple transport options (stdio, SSE, HTTP, WebSocket)
- Transport-agnostic protocol design
- See: High-Level Architecture

### Server APIs
- FastMCP decorator-based high-level API
- Low-level server for advanced use cases
- See: High-Level Architecture, Server Request Flow

### Protocol Primitives
- **Tools**: Model-controlled actions
- **Resources**: Application-controlled data
- **Prompts**: User-controlled templates
- See: The Three Primitives, Primitive Interactions

### Context System
- Automatic context injection
- Access to session, server, and lifecycle resources
- See: Context Injection System

### Type Safety
- Pydantic-based validation
- Automatic schema generation
- Structured output support
- See: Structured Output Flow

### Authentication
- OAuth 2.1 client and server support
- Token management and validation
- See: Authentication Flow

## Reading Guide

**For new users:**
1. Start with [Architecture Overview](../architecture.md) - High-Level Architecture
2. Read about [Protocol Primitives](protocol-primitives.md) - The Three Primitives
3. Review [Architecture Overview](../architecture.md) - Server Request Flow

**For server developers:**
1. Review [Server Request Flow](../architecture.md)
2. Study [Tool Execution Detail](protocol-primitives.md)
3. Understand [Context Injection System](protocol-primitives.md)
4. Learn [Structured Output Flow](protocol-primitives.md)

**For client developers:**
1. Review [Client Connection Flow](../architecture.md)
2. Study [Primitive Interactions](protocol-primitives.md)
3. Understand [Authentication Flow](protocol-primitives.md)

**For advanced users:**
1. Study all diagrams in [Architecture Overview](../architecture.md)
2. Review all state machines in [Protocol Primitives](protocol-primitives.md)
3. Understand module structure and extension points

## Diagram Conventions

- **Green boxes**: Core implementation components
- **Blue boxes**: Client-side components
- **Yellow boxes**: Protocol/type definitions
- **Pink boxes**: User/application interfaces
- **Solid arrows**: Data/control flow
- **Dashed arrows**: Optional/reference relationships

## Additional Resources

- [Main README](../../README.md) - Quick start and examples
- [API Reference](../api.md) - Detailed API documentation
- [MCP Specification](https://modelcontextprotocol.io/specification/latest) - Protocol specification
- [Example Servers](../../examples/) - Complete working examples
