# ---------------------------------------------------------
# CUSTOM SCALARS
# ---------------------------------------------------------
# Required for handling dynamic JSON configurations and timestamps
scalar JSON
scalar DateTime

# ---------------------------------------------------------
# ENUMS
# ---------------------------------------------------------
enum BlueprintDifficulty {
  BEGINNER
  INTERMEDIATE
  EXPERT
}

enum NodeType {
  trigger
  action
  utility
}

enum RunStatus {
  pending
  running
  success
  failed
  cancelled
  skipped
}

enum IntegrationProvider {
  google
  slack
  gmail
  google_sheet
  facebook
}

# ---------------------------------------------------------
# TYPES (Output Definitions)
# ---------------------------------------------------------
type User {
  id: ID!
  email: String!
  username: String!
  avatarKey: String
  createdAt: DateTime!
}

type AuthPayload {
  token: String!
  user: User!
}

type MetricData {
  label: String!
  value: Int!
}

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

type BlueprintConnection {
  items: [Blueprint!]!
  totalCount: Int!
}

type Folder {
  id: ID!
  name: String!
  parentId: ID
  ownerUserId: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
}

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

type WorkflowVersion {
  id: ID!
  workflowId: ID!
  versionNumber: Int!
  status: String! # 'draft' | 'published'
  nodes: [WorkflowNode!]!
  edges: [WorkflowEdge!]!
  createdAt: DateTime!
}

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

type WorkflowEdge {
  id: ID!
  sourceNodeId: ID!
  targetNodeId: ID!
  sourceHandle: String! # String type to support dynamic AI routing (e.g., "Positive", "Negative")
  targetHandle: String
}

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

type IntegrationAccount {
  id: ID!
  provider: IntegrationProvider!
  accountEmail: String!
  status: String! # 'connected' | 'expired' | 'revoked'
  createdAt: DateTime!
}

# ---------------------------------------------------------
# INPUTS (For Mutations)
# ---------------------------------------------------------
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

input EdgeInput {
  id: ID!
  sourceNodeId: ID!
  targetNodeId: ID!
  sourceHandle: String!
  targetHandle: String
}

# ---------------------------------------------------------
# ROOT QUERY (Data Fetching)
# ---------------------------------------------------------
type Query {
  # Auth
  me: User!

  # Dashboard
  recentWorkflows(limit: Int = 5): [Workflow!]!
  workflowMetrics(timeframe: String!): [MetricData!]!
  
  # Blueprints
  blueprints(category: String, difficulty: String, limit: Int = 10, offset: Int = 0): BlueprintConnection!

  # Folders & Workflows
  # If parentId is null, it fetches root folders
  folders(parentId: ID): [Folder!]!
  # If folderId is null, it fetches workflows in the root directory
  workflows(folderId: ID): [Workflow!]!
  # Fetches complete workflow data including versions, nodes, and edges
  workflow(id: ID!): Workflow

  # Logs & Executions
  workflowRuns(workflowId: ID!, isTestRun: Boolean, limit: Int = 20): [WorkflowRun!]!
  runSteps(workflowRunId: ID!): [WorkflowRunStep!]!

  # Integrations
  integrations: [IntegrationAccount!]!
}

# ---------------------------------------------------------
# ROOT MUTATION (Data Modification)
# ---------------------------------------------------------
type Mutation {
  # Folders
  createFolder(name: String!, parentId: ID): Folder!
  updateFolder(id: ID!, name: String, parentId: ID): Folder!
  deleteFolder(id: ID!): Boolean!

  # Workflows (Basic Meta)
  createWorkflow(name: String!, folderId: ID): Workflow!
  updateWorkflow(id: ID!, name: String, folderId: ID): Workflow!
  deleteWorkflow(id: ID!): Boolean!

  # Builder / Canvas (Bulk save)
  # Frontend sends the entire canvas state to be saved as a draft
  saveWorkflowDraft(workflowId: ID!, nodes: [NodeInput!]!, edges: [EdgeInput!]!): WorkflowVersion!
  
  # Locks the current draft and sets it as the active version for real executions
  publishWorkflow(workflowId: ID!): WorkflowVersion!

  # Blueprints
  # Clones a blueprint into the user's workspace
  useBlueprint(blueprintId: ID!, targetFolderId: ID): Workflow!

  # Execution
  # Triggers a manual test run from the builder UI
  testRunWorkflow(workflowId: ID!, inputPayload: JSON): WorkflowRun!

  # Integrations
  connectIntegration(provider: IntegrationProvider!, authCode: String!): IntegrationAccount!
  revokeIntegration(id: ID!): Boolean!
}
