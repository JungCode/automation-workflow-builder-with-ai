# Database Entity Relationship Diagram

This diagram visualizes the relationships between all tables in the Automation Workflow Builder database.

## ER Diagram

```mermaid
erDiagram
    users ||--o{ workflows : "creates"
    users ||--o{ workflow_versions : "creates"
    users ||--o{ integration_accounts : "owns"
    users ||--o{ workflow_runs : "initiates"
    users ||--o{ folders : "owns"

    folders ||--o{ workflows : "contains"
    folders }o--o{ folders : "parent_of"

    workflows ||--o{ workflow_versions : "has"
    workflows ||--o| blueprints : "template_for"
    workflows ||--o{ workflow_runs : "executes"
    workflows |o--|| workflow_versions : "current_version"

    workflow_versions ||--o{ workflow_nodes : "contains"
    workflow_versions ||--o{ workflow_edges : "contains"
    workflow_versions ||--o{ workflow_runs : "executed_as"

    workflow_nodes ||--o{ workflow_edges : "source"
    workflow_nodes ||--o{ workflow_edges : "target"
    workflow_nodes |o--|| integration_accounts : "uses"
    workflow_nodes ||--o{ workflow_run_steps : "executed_in"

    workflow_runs ||--o{ workflow_run_steps : "contains"

    users {
        uuid id PK
        string email UK
        string password_hash
        string username
        string avatar_key
        date created_at
        date updated_at
    }

    folders {
        uuid id PK
        string name
        uuid parent_id FK
        uuid owner_user_id FK
        date created_at
        date updated_at
    }

    blueprints {
        uuid id PK
        uuid template_workflow_id FK,UK
        string name
        string description
        number usage_count
        number hours_saved_per_week
        string preview_image_key
        enum difficulty
        enum category
        enum status
        date created_at
        date updated_at
    }

    workflows {
        uuid id PK
        string name
        string description
        uuid created_by_user_id FK
        uuid current_version_id FK
        uuid folder_id FK
        date created_at
        date updated_at
    }

    workflow_versions {
        uuid id PK
        uuid workflow_id FK
        number version_number
        enum status
        uuid created_by_user_id FK
        date created_at
        date updated_at
    }

    workflow_nodes {
        uuid id PK
        uuid workflow_version_id FK
        uuid integration_account_id FK
        enum node_type
        enum provider_app
        string action_key
        string label
        number position_x
        number position_y
        jsonb config_json
        date created_at
        date updated_at
    }

    workflow_edges {
        uuid id PK
        uuid workflow_version_id FK
        uuid source_node_id FK
        uuid target_node_id FK
        enum source_handle
        string target_handle
        date created_at
        date updated_at
    }

    integration_accounts {
        uuid id PK
        uuid owner_user_id FK
        enum provider
        string account_email
        string credential_ref
        enum status
        date created_at
        date updated_at
    }

    workflow_runs {
        uuid id PK
        uuid workflow_id FK
        uuid workflow_version_id FK
        uuid initiated_by_user_id FK
        enum trigger_type
        jsonb input_payload
        enum status
        boolean is_test_run
        date started_at
        date finished_at
        string error_message
        date created_at
        date updated_at
    }

    workflow_run_steps {
        uuid id PK
        uuid workflow_run_id FK
        uuid workflow_node_id FK
        number execution_order
        enum status
        jsonb input_data
        jsonb output_data
        date started_at
        date finished_at
        number duration_ms
        string error_message
        date created_at
        date updated_at
    }
```

## Relationship Summary

| From Table          | To Table             | Relationship | Description                                      |
| ------------------- | -------------------- | ------------ | ------------------------------------------------ |
| `users`             | `workflows`          | 1:N          | A user can create many workflows                 |
| `users`             | `workflow_versions`  | 1:N          | A user can create many versions                  |
| `users`             | `integration_accounts` | 1:N        | A user can have many integration accounts        |
| `users`             | `workflow_runs`      | 1:N          | A user can initiate many workflow runs           |
| `users`             | `folders`            | 1:N          | A user can own many folders                      |
| `folders`           | `workflows`          | 1:N          | A folder contains many workflows                 |
| `folders`           | `folders`            | 1:N          | A folder can have child folders                  |
| `workflows`         | `folders`            | N:1          | A workflow belongs to one folder                 |
| `workflows`         | `workflow_versions`  | 1:N          | A workflow has many versions                     |
| `workflows`         | `blueprints`         | 1:1          | A workflow can be a template for one blueprint   |
| `workflows`         | `workflow_runs`      | 1:N          | A workflow can have many runs                    |
| `workflow_versions` | `workflow_nodes`     | 1:N          | A version contains many nodes                    |
| `workflow_versions` | `workflow_edges`     | 1:N          | A version contains many edges                    |
| `workflow_versions` | `workflow_runs`      | 1:N          | A version can be executed many times             |
| `workflow_nodes`    | `workflow_edges`     | 1:N          | A node can be source/target of many edges        |
| `workflow_nodes`    | `integration_accounts` | N:1        | A node can use one integration account           |
| `workflow_nodes`    | `workflow_run_steps` | 1:N          | A node can be executed in many run steps         |
| `workflow_runs`     | `workflow_run_steps` | 1:N          | A run contains many execution steps              |

## Visual Legend

- **PK** = Primary Key
- **FK** = Foreign Key
- **UK** = Unique Key
- `||--o{` = One-to-Many relationship
- `||--||` = One-to-One relationship
- `|o--||` = Zero-or-One to One relationship
