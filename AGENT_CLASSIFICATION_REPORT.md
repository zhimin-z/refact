# Refact Agent Classification Report

This report provides a comprehensive classification of the Refact AI agent according to the SWE Agent Taxonomy framework across multiple dimensions: architecture, cognition (workflow, planning, reasoning), memory, and tools.

---

## Agent Architecture

**Labels:** SINGLE-AGENT, MULTI-AGENT-HIERARCHICAL

**Justification:**

The Refact agent primarily operates as a **SINGLE-AGENT** system where a single agent independently handles development tasks without peer-to-peer coordination or debate mechanisms. However, it also demonstrates **MULTI-AGENT-HIERARCHICAL** characteristics through its subagent delegation mechanism. The agent can delegate focused sub-tasks to specialized worker subagents using the `subagent()` tool (implemented in `src/tools/tool_subagent.rs`), where the main agent acts as a coordinator that monitors progress and integrates results. The subagent system prompt explicitly states: "You are a focused sub-agent executing a specific task. You have been delegated this task by a parent agent." This creates a manager-worker structure where the parent agent delegates, monitors, and integrates specialized work.

---

## Agent Cognition

### Workflow Patterns

**Labels:** ITERATIVE

**Justification:**

The Refact agent follows an **ITERATIVE** workflow pattern, implementing a continuous loop of trial and adjustment. The agent executes actions through tool calls, observes results, and uses feedback to plan subsequent steps. This is evidenced by the tool execution cycle in `tools_execute.rs` where the agent processes tool calls, receives tool responses, and continues the conversation loop. The architecture supports the classic Thought → Action → Observation cycle, where the agent iteratively refines its approach based on immediate feedback from tool executions (file operations, code analysis, searches, etc.) until task completion.

### Planning Strategies

**Labels:** HIERARCHICAL, INTERACTIVE, REPLANNING

**Justification:**

The Refact agent employs multiple planning strategies:

1. **HIERARCHICAL**: The `tool_strategic_planning.rs` implements strategic planning by breaking down complex problems into detailed steps. The tool uses a reasoning model (o3 mini) to "identify and solve the problem" and "make a couple of alternative ways to solve the problem." The subagent tool also demonstrates hierarchical decomposition by delegating focused sub-tasks.

2. **INTERACTIVE**: The agent integrates human-in-the-loop approval mechanisms. The INTEGRATIONS.md file describes command confirmation patterns where certain operations require user approval before execution. The patch tool requires user approval before applying code changes, as documented in the overview: "The Agent provides the code changes it wants to make" and "You approve the 'Patches' before they're applied."

3. **REPLANNING**: The strategic planning tool explicitly creates "alternative ways to solve the problem, if the initial solution doesn't work," demonstrating dynamic adjustment capabilities. The agent can adapt its approach based on execution results.

### Reasoning Techniques

**Labels:** CHAIN-OF-THOUGHT, REACT, TOOL-COMPOSITION, PROGRAM-AIDED, ZERO-SHOT

**Justification:**

The Refact agent employs multiple reasoning techniques:

1. **CHAIN-OF-THOUGHT**: The agent supports reasoning through extended thinking modes. Multiple model providers support reasoning capabilities (OpenAI reasoning_effort, Anthropic thinking, Qwen enable_thinking) as seen in `call_validation.rs`. The strategic planning tool uses explicit step-by-step problem decomposition.

2. **REACT**: The agent implements the ReAct pattern (Reasoning + Acting) through its tool-calling loop. As documented in tools.md, the workflow includes "Understanding Phase → Planning Phase → Execution Phase," where the agent interleaves thinking (via the `think` tool using o3 mini) with actions (tool executions) and observations (tool results).

3. **TOOL-COMPOSITION**: The agent strategically chains multiple tools together. The tools.md documentation describes how "the Agent combines these tools strategically" in a multi-phase workflow, using outputs from tools like `tree` and `locate` to inform subsequent `cat` and `patch` operations.

4. **PROGRAM-AIDED**: The agent can execute shell commands and scripts through integration tools (shell, pdb debugger, Docker) as documented in INTEGRATIONS.md and the integrations directory, using code execution to solve problems.

5. **ZERO-SHOT**: The agent can solve problems directly using its training without requiring special reasoning frameworks for simpler tasks, as evidenced by its support for various completion and chat modes.

---

## Agent Memory

**Labels:** EPHEMERAL, PERSISTENT, EPISODIC, VECTOR-DB, MULTI-TIERED, SEMANTIC, PROCEDURAL

**Justification:**

The Refact agent implements a sophisticated multi-layered memory system:

1. **EPHEMERAL**: Each chat session maintains conversational context through the message history within that session, but this context is session-specific.

2. **PERSISTENT**: The `memories.rs` module implements long-term memory storage using SQLite (`memories.sqlite`). The code includes memory migration functionality and cloud-based memory storage through `memories_add()`, enabling knowledge persistence across sessions and interactions.

3. **EPISODIC**: The memory system stores specific interaction records with metadata. The `MemoRecord` structure in `memories.rs` includes `iknow_id`, `iknow_tags`, and `iknow_memory` fields, suggesting structured storage of past events and their context.

4. **VECTOR-DB**: The `vecdb` module (with files like `vdb_sqlite.rs`, `vdb_thread.rs`, `vdb_highlev.rs`) implements a vector database for semantic search. The agent uses embeddings to convert code into numerical vectors, enabling semantic similarity search through the `search` tool as described in tools.md: "Find similar pieces of code or text using vector database."

5. **MULTI-TIERED**: The system manages different memory tiers - immediate session context (ephemeral working memory), vector database for fast semantic retrieval, and persistent SQLite storage for long-term knowledge. The AST indexer thread maintains separate structural indexes for code understanding.

6. **SEMANTIC**: The knowledge tools (`tool_knowledge.rs`, `tool_create_knowledge.rs`, `tool_create_memory_bank.rs`) implement semantic knowledge storage and retrieval. The `knowledge` tool "fetches successful trajectories" using "vector similarity search" based on semantic search keys. The system stores structured conceptual knowledge about tools, project components, objectives, and frameworks, enabling sophisticated reasoning about relationships.

7. **PROCEDURAL**: The knowledge system stores and retrieves "successful trajectories to help you accomplish your task" as documented in the `knowledge` tool description. This represents operational expertise - how-to knowledge about successfully completing specific types of tasks, which can be recalled and applied to similar future tasks.

---

## Agent Tools

**Labels:** FILE-MANAGEMENT, CODE-EDITING, STRUCTURAL-RETRIEVAL, EMBEDDING-RETRIEVAL, VERSION-CONTROL, PYTHON-TOOLS, DATABASE-TOOLS, SYSTEM-UTILITIES, SHELL-SCRIPTING, WEB-TOOLS, DEBUGGING-TOOLS, CONTAINER-TOOLS

**Additional Note:** The agent also supports **Model Context Protocol (MCP)** integration, which enables connection to external MCP servers to extend tool capabilities dynamically. MCP servers can be configured via stdio or SSE protocols as documented in `integrations/mcp/` and `docs/features/autonomous-agent/integrations/mcp.md`.

**Justification:**

The Refact agent provides extensive tool capabilities across multiple categories:

1. **FILE-MANAGEMENT**: Basic file operations through tools like `cat` (read files), `tree` (directory structure), `rm` (delete), `mv` (move/rename) as implemented in the tools directory (`tools_list.rs` lines 84-101).

2. **CODE-EDITING**: Specialized code modification through file_edit module tools including `ToolCreateTextDoc`, `ToolUpdateTextDoc`, and `ToolUpdateTextDocByLines` for precise code modifications (`tools_list.rs` lines 94-101).

3. **STRUCTURAL-RETRIEVAL**: Advanced AST-based code comprehension through `definition` tool (`ToolAstDefinition`) for symbol definitions as shown in tools.md. The AST module supports Java, JavaScript, TypeScript, Python, and Rust with syntax-aware search capabilities.

4. **EMBEDDING-RETRIEVAL**: Vector database semantic search through the `search` tool (`ToolSearch`), using embeddings to find similar code patterns as documented in tools.md and implemented via the vecdb module.

5. **VERSION-CONTROL**: Git integration through `integr_github.rs` and `integr_gitlab.rs` modules, with experimental GitHub CLI (`gh` command) support as noted in INTEGRATIONS.md.

6. **PYTHON-TOOLS**: Python debugger integration through `integr_pdb.rs` for runtime debugging capabilities (marked experimental in README.md).

7. **DATABASE-TOOLS**: MySQL and PostgreSQL integrations (`integr_mysql.rs`, `integr_postgres.rs`) for database operations as documented in the integrations directory.

8. **SYSTEM-UTILITIES**: Shell command execution through `integr_shell.rs` and command-line service integration (`integr_cmdline_service.rs`), providing access to process control and system operations.

9. **SHELL-SCRIPTING**: Full shell command support through the shell integration, allowing execution of arbitrary bash commands and scripts for automation.

10. **WEB-TOOLS**: Web page fetching through the `web` tool (`ToolWeb` in `tool_web.rs`) for reading documentation and online resources, plus Chrome browser integration (`integr_chrome.rs`) for advanced web interaction.

11. **DEBUGGING-TOOLS**: Python debugger (pdb) integration through `integr_pdb.rs` for interactive debugging sessions and runtime introspection.

12. **CONTAINER-TOOLS**: Docker integration (experimental) through the `docker` subdirectory in integrations, enabling container management and orchestration.

---

## Summary

The Refact agent is a sophisticated autonomous software development agent that combines:

- **Architecture**: Primarily single-agent with hierarchical multi-agent capabilities through subagent delegation
- **Workflow**: Iterative action-observation-refinement cycles
- **Planning**: Hierarchical decomposition with interactive human approval and replanning capabilities
- **Reasoning**: Multi-modal reasoning including chain-of-thought, ReAct, tool composition, and program-aided techniques
- **Memory**: Multi-tiered system combining ephemeral session context, persistent storage, episodic memory, and vector database for semantic retrieval
- **Tools**: Comprehensive toolset spanning 12 categories from file management to debugging, databases, and containerization

This classification demonstrates that Refact is a full-featured autonomous agent designed for end-to-end software engineering tasks with sophisticated cognitive capabilities, flexible planning, and extensive environmental interaction through diverse tool integrations.
