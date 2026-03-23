# Automation Workflow Builder with AI

## 1. Introduction

### Objective

The system aims to build a platform that enables users to **automate workflows** by visually designing processes using drag-and-drop nodes.

The platform focuses on:

- Reducing repetitive manual tasks
- Allowing users without programming knowledge to build automations
- Connecting multiple services and data sources within a single workflow
- Improving efficiency and minimizing human errors

Additionally, the system integrates an **AI Decision Node**, which allows workflows to automatically make decisions and route the execution path based on input data.

---

## 2. Workflow Builder (Drag-and-Drop Node System)

### Goal

Allow users to design workflows visually by dragging nodes onto a canvas and connecting them to form a data pipeline.

### 2.1 Canvas / Editor

Main features:

- Create, edit, and delete workflows
- Drag and drop nodes onto the canvas
- Connect nodes using edges to represent data flow
- Visualize the execution flow of the workflow
- Support branching logic (IF / Switch)
- Support reusable **sub-flows** for modular workflow design

### 2.2 Node Library

The system provides a set of nodes that users can use to construct workflows.

#### Trigger Nodes

Used to start a workflow.

Examples:

- Webhook trigger
- Google Form submission
- Scheduled trigger

#### Action Nodes

Perform operations in external systems.

Examples:

- Google Sheets (append or update data)
- Gmail (create draft or send email)
- Slack (send notifications)

#### Utility Nodes

Used for data processing and workflow control.

Examples:

- Formatter
- Filter
- Delay
- Merge

These nodes enable users to connect multiple systems and automate cross-platform workflows.

### 2.3 Node Configuration Panel

Each node includes a configuration panel where users can:

- Configure node inputs and outputs
- Map data fields between nodes
- Use variables from previous node outputs

Example:

```
Node A output → Node B input
```

This allows data to flow through the entire workflow pipeline.

### 2.4 Run & Debug

Users can test workflows before activating them.

Features include:

- Run workflow with sample data
- View execution history
- Inspect node outputs
- Detect and analyze errors

Displayed information:

- Node execution order
- Execution time
- Output data
- Error messages

This feature helps users debug and validate their workflow logic.

### 2.5 Versioning & Publish

Workflow management features include:

- Saving multiple workflow versions
- Activating or deactivating workflows

Example states:

```text
Active
Inactive
```

This allows users to safely modify workflows without disrupting running automations.

---

## 3. Decision Nodes

The system provides decision nodes that allow workflows to branch based on conditions.

Two types of decision mechanisms are supported:

1. AI Decision Node
2. Rule-Based Decision Node

### 3.1 AI Decision Node

#### Goal

Allow workflows to automatically make decisions based on input data using artificial intelligence.

#### Input

The AI node receives:

- Current workflow payload
- Requirement or context source (depending on the use case)

Example sources may include:

- Job descriptions
- Business rules
- Policy documents

#### Output Structure

The AI node returns a structured decision object:

```text
decision
confidence
reason
```

Example:

```text
decision: PASS
confidence: 0.87
reason: Candidate has relevant skills and experience
```

#### Routing Logic

Workflow branches are determined by the decision value.

Example:

```text
PASS → Node A
REJECT → Node B
REVIEW → Node C
```

#### AI Node Configuration

Users can configure:

- Allowed decision routes (whitelist)
- Confidence threshold

Example:

```text
confidence < 0.6 → REVIEW
```

- Prompt template
- System rules

#### Optional: Retrieval-Augmented Generation (RAG)

The AI node may use external knowledge sources to improve decision accuracy.

Possible knowledge sources include:

- Job descriptions
- Policy documents
- Knowledge bases

These sources are selected via a **Knowledge Source** configuration.

### 3.2 Rule-Based Decision Node

The system also supports manual decision logic using rule-based conditions.

#### Main Functions

Users can define logic such as:

##### IF / ELSE

```text
IF experience > 2 years
→ PASS
ELSE
→ REJECT
```

##### Switch Conditions

```text
role = developer → Dev flow
role = designer → Design flow
```

##### Rule Sets

Multiple conditions can be combined using logical operators.

Example:

```text
experience > 2
AND skill = Python
```

#### Output

The rule-based node returns the same output format as the AI node:

```text
decision
```

This ensures that downstream nodes can use a **consistent routing mechanism**.

---

## 4. MVP Acceptance Criteria

The system meets MVP requirements when:

- Users can create a workflow using drag-and-drop nodes
- The workflow includes at least **one branching condition**
- The workflow executes **end-to-end successfully**
- Execution logs are stored and viewable
- Decision nodes return a `decision` value
- Workflow routing behaves correctly based on the decision output

---
