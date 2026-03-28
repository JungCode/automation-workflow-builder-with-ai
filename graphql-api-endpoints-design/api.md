# GraphQL API Schema

This document defines the GraphQL API schema for the Automation Workflow Builder platform.

## Table of Contents

- [Custom Scalars](#custom-scalars)
- [Enums](#enums)
- [Types](#types)
- [Inputs](#inputs)
- [Queries](#queries)
- [Mutations](#mutations)

---

## Custom Scalars

Required for handling dynamic JSON configurations and timestamps.

```graphql
scalar JSON
scalar DateTime
```

---

## Enums

### BlueprintDifficulty

Difficulty levels for blueprints.

```graphql
enum BlueprintDifficulty {
  BEGINNER
  INTERMEDIATE
  EXPERT
}
```

### NodeType

Types of nodes in a workflow.

```graphql
enum NodeType {
  trigger
  action
  utility
}
```

### RunStatus

Status of workflow runs and steps.

```graphql
enum RunStatus {
  pending
  running
  success
  failed
  cancelled
  skipped
}
```

### IntegrationProvider

Supported integration providers.

```graphql
enum IntegrationProvider {
  google
  slack
  gmail
  google_sheet
  facebook
}
```

---

## Types

### User

User account information.

```graphql
type User {
  id: ID!
  email: String!
  username: String!
  avatarKey: String
  createdAt: DateTime!
}
```

### AuthPayload

Authentication response payload.

```graphql
type AuthPayload {
  token: String!
  user: User!
}
```

### MetricData

Dashboard metric data point.

```graphql
type MetricData {
  label: String!
  value: Int!
}
```

### Blueprint

Pre-built workflow template.

```graphql
type Blueprint {
  id: ID!
  name: String!
  description: String!
  usageCount: Int!
  hoursSavedPerWeek: Int!
  previewImageKey: String
  difficulty: BlueprintDifficulty!
  category: String!
}
```

### BlueprintConnection

Paginated blueprint results.

```graphql
type BlueprintConnection {
  items: [Blueprint!]!
  totalCount: Int!
}
```

### Folder

Organizational folder for workflows.

```graphql
type Folder {
  id: ID!
  name: String!
  parentId: ID
  ownerUserId: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### Workflow

Main workflow definition.

```graphql
type Workflow {
  id: ID!
  name: String!
  description: String
  folderId: ID
  currentVersionId: ID
  createdByUserId: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
  # Nested resolution: Fetch the current active canvas if needed
  currentVersion: WorkflowVersion
}
```

### WorkflowVersion

Version history for a workflow.

```graphql
type WorkflowVersion {
  id: ID!
  workflowId: ID!
  versionNumber: Int!
  status: String! # 'draft' | 'published'
  nodes: [WorkflowNode!]!
  edges: [WorkflowEdge!]!
  createdAt: DateTime!
}
```

### WorkflowNode

Individual node within a workflow version.

```graphql
type WorkflowNode {
  id: ID!
  nodeType: NodeType!
  providerApp: String!
  actionKey: String!
  label: String!
  positionX: Float
  positionY: Float
  configJson: JSON! # Contains prompts, templates, API configs
  integrationAccountId: ID
}
```

### WorkflowEdge

Connection between workflow nodes.

```graphql
type WorkflowEdge {
  id: ID!
  sourceNodeId: ID!
  targetNodeId: ID!
  sourceHandle: String! # String type to support dynamic AI routing (e.g., "Positive", "Negative")
  targetHandle: String
}
```

### WorkflowRun

Execution record for a workflow instance.

```graphql
type WorkflowRun {
  id: ID!
  workflowId: ID!
  workflowVersionId: ID!
  triggerType: String! # 'manual' | 'webhook' | 'schedule'
  isTestRun: Boolean!
  status: RunStatus!
  startedAt: DateTime!
  finishedAt: DateTime
  errorMessage: String
  inputPayload: JSON
}
```

### WorkflowRunStep

Individual node execution record within a workflow run.

```graphql
type WorkflowRunStep {
  id: ID!
  workflowRunId: ID!
  workflowNodeId: ID!
  executionOrder: Int!
  status: RunStatus!
  inputData: JSON  # The actual data passed into the node during execution
  outputData: JSON # The actual result returned by the node (e.g., AI response)
  startedAt: DateTime!
  finishedAt: DateTime
  durationMs: Int
  errorMessage: String
}
```

### IntegrationAccount

OAuth connection to an external service.

```graphql
type IntegrationAccount {
  id: ID!
  provider: IntegrationProvider!
  accountEmail: String!
  status: String! # 'connected' | 'expired' | 'revoked'
  createdAt: DateTime!
}
```

---

## Inputs

### RegisterInput

Input for user registration.

```graphql
input RegisterInput {
  email: String!
  password: String!
  username: String!
}
```

### LoginInput

Input for user login.

```graphql
input LoginInput {
  email: String!
  password: String!
}
```

### NodeInput

Input for creating or updating workflow nodes.

```graphql
input NodeInput {
  id: ID! # Client generates UUID or passes existing
  nodeType: NodeType!
  providerApp: String!
  actionKey: String!
  label: String!
  positionX: Float
  positionY: Float
  configJson: JSON!
  integrationAccountId: ID
}
```

### EdgeInput

Input for creating or updating workflow edges.

```graphql
input EdgeInput {
  id: ID!
  sourceNodeId: ID!
  targetNodeId: ID!
  sourceHandle: String!
  targetHandle: String
}
```

---

## Queries

### Authentication

```graphql
# Get current authenticated user
me: User!
```

### Dashboard

```graphql
# Get recent workflows for the dashboard
recentWorkflows(limit: Int = 5): [Workflow!]!

# Get workflow metrics for a timeframe
workflowMetrics(timeframe: String!): [MetricData!]!
```

### Blueprints

```graphql
# Get blueprints with optional filters
blueprints(
  category: String
  difficulty: String
  limit: Int = 10
  offset: Int = 0
): BlueprintConnection!
```

### Folders & Workflows

```graphql
# Get folders (root folders if parentId is null)
folders(parentId: ID): [Folder!]!

# Get workflows (root workflows if folderId is null)
workflows(folderId: ID): [Workflow!]!

# Get complete workflow data including versions, nodes, and edges
workflow(id: ID!): Workflow
```

### Logs & Executions

```graphql
# Get workflow runs with optional test run filter
workflowRuns(workflowId: ID!, isTestRun: Boolean, limit: Int = 20): [WorkflowRun!]!

# Get execution steps for a workflow run
runSteps(workflowRunId: ID!): [WorkflowRunStep!]!
```

### Integrations

```graphql
# Get all integration accounts for the current user
integrations: [IntegrationAccount!]!
```

---

## Mutations

### Authentication

```graphql
# Register a new user account
register(input: RegisterInput!): AuthPayload!

# Login with existing credentials
login(input: LoginInput!): AuthPayload!
```

### Folders

```graphql
# Create a new folder
createFolder(name: String!, parentId: ID): Folder!

# Update an existing folder
updateFolder(id: ID!, name: String, parentId: ID): Folder!

# Delete a folder
deleteFolder(id: ID!): Boolean!
```

### Workflows

```graphql
# Create a new workflow
createWorkflow(name: String!, folderId: ID): Workflow!

# Update workflow metadata
updateWorkflow(id: ID!, name: String, folderId: ID): Workflow!

# Delete a workflow
deleteWorkflow(id: ID!): Boolean!
```

### Builder / Canvas

```graphql
# Save the entire canvas state as a draft
saveWorkflowDraft(
  workflowId: ID!
  nodes: [NodeInput!]!
  edges: [EdgeInput!]!
): WorkflowVersion!

# Publish the current draft as the active version
publishWorkflow(workflowId: ID!): WorkflowVersion!
```

### Blueprints

```graphql
# Clone a blueprint into the user's workspace
useBlueprint(blueprintId: ID!, targetFolderId: ID): Workflow!
```

### Execution

```graphql
# Trigger a manual test run from the builder UI
testRunWorkflow(workflowId: ID!, inputPayload: JSON): WorkflowRun!
```

### Integrations

```graphql
# Connect a new integration
connectIntegration(provider: IntegrationProvider!, authCode: String!): IntegrationAccount!

# Revoke an integration connection
revokeIntegration(id: ID!): Boolean!
```
