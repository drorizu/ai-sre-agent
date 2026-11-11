
# SRE Agent – Request and Investigation Workflow

## Overview

The SRE Agent processes investigation requests submitted by Site Reliability Engineers (SREs). It uses an LLM-powered Agent Orchestrator to understand user questions, select appropriate skills, and coordinate their execution through the Investigation Orchestrator.

The architecture separates decision-making, function execution, and investigation lifecycle management into distinct components.

## Architecture Components

| Component | Responsibility |
|---|---|
| **Frontend** | Provides the interface through which SREs submit investigation requests and view results. |
| **FastAPI** | Receives requests from the frontend and forwards them to the Agent Orchestrator. |
| **Agent Orchestrator** | Coordinates interactions between the LLM and Investigation Orchestrator. Retrieves available skills and manages agent context. |
| **LLM** | Analyses user questions, selects appropriate skills, extracts function arguments, and generates natural-language responses. |
| **Investigation Orchestrator** | Executes requested functions, manages investigation state, and tracks the investigation lifecycle. |
| **Skills** | Provide individual functions and capabilities used to investigate infrastructure and application issues. |

---

## Request and Investigation Workflow

The following sequence diagram illustrates how an investigation request travels through the system, from the initial user question to the final response.

```mermaid
sequenceDiagram
    autonumber

    actor SRE
    participant FE as Frontend
    participant API as FastAPI
    participant AO as Agent Orchestrator
    participant LLM
    participant IO as Investigation Orchestrator
    participant SK as Skills

    SRE->>FE: Submit question
    FE->>API: Investigation request
    API->>AO: Forward request

    Note over AO,SK: Skill discovery

    AO->>SK: Retrieve available skills
    SK-->>AO: Skill definitions

    Note over AO,LLM: Skill selection and argument extraction

    AO->>LLM: User question + available skills
    LLM-->>AO: Selected function + arguments

    Note over AO,IO: Function execution

    AO->>IO: Execute function call
    IO->>SK: Invoke selected skill
    SK-->>IO: Execution result
    IO->>IO: Update investigation state
    IO-->>AO: Execution result + investigation state

    Note over AO,LLM: Result analysis

    AO->>LLM: Updated context + investigation results
    LLM-->>AO: Natural-language response

    Note over AO,IO: Investigation completion

    AO->>IO: Complete investigation
    IO->>IO: Set status to COMPLETED
    IO-->>AO: Final investigation state

    AO->>API: Final response
    API->>FE: Deliver response
    FE->>SRE: Display response
```

---

## Workflow Description

### 1. User Submits a Question

The SRE submits an investigation request through the frontend.

### 2. FastAPI Forwards the Request

FastAPI receives the request and forwards it to the Agent Orchestrator.

### 3. Agent Orchestrator Retrieves Available Skills

The Agent Orchestrator retrieves the available skills, including their descriptions, function definitions, and required parameters.

It provides these definitions, together with the user's question, to the LLM.

### 4. LLM Selects a Skill and Extracts Arguments

The LLM analyses the user's question, identifies the appropriate skill, extracts the required arguments, and determines which function needs to be executed.

### 5. LLM Returns a Function Call

The LLM returns a structured function call containing the selected function and its arguments to the Agent Orchestrator.

**Example:**

```python
check_cpu(server="server1")
```

### 6. Agent Orchestrator Forwards the Function Call

The Agent Orchestrator forwards the selected function and its arguments to the Investigation Orchestrator.

### 7. Investigation Orchestrator Executes the Function

The Investigation Orchestrator:

- Invokes the requested skill function.
- Awaits its execution.
- Retrieves the execution result.
- Updates and saves the investigation state.
- Returns the execution result and investigation state to the Agent Orchestrator.

### 8. Agent Orchestrator Updates the Context

The Agent Orchestrator incorporates the returned results into its context and sends the updated context to the LLM.

### 9. LLM Generates the Response

The LLM analyses the investigation results and generates a natural-language response based on the available information.

It returns the response to the Agent Orchestrator.

### 10. Investigation Is Marked as Completed

Once all required investigation functions have finished executing and the investigation has reached its completion criteria, the Investigation Orchestrator updates the investigation status to `COMPLETED`.

### 11. Response Is Delivered to the Frontend

The Agent Orchestrator returns the final response to FastAPI.

FastAPI forwards it to the frontend, where it is displayed to the SRE.

---

## Investigation Execution Loop

An investigation is not necessarily limited to a single function call.

Depending on the user's question and the results returned by individual skills, the LLM may need to request multiple function executions before producing a final response.

**Example: Investigating a Server Performance Issue**

1. Check CPU utilization.
2. Check memory utilization.
3. Retrieve application logs.
4. Check service health.
5. Analyse and correlate the results.
6. Generate a final investigation report.

The Agent Orchestrator coordinates this iterative interaction between the LLM and Investigation Orchestrator until sufficient information is available to produce a final response.

---

## Investigation State Management

The system maintains investigation state throughout the execution lifecycle.

The responsibilities are separated as follows:

- **Investigation Orchestrator:** Owns the investigation execution state, function results, and lifecycle status.
- **Agent Orchestrator:** Maintains the agent context and incorporates investigation results into subsequent LLM interactions.

This separation ensures that investigation execution and LLM reasoning remain independent while allowing them to work together.

### Investigation Lifecycle

| Status | Description |
|---|---|
| `PENDING` | Investigation request has been received but execution has not started. |
| `RUNNING` | Investigation is currently executing. |
| `COMPLETED` | Investigation has successfully finished. |
| `FAILED` | Investigation execution has failed. |

---

## Summary

The SRE Agent architecture provides a modular approach to automated infrastructure investigations.

By separating the responsibilities of the Agent Orchestrator, LLM, and Investigation Orchestrator, the system enables:

- **Dynamic skill selection:** The LLM selects appropriate skills based on the user's question.
- **Automated execution:** The Investigation Orchestrator executes the selected functions.
- **Iterative investigations:** Multiple skills can be executed to gather and correlate information.
- **State management:** Investigation progress and execution results are tracked throughout the lifecycle.
- **Natural-language responses:** The LLM converts investigation results into understandable responses for SREs.

