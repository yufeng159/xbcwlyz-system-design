# 数据库设计

## 1. 数据库选型

- **主数据库**：MySQL 8.0
- **缓存数据库**：Redis
- **日志和搜索**：Elasticsearch

## 2. 设计原则

- **范式化与反范式化**：在满足业务需求的前提下，尽量遵循第三范式，但在需要提高查询性能时，适当采用反范式化设计。
- **数据一致性**：通过事务、外键等机制保证数据的一致性。
- **可扩展性**：采用分库分表策略，支持未来数据量的增长。
- **安全性**：对敏感数据进行加密存储。

## 3. 概念模型设计 (ER图)

```mermaid
erDiagram
    USER ||--o{ PROJECT : "创建"
    USER ||--o{ CARD : "拥有"
    USER ||--o{ BALANCE : "拥有"
    USER ||--o{ LOGIN_LOG : "有"
    AGENT ||--|{ USER : "管理"
    PROJECT ||--o{ CARD : "属于"
    CARD ||--o{ VERIFICATION_LOG : "有"
    BALANCE ||--o{ TRANSACTION_LOG : "有"
    SYSTEM_CONFIG ||--|{ BANNER : "管理"}

    USER {
        bigint id PK
        varchar username
        varchar password
        varchar email
        varchar phone
        int role_id
        int status
        datetime created_at
        datetime updated_at
    }

    AGENT {
        bigint id PK
        bigint user_id FK
        bigint parent_id
        varchar level
        decimal commission_rate
    }

    PROJECT {
        bigint id PK
        bigint user_id FK
        varchar name
        varchar description
        varchar app_key
        varchar app_secret
        int status
        datetime created_at
        datetime updated_at
    }

    CARD {
        bigint id PK
        bigint project_id FK
        bigint user_id FK
        varchar card_no
        varchar password
        int status
        datetime created_at
        datetime activated_at
        datetime expired_at
    }

    VERIFICATION_LOG {
        bigint id PK
        bigint card_id FK
        varchar ip_address
        varchar user_agent
        int result
        datetime created_at
    }

    BALANCE {
        bigint id PK
        bigint user_id FK
        decimal amount
        datetime updated_at
    }

    TRANSACTION_LOG {
        bigint id PK
        bigint balance_id FK
        decimal amount
        int type
        varchar description
        datetime created_at
    }

    LOGIN_LOG {
        bigint id PK
        bigint user_id FK
        varchar ip_address
        varchar user_agent
        datetime login_at
    }

    SYSTEM_CONFIG {
        int id PK
        varchar key
        varchar value
        varchar description
    }

    BANNER {
        int id PK
        varchar title
        varchar image_url
        varchar link_url
        int sort_order
        int status
    }
```

## 4. 物理模型设计

### 4.1 用户库 (user_db)

- **t_user** (用户信息表)
- **t_agent** (代理商信息表)
- **t_login_log** (登录日志表)

### 4.2 项目库 (project_db)

- **t_project** (项目信息表)

### 4.3 卡片库 (card_db)

- **t_card** (卡片信息表)
- **t_verification_log** (卡片核销日志表)

### 4.4 资金库 (balance_db)

- **t_balance** (用户余额表)
- **t_transaction_log** (交易流水表)

### 4.5 系统库 (system_db)

- **t_system_config** (系统配置表)
- **t_banner** (Banner配置表)

## 5. 分库分表策略

- **垂直拆分**：根据业务功能将数据库拆分为用户库、项目库、卡片库、资金库和系统库。
- **水平拆分**：
  - `t_card` 表和 `t_verification_log` 表：根据 `card_no` 的哈希值进行分片。
  - `t_transaction_log` 表：根据 `user_id` 进行分片。

## 6. 索引设计

- **主键索引**：所有表都包含一个自增的`id`作为主键。
- **唯一索引**：`t_user`的`username`、`email`、`phone`；`t_project`的`app_key`；`t_card`的`card_no`。
- **普通索引**：为经常用于查询条件的字段创建索引，如`t_card`的`user_id`、`status`；`t_transaction_log`的`user_id`、`type`。

## 7. 数据字典

| 字段名 | 数据类型 | 描述 |
|---|---|---|
| `status` | `int` | 状态 (例如: 0-禁用, 1-启用, 2-待审核) |
| `role_id` | `int` | 角色ID (例如: 1-超级管理员, 2-代理商, 3-普通用户) |
| `type` | `int` | 类型 (例如: 1-充值, 2-消费, 3-提现) |

