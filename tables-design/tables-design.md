# Database Tables Design

This document describes the database schema for the Automation Workflow Builder platform.

---

## Table of Contents

- [Users](#users)
- [Blueprints](#blueprints)
- [Workflows](#workflows)
- [Workflow Versions](#workflow_versions)
- [Workflow Nodes](#workflow_nodes)
- [Workflow Edges](#workflow_edges)
- [Integration Accounts](#integration_accounts)
- [Workflow Runs](#workflow_runs)
- [Workflow Run Steps](#workflow_run_steps)

---

## Users

Stores user account information.

| Column          | Type     | Constraints      | Description                    |
| --------------- | -------- | ---------------- | ------------------------------ |
| `id`            | `uuid`   | PRIMARY KEY      | Unique identifier              |
| `email`         | `string` | UNIQUE, NOT NULL | User email address             |
| `password_hash` | `string` | NOT NULL         | Hashed password                |
| `username`      | `string` | NOT NULL         | Display name                   |
| `avatar_key`    | `string` | NULLABLE         | Reference to avatar image      |
| `created_at`    | `Date`   | NOT NULL         | Record creation timestamp      |
| `updated_at`    | `Date`   | NOT NULL         | Record last update timestamp   |

---

## Blueprints

Pre-built workflow templates that users can use as starting points.

| Column                  | Type     | Constraints           | Description                                |
| ----------------------- | -------- | --------------------- | ------------------------------------------ |
| `id`                    | `uuid`   | PRIMARY KEY           | Unique identifier                          |
| `template_workflow_id`  | `uuid`   | UNIQUE, FK(workflows) | Reference to the template workflow         |
| `name`                  | `string` | NOT NULL              | Blueprint name                             |
| `description`           | `string` | NOT NULL              | Blueprint description                      |
| `usage_count`           | `number` | NOT NULL              | Number of times used                       |
| `hours_saved_per_week`  | `number` | NOT NULL              | Estimated hours saved                      |
| `preview_image_key`     | `string` | NULLABLE              | Reference to preview image                 |
| `difficulty`            | `enum`   | NOT NULL              | `BEGINNER` \| `INTERMEDIATE` \| `EXPERT`   |
| `category`              | `enum`   | NOT NULL              | See categories below                       |
| `status`                | `enum`   | NOT NULL              | `draft` \| `published` \| `archived` \| `inactive` |
| `created_at`            | `Date`   | NOT NULL              | Record creation timestamp                  |
| `updated_at`            | `Date`   | NOT NULL              | Record last update timestamp               |

**Categories:** `CRM` | `GPT-4` | `SQL` | `SYNC` | `SLACK` | `NLP` | `FINTECH` | `SECURITY` | `L10N` | `API` | `OPS` | `HR` | `GIT` | `REPORT`

---

## Workflows

Main workflow definitions owned by users.

| Column               | Type     | Constraints                  | Description                        |
| -------------------- | -------- | ---------------------------- | ---------------------------------- |
| `id`                 | `uuid`   | PRIMARY KEY                  | Unique identifier                  |
| `name`               | `string` | NOT NULL                     | Workflow name                      |
| `description`        | `string` | NULLABLE                     | Workflow description               |
| `created_by_user_id` | `uuid`   | NOT NULL, FK(users)          | Owner user reference               |
| `current_version_id` | `uuid`   | NULLABLE, FK(workflow_versions) | Currently active version        |
| `created_at`         | `Date`   | NOT NULL                     | Record creation timestamp          |
| `updated_at`         | `Date`   | NOT NULL                     | Record last update timestamp       |

---

## Workflow Versions

Version history for workflows, supporting draft and published states.

| Column               | Type     | Constraints                      | Description                              |
| -------------------- | -------- | -------------------------------- | ---------------------------------------- |
| `id`                 | `uuid`   | PRIMARY KEY                      | Unique identifier                        |
| `workflow_id`        | `uuid`   | NOT NULL, FK(workflows)          | Parent workflow reference                |
| `version_number`     | `number` | NOT NULL                         | Version sequence number                  |
| `status`             | `enum`   | NOT NULL                         | `draft` \| `published`                   |
| `created_by_user_id` | `uuid`   | NOT NULL, FK(users)              | User who created this version            |
| `created_at`         | `Date`   | NOT NULL                         | Record creation timestamp                |
| `updated_at`         | `Date`   | NOT NULL                         | Record last update timestamp             |

**Constraints:**

- `UNIQUE(workflow_id, version_number)` - Each version number is unique per workflow
- Only **one published version** is allowed per workflow

---

## Workflow Nodes

Individual nodes within a workflow version (triggers, actions, utilities).

| Column                   | Type     | Constraints                        | Description                                 |
| ------------------------ | -------- | ---------------------------------- | ------------------------------------------- |
| `id`                     | `uuid`   | PRIMARY KEY                        | Unique identifier                           |
| `workflow_version_id`    | `uuid`   | NOT NULL, FK(workflow_versions)    | Parent workflow version                     |
| `integration_account_id` | `uuid`   | NULLABLE, FK(integration_accounts) | Linked integration account                  |
| `node_type`              | `enum`   | NOT NULL                           | `trigger` \| `action` \| `utility`          |
| `provider_app`           | `enum`   | NOT NULL                           | See providers below                         |
| `action_key`             | `string` | NOT NULL                           | Specific action identifier                  |
| `label`                  | `string` | NOT NULL                           | Display label for the node                  |
| `position_x`             | `number` | NULLABLE                           | X coordinate (null for auto-layout)         |
| `position_y`             | `number` | NULLABLE                           | Y coordinate (null for auto-layout)         |
| `config_json`            | `JSONB`  | NOT NULL                           | Node configuration (API keys, templates, etc.) |
| `created_at`             | `Date`   | NOT NULL                           | Record creation timestamp                   |
| `updated_at`             | `Date`   | NOT NULL                           | Record last update timestamp                |

**Provider Apps:** `google_form` | `slack` | `gmail` | `google_sheet` | `facebook` | `system` | `utility`

---

## Workflow Edges

Connections between nodes defining data flow and execution paths.

| Column               | Type     | Constraints                     | Description                                        |
| -------------------- | -------- | ------------------------------- | -------------------------------------------------- |
| `id`                 | `uuid`   | PRIMARY KEY                     | Unique identifier                                  |
| `workflow_version_id`| `uuid`   | NOT NULL, FK(workflow_versions) | Parent workflow version                            |
| `source_node_id`     | `uuid`   | NOT NULL, FK(workflow_nodes)    | Source node reference                              |
| `target_node_id`     | `uuid`   | NOT NULL, FK(workflow_nodes)    | Target node reference                              |
| `source_handle`      | `enum`   | NOT NULL                        | `default` \| `true` \| `false` \| `success` \| `error` |
| `target_handle`      | `string` | NULLABLE                        | Input key for multi-input target nodes             |
| `created_at`         | `Date`   | NOT NULL                        | Record creation timestamp                          |
| `updated_at`         | `Date`   | NOT NULL                        | Record last update timestamp                       |

**Constraints:**

- `UNIQUE(workflow_version_id, source_node_id, source_handle, target_node_id, target_handle)`

---

## Integration Accounts

OAuth connections to external services.

| Column           | Type     | Constraints         | Description                                |
| ---------------- | -------- | ------------------- | ------------------------------------------ |
| `id`             | `uuid`   | PRIMARY KEY         | Unique identifier                          |
| `owner_user_id`  | `uuid`   | NOT NULL, FK(users) | Owner user reference                       |
| `provider`       | `enum`   | NOT NULL            | `google` \| `slack` \| `gmail` \| `google_sheet` \| `facebook` |
| `account_email`  | `string` | NOT NULL            | Email associated with the account          |
| `credential_ref` | `string` | NOT NULL            | Reference to encrypted credentials         |
| `status`         | `enum`   | NOT NULL            | `connected` \| `expired` \| `revoked`      |
| `created_at`     | `Date`   | NOT NULL            | Record creation timestamp                  |
| `updated_at`     | `Date`   | NOT NULL            | Record last update timestamp               |

---

## Workflow Runs

Execution records for workflow instances.

| Column                 | Type     | Constraints                     | Description                              |
| ---------------------- | -------- | ------------------------------- | ---------------------------------------- |
| `id`                   | `uuid`   | PRIMARY KEY                     | Unique identifier                        |
| `workflow_id`          | `uuid`   | NOT NULL, FK(workflows)         | Parent workflow reference                |
| `workflow_version_id`  | `uuid`   | NOT NULL, FK(workflow_versions) | Executed version reference               |
| `initiated_by_user_id` | `uuid`   | NULLABLE, FK(users)             | User who triggered (null for system)     |
| `trigger_type`         | `enum`   | NOT NULL                        | `manual` \| `webhook` \| `schedule`      |
| `input_payload`        | `JSONB`  | NULLABLE                        | Input data for the run                   |
| `status`               | `enum`   | NOT NULL                        | `pending` \| `running` \| `success` \| `failed` \| `cancelled` |
| `started_at`           | `Date`   | NOT NULL                        | Execution start timestamp                |
| `finished_at`          | `Date`   | NULLABLE                        | Execution end timestamp                  |
| `error_message`        | `string` | NULLABLE                        | Error details if failed                  |
| `created_at`           | `Date`   | NOT NULL                        | Record creation timestamp                |
| `updated_at`           | `Date`   | NOT NULL                        | Record last update timestamp             |

---

## Workflow Run Steps

Individual node execution records within a workflow run.

| Column            | Type     | Constraints                  | Description                              |
| ----------------- | -------- | ---------------------------- | ---------------------------------------- |
| `id`              | `uuid`   | PRIMARY KEY                  | Unique identifier                        |
| `workflow_run_id` | `uuid`   | NOT NULL, FK(workflow_runs)  | Parent run reference                     |
| `workflow_node_id`| `uuid`   | NOT NULL, FK(workflow_nodes) | Executed node reference                  |
| `execution_order` | `number` | NOT NULL                     | Order of execution in the run            |
| `status`          | `enum`   | NOT NULL                     | `pending` \| `running` \| `success` \| `failed` \| `skipped` |
| `input_data`      | `JSONB`  | NULLABLE                     | Input data for this step                 |
| `output_data`     | `JSONB`  | NULLABLE                     | Output data from this step               |
| `started_at`      | `Date`   | NOT NULL                     | Step start timestamp                     |
| `finished_at`     | `Date`   | NULLABLE                     | Step end timestamp                       |
| `duration_ms`     | `number` | NULLABLE                     | Execution duration in milliseconds       |
| `error_message`   | `string` | NULLABLE                     | Error details if failed                  |
| `created_at`      | `Date`   | NOT NULL                     | Record creation timestamp                |
| `updated_at`      | `Date`   | NOT NULL                     | Record last update timestamp             |

---
