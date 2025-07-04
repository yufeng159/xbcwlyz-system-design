# API接口设计

## 1. 设计原则

- **RESTful风格**：使用标准的HTTP方法 (GET, POST, PUT, DELETE) 对资源进行操作。
- **版本控制**：在URL中加入版本号，如 `/api/v1/`。
- **统一响应格式**：所有API响应使用统一的JSON格式。
- **安全性**：所有需要认证的接口使用JWT进行身份验证。
- **幂等性**：对于PUT和DELETE等操作，保证其幂等性。

## 2. 统一响应格式

```json
{
  "code": 0, // 0: 成功, 其他: 失败
  "message": "Success",
  "data": {}
}
```

## 3. 统一错误码

| Code | Message | Description |
|---|---|---|
| 0 | Success | 操作成功 |
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未授权或Token失效 |
| 403 | Forbidden | 无权限访问 |
| 404 | Not Found | 资源不存在 |
| 500 | Internal Server Error | 服务器内部错误 |
| 1001 | User Not Found | 用户不存在 |
| 1002 | Invalid Password | 密码错误 |
| 2001 | Card Not Found | 卡密不存在 |
| 2002 | Card Used | 卡密已被使用 |

## 4. 认证授权

- **登录接口**：`POST /api/v1/auth/login`，成功后返回JWT Token。
- **Token传递**：客户端在后续请求的 `Authorization` Header 中携带Token，格式为 `Bearer <token>`。
- **Token刷新**：提供刷新Token的接口 `POST /api/v1/auth/refresh`。

## 5. API接口详情

### 5.1 用户服务 (User Service)

- **用户注册**
  - `POST /api/v1/users/register`
- **用户登录**
  - `POST /api/v1/auth/login`
- **获取当前用户信息**
  - `GET /api/v1/users/me`
- **更新用户信息**
  - `PUT /api/v1/users/me`
- **获取下级代理列表**
  - `GET /api/v1/agents/subordinates`

### 5.2 项目服务 (Project Service)

- **创建项目**
  - `POST /api/v1/projects`
- **获取项目列表**
  - `GET /api/v1/projects`
- **获取项目详情**
  - `GET /api/v1/projects/{projectId}`
- **更新项目**
  - `PUT /api/v1/projects/{projectId}`

### 5.3 卡密服务 (Card Service)

- **生成卡密**
  - `POST /api/v1/cards/generate`
- **查询卡密列表**
  - `GET /api/v1/cards`
- **核销卡密 (外部接口)**
  - `POST /api/v1/cards/verify`
- **查询卡密 (外部接口)**
  - `GET /api/v1/cards/query`

### 5.4 额度服务 (Balance Service)

- **获取余额**
  - `GET /api/v1/balance`
- **充值**
  - `POST /api/v1/balance/recharge`
- **获取交易流水**
  - `GET /api/v1/transactions`

### 5.5 统计服务 (Statistics Service)

- **获取卡密统计**
  - `GET /api/v1/stats/cards`
- **获取额度统计**
  - `GET /api/v1/stats/balance`

### 5.6 系统服务 (System Service)

- **获取系统配置**
  - `GET /api/v1/system/config`
- **获取Banner列表**
  - `GET /api/v1/banners`

## 6. 分页、排序和过滤

- **分页**：使用 `page` 和 `pageSize` 参数。
  - `GET /api/v1/cards?page=1&pageSize=20`
- **排序**：使用 `sort` 和 `order` 参数。
  - `GET /api/v1/cards?sort=created_at&order=desc`
- **过滤**：在查询参数中直接添加过滤条件。
  - `GET /api/v1/cards?status=1&project_id=123`
