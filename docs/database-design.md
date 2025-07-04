# 数据库设计

## 1. 数据库选型

本系统选择PostgreSQL作为主要数据库，原因如下：

- 强大的事务支持和ACID特性
- 丰富的数据类型和索引类型
- 优秀的并发性能和可扩展性
- 强大的JSON支持，适合存储设备信息等半结构化数据
- 完善的社区支持和文档

## 2. 数据库表设计

### 2.1 用户相关表

#### 2.1.1 用户表 (users)

存储系统中所有用户的基本信息，包括超级管理员、代理管理员和普通代理。

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) NOT NULL UNIQUE,
  password VARCHAR(100) NOT NULL,
  email VARCHAR(100) NOT NULL UNIQUE,
  role VARCHAR(20) NOT NULL, -- 'super_admin', 'admin', 'agent'
  parent_id INT REFERENCES users(id),
  balance DECIMAL(10,2) DEFAULT 0,
  frozen_balance DECIMAL(10,2) DEFAULT 0,
  status VARCHAR(10) DEFAULT 'active', -- 'active', 'suspended', 'deleted'
  last_login_time TIMESTAMP,
  last_login_ip VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_users_parent_id ON users(parent_id);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_status ON users(status);
```

字段说明：
- `id`: 用户唯一标识
- `username`: 用户名，用于登录
- `password`: 加密存储的密码
- `email`: 邮箱，用于找回密码等操作
- `role`: 用户角色，区分超级管理员、代理管理员和普通代理
- `parent_id`: 上级用户ID，构建代理层级关系
- `balance`: 用户当前可用额度
- `frozen_balance`: 冻结额度，用于转账等操作
- `status`: 用户状态
- `last_login_time`: 最后登录时间
- `last_login_ip`: 最后登录IP
- `created_at`: 创建时间
- `updated_at`: 更新时间

#### 2.1.2 用户配置表 (user_configs)

存储用户个性化配置信息，采用键值对形式。

```sql
CREATE TABLE user_configs (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id),
  config_key VARCHAR(50) NOT NULL,
  config_value TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, config_key)
);

-- 索引
CREATE INDEX idx_user_configs_user_id ON user_configs(user_id);
```

字段说明：
- `id`: 配置项唯一标识
- `user_id`: 关联的用户ID
- `config_key`: 配置项键名
- `config_value`: 配置项值
- `created_at`: 创建时间
- `updated_at`: 更新时间

#### 2.1.3 用户登录日志表 (user_login_logs)

记录用户登录历史，用于安全审计和异常登录检测。

```sql
CREATE TABLE user_login_logs (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id),
  login_ip VARCHAR(50) NOT NULL,
  login_location VARCHAR(100),
  device_info TEXT,
  login_status VARCHAR(10) NOT NULL, -- 'success', 'failed'
  fail_reason VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_user_login_logs_user_id ON user_login_logs(user_id);
CREATE INDEX idx_user_login_logs_created_at ON user_login_logs(created_at);
```

字段说明：
- `id`: 日志唯一标识
- `user_id`: 关联的用户ID
- `login_ip`: 登录IP地址
- `login_location`: 登录地理位置
- `device_info`: 登录设备信息
- `login_status`: 登录状态
- `fail_reason`: 失败原因
- `created_at`: 创建时间

### 2.2 项目相关表

#### 2.2.1 项目表 (projects)

存储系统中所有软件项目的基本信息。

```sql
CREATE TABLE projects (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  version VARCHAR(20),
  api_key VARCHAR(64) NOT NULL UNIQUE,
  secret_key VARCHAR(64) NOT NULL,
  callback_url VARCHAR(255),
  status VARCHAR(10) DEFAULT 'active', -- 'active', 'suspended', 'deleted'
  created_by INT NOT NULL REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_projects_created_by ON projects(created_by);
CREATE INDEX idx_projects_status ON projects(status);
```

字段说明：
- `id`: 项目唯一标识
- `name`: 项目名称
- `description`: 项目描述
- `version`: 项目版本
- `api_key`: API密钥，用于接口认证
- `secret_key`: 密钥，用于数据加密
- `callback_url`: 回调URL，用于通知项目方
- `status`: 项目状态
- `created_by`: 创建者ID
- `created_at`: 创建时间
- `updated_at`: 更新时间

#### 2.2.2 项目授权表 (project_authorizations)

记录项目对代理的授权关系。

```sql
CREATE TABLE project_authorizations (
  id SERIAL PRIMARY KEY,
  project_id INT NOT NULL REFERENCES projects(id),
  user_id INT NOT NULL REFERENCES users(id),
  authorized_by INT NOT NULL REFERENCES users(id),
  expiration_date TIMESTAMP,
  status VARCHAR(10) DEFAULT 'active', -- 'active', 'expired', 'revoked'
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(project_id, user_id)
);

-- 索引
CREATE INDEX idx_project_authorizations_project_id ON project_authorizations(project_id);
CREATE INDEX idx_project_authorizations_user_id ON project_authorizations(user_id);
CREATE INDEX idx_project_authorizations_status ON project_authorizations(status);
```

字段说明：
- `id`: 授权记录唯一标识
- `project_id`: 项目ID
- `user_id`: 被授权用户ID
- `authorized_by`: 授权者ID
- `expiration_date`: 授权过期时间
- `status`: 授权状态
- `created_at`: 创建时间
- `updated_at`: 更新时间

#### 2.2.3 项目申请表 (project_applications)

记录代理申请项目授权的信息。

```sql
CREATE TABLE project_applications (
  id SERIAL PRIMARY KEY,
  project_id INT NOT NULL REFERENCES projects(id),
  user_id INT NOT NULL REFERENCES users(id),
  reason TEXT,
  status VARCHAR(10) DEFAULT 'pending', -- 'pending', 'approved', 'rejected'
  processed_by INT REFERENCES users(id),
  processed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_project_applications_project_id ON project_applications(project_id);
CREATE INDEX idx_project_applications_user_id ON project_applications(user_id);
CREATE INDEX idx_project_applications_status ON project_applications(status);
```

字段说明：
- `id`: 申请记录唯一标识
- `project_id`: 项目ID
- `user_id`: 申请者ID
- `reason`: 申请理由
- `status`: 申请状态
- `processed_by`: 处理者ID
- `processed_at`: 处理时间
- `created_at`: 创建时间

### 2.3 卡密相关表

#### 2.3.1 卡密表 (cards)

存储系统中所有卡密的基本信息。

```sql
CREATE TABLE cards (
  id SERIAL PRIMARY KEY,
  card_no VARCHAR(32) NOT NULL UNIQUE,
  project_id INT NOT NULL REFERENCES projects(id),
  batch_id VARCHAR(32) NOT NULL,
  created_by INT NOT NULL REFERENCES users(id),
  status VARCHAR(10) DEFAULT 'unused', -- 'unused', 'used', 'expired', 'disabled'
  duration INT NOT NULL, -- 有效期天数
  device_id VARCHAR(64), -- 绑定的设备ID
  device_info TEXT, -- 设备信息JSON
  remark TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  activated_at TIMESTAMP,
  expire_at TIMESTAMP
);

-- 索引
CREATE INDEX idx_cards_project_id ON cards(project_id);
CREATE INDEX idx_cards_created_by ON cards(created_by);
CREATE INDEX idx_cards_batch_id ON cards(batch_id);
CREATE INDEX idx_cards_status ON cards(status);
CREATE INDEX idx_cards_device_id ON cards(device_id);
```

字段说明：
- `id`: 卡密唯一标识
- `card_no`: 卡密号码
- `project_id`: 关联的项目ID
- `batch_id`: 批次ID，用于批量管理
- `created_by`: 创建者ID
- `status`: 卡密状态
- `duration`: 有效期天数
- `device_id`: 绑定的设备ID
- `device_info`: 设备信息，JSON格式
- `remark`: 备注
- `created_at`: 创建时间
- `activated_at`: 激活时间
- `expire_at`: 过期时间

#### 2.3.2 卡密批次表 (card_batches)

记录卡密批量生成的批次信息。

```sql
CREATE TABLE card_batches (
  id SERIAL PRIMARY KEY,
  batch_id VARCHAR(32) NOT NULL UNIQUE,
  project_id INT NOT NULL REFERENCES projects(id),
  created_by INT NOT NULL REFERENCES users(id),
  card_count INT NOT NULL,
  duration INT NOT NULL, -- 有效期天数
  cost_balance DECIMAL(10,2) NOT NULL, -- 消耗额度
  remark TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_card_batches_project_id ON card_batches(project_id);
CREATE INDEX idx_card_batches_created_by ON card_batches(created_by);
```

字段说明：
- `id`: 批次记录唯一标识
- `batch_id`: 批次ID
- `project_id`: 关联的项目ID
- `created_by`: 创建者ID
- `card_count`: 卡密数量
- `duration`: 有效期天数
- `cost_balance`: 消耗额度
- `remark`: 备注
- `created_at`: 创建时间

#### 2.3.3 卡密激活记录表 (card_activations)

记录卡密的激活历史，用于追踪卡密使用情况。

```sql
CREATE TABLE card_activations (
  id SERIAL PRIMARY KEY,
  card_id INT NOT NULL REFERENCES cards(id),
  device_id VARCHAR(64) NOT NULL,
  device_info TEXT,
  ip_address VARCHAR(50),
  location VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_card_activations_card_id ON card_activations(card_id);
CREATE INDEX idx_card_activations_device_id ON card_activations(device_id);
CREATE INDEX idx_card_activations_created_at ON card_activations(created_at);
```

字段说明：
- `id`: 激活记录唯一标识
- `card_id`: 关联的卡密ID
- `device_id`: 设备ID
- `device_info`: 设备信息，JSON格式
- `ip_address`: IP地址
- `location`: 地理位置
- `created_at`: 创建时间

### 2.4 额度相关表

#### 2.4.1 额度变动记录表 (balance_records)

记录用户额度的变动历史。

```sql
CREATE TABLE balance_records (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id),
  type VARCHAR(20) NOT NULL, -- 'recharge', 'transfer_in', 'transfer_out', 'consume', 'refund'
  amount DECIMAL(10,2) NOT NULL,
  balance DECIMAL(10,2) NOT NULL, -- 变动后余额
  related_user_id INT REFERENCES users(id), -- 关联用户（转账对象）
  related_id INT, -- 关联ID（卡密批次ID等）
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_balance_records_user_id ON balance_records(user_id);
CREATE INDEX idx_balance_records_type ON balance_records(type);
CREATE INDEX idx_balance_records_related_user_id ON balance_records(related_user_id);
CREATE INDEX idx_balance_records_created_at ON balance_records(created_at);
```

字段说明：
- `id`: 记录唯一标识
- `user_id`: 用户ID
- `type`: 变动类型
- `amount`: 变动金额
- `balance`: 变动后余额
- `related_user_id`: 关联用户ID
- `related_id`: 关联ID
- `description`: 描述
- `created_at`: 创建时间

#### 2.4.2 提现申请表 (withdrawal_requests)

记录用户提现申请信息。

```sql
CREATE TABLE withdrawal_requests (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id),
  amount DECIMAL(10,2) NOT NULL,
  account_info TEXT NOT NULL, -- JSON格式账户信息
  status VARCHAR(10) DEFAULT 'pending', -- 'pending', 'approved', 'rejected', 'completed'
  processed_by INT REFERENCES users(id),
  processed_at TIMESTAMP,
  remark TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_withdrawal_requests_user_id ON withdrawal_requests(user_id);
CREATE INDEX idx_withdrawal_requests_status ON withdrawal_requests(status);
CREATE INDEX idx_withdrawal_requests_created_at ON withdrawal_requests(created_at);
```

字段说明：
- `id`: 申请唯一标识
- `user_id`: 用户ID
- `amount`: 提现金额
- `account_info`: 账户信息，JSON格式
- `status`: 申请状态
- `processed_by`: 处理者ID
- `processed_at`: 处理时间
- `remark`: 备注
- `created_at`: 创建时间
- `updated_at`: 更新时间

### 2.5 系统相关表

#### 2.5.1 系统配置表 (system_configs)

存储系统全局配置信息。

```sql
CREATE TABLE system_configs (
  id SERIAL PRIMARY KEY,
  config_key VARCHAR(50) NOT NULL UNIQUE,
  config_value TEXT,
  config_type VARCHAR(20) NOT NULL, -- 'system', 'payment', 'notification'
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_system_configs_config_type ON system_configs(config_type);
```

字段说明：
- `id`: 配置项唯一标识
- `config_key`: 配置项键名
- `config_value`: 配置项值
- `config_type`: 配置类型
- `description`: 描述
- `created_at`: 创建时间
- `updated_at`: 更新时间

#### 2.5.2 轮播图表 (banners)

存储首页轮播图信息。

```sql
CREATE TABLE banners (
  id SERIAL PRIMARY KEY,
  image_url VARCHAR(255) NOT NULL,
  link_url VARCHAR(255),
  order_num INT DEFAULT 0,
  status VARCHAR(10) DEFAULT 'active', -- 'active', 'inactive'
  created_by INT NOT NULL REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_banners_status ON banners(status);
CREATE INDEX idx_banners_order_num ON banners(order_num);
```

字段说明：
- `id`: 轮播图唯一标识
- `image_url`: 图片URL
- `link_url`: 链接URL
- `order_num`: 排序号
- `status`: 状态
- `created_by`: 创建者ID
- `created_at`: 创建时间
- `updated_at`: 更新时间

#### 2.5.3 系统日志表 (system_logs)

记录系统操作日志，用于审计和问题排查。

```sql
CREATE TABLE system_logs (
  id SERIAL PRIMARY KEY,
  level VARCHAR(10) NOT NULL, -- 'info', 'warning', 'error'
  module VARCHAR(50) NOT NULL,
  content TEXT NOT NULL,
  ip_address VARCHAR(50),
  user_id INT REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_system_logs_level ON system_logs(level);
CREATE INDEX idx_system_logs_module ON system_logs(module);
CREATE INDEX idx_system_logs_user_id ON system_logs(user_id);
CREATE INDEX idx_system_logs_created_at ON system_logs(created_at);
```

字段说明：
- `id`: 日志唯一标识
- `level`: 日志级别
- `module`: 模块名称
- `content`: 日志内容
- `ip_address`: IP地址
- `user_id`: 用户ID
- `created_at`: 创建时间

#### 2.5.4 通知消息表 (notifications)

存储系统通知和用户消息。

```sql
CREATE TABLE notifications (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id),
  title VARCHAR(100) NOT NULL,
  content TEXT NOT NULL,
  type VARCHAR(20) NOT NULL, -- 'system', 'personal'
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 索引
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_type ON notifications(type);
CREATE INDEX idx_notifications_is_read ON notifications(is_read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at);
```

字段说明：
- `id`: 通知唯一标识
- `user_id`: 接收用户ID，为NULL表示全局通知
- `title`: 通知标题
- `content`: 通知内容
- `type`: 通知类型
- `is_read`: 是否已读
- `created_at`: 创建时间

## 3. 数据库关系图

```
+---------------+       +----------------------+       +-------------------+
|    users      |       | project_authorizations|       |     projects     |
+---------------+       +----------------------+       +-------------------+
| id            |<----->| user_id              |       | id               |
| username      |       | project_id           |<----->| name             |
| password      |       | authorized_by        |       | description      |
| email         |       | expiration_date      |       | version          |
| role          |       | status               |       | api_key          |
| parent_id     |       | created_at           |       | secret_key       |
| balance       |       | updated_at           |       | callback_url     |
| frozen_balance|       +----------------------+       | status           |
| status        |                                     | created_by       |
| last_login_time|                                    | created_at       |
| last_login_ip |                                     | updated_at       |
| created_at    |                                     +-------------------+
| updated_at    |                                               ^
+---------------+                                               |
      ^                                                         |
      |                                                         |
      v                                                         |
+---------------+       +-------------------+       +-------------------+
| user_configs  |       |    cards          |       | card_batches      |
+---------------+       +-------------------+       +-------------------+
| id            |       | id                |       | id                |
| user_id       |       | card_no           |       | batch_id          |
| config_key    |       | project_id        |------>| project_id        |
| config_value  |       | batch_id          |<------| created_by        |
| created_at    |       | created_by        |       | card_count        |
| updated_at    |       | status            |       | duration          |
+---------------+       | duration          |       | cost_balance      |
                        | device_id         |       | remark            |
                        | device_info       |       | created_at        |
                        | remark            |       +-------------------+
                        | created_at        |
                        | activated_at      |
                        | expire_at         |
                        +-------------------+
                                 ^
                                 |
                                 |
                        +-------------------+
                        | card_activations  |
                        +-------------------+
                        | id                |
                        | card_id           |
                        | device_id         |
                        | device_info       |
                        | ip_address        |
                        | location          |
                        | created_at        |
                        +-------------------+
```

## 4. 数据库优化策略

### 4.1 索引优化

- 为常用查询条件创建适当的索引
- 对于复合索引，遵循最左前缀原则
- 避免过度索引，平衡查询性能和写入性能
- 定期分析索引使用情况，优化低效索引

### 4.2 分区策略

对于大表采用分区策略：

- 卡密表按项目ID进行分区
- 日志表按时间范围分区
- 额度记录表按用户ID和时间范围分区

### 4.3 读写分离

- 主库负责写操作和核心业务读操作
- 从库负责统计查询和报表生成等非核心读操作
- 使用连接池管理数据库连接

### 4.4 数据归档

- 定期将过期卡密、历史日志等数据归档到历史表
- 设置数据保留策略，避免表过大影响性能
- 使用分区表简化归档操作

## 5. 数据安全策略

### 5.1 数据加密

- 密码使用Argon2或bcrypt算法加密存储
- 敏感信息（如API密钥）使用AES加密存储
- 传输数据使用TLS/SSL加密

### 5.2 数据备份

- 每日全量备份
- 实时WAL日志备份
- 定期测试恢复流程
- 异地备份策略

### 5.3 访问控制

- 最小权限原则配置数据库用户
- 应用使用只读账号进行查询操作
- 定期审计数据库访问日志
- 使用防火墙限制数据库访问来源

## 6. 数据迁移和版本控制

### 6.1 迁移策略

- 使用Flyway或Liquibase管理数据库版本
- 每次变更创建独立的迁移脚本
- 迁移脚本包含向前和向后兼容的操作
- 在测试环境验证迁移脚本

### 6.2 版本控制

- 数据库脚本纳入代码版本控制
- 使用语义化版本号管理数据库版本
- 记录每次变更的目的和影响
- 保持开发、测试和生产环境的数据库结构一致性