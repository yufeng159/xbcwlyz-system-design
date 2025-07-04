# API接口设计

## 1. 认证相关

### 1.1 用户注册
- **接口**: `POST /api/auth/register`
- **请求参数**: 
  ```json
  {
    "username": "string", // 用户名
    "password": "string", // 密码
    "email": "string",    // 邮箱
    "captcha": "string",  // 图形验证码
    "inviteCode": "string" // 邀请码（可选）
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "注册成功",
    "data": {
      "userId": 123,
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  }
  ```

### 1.2 用户登录
- **接口**: `POST /api/auth/login`
- **请求参数**: 
  ```json
  {
    "username": "string", // 用户名
    "password": "string", // 密码
    "captcha": "string"   // 图形验证码
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "登录成功",
    "data": {
      "userId": 123,
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "role": "agent" // 角色：super_admin, admin, agent
    }
  }
  ```

### 1.3 退出登录
- **接口**: `POST /api/auth/logout`
- **请求参数**: 
  ```json
  {
    "token": "string" // JWT令牌
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "退出成功"
  }
  ```

### 1.4 发送邮箱验证码
- **接口**: `POST /api/auth/send-code`
- **请求参数**: 
  ```json
  {
    "email": "string", // 邮箱地址
    "type": "string"   // 类型：register/reset
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "验证码已发送",
    "data": {
      "expireTime": 300 // 过期时间（秒）
    }
  }
  ```

### 1.5 验证邮箱验证码
- **接口**: `POST /api/auth/verify-code`
- **请求参数**: 
  ```json
  {
    "email": "string", // 邮箱地址
    "code": "string"   // 验证码
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "验证成功",
    "data": {
      "verified": true
    }
  }
  ```

### 1.6 获取图形验证码
- **接口**: `GET /api/auth/captcha`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "captchaId": "abc123",
      "captchaImage": "base64编码的图片"
    }
  }
  ```

### 1.7 重置密码
- **接口**: `POST /api/auth/reset-password`
- **请求参数**: 
  ```json
  {
    "email": "string",     // 邮箱地址
    "code": "string",      // 验证码
    "newPassword": "string" // 新密码
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "密码重置成功"
  }
  ```

## 2. 用户管理

### 2.1 获取个人信息
- **接口**: `GET /api/users/profile`
- **请求参数**: 无（从token获取用户ID）
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "userId": 123,
      "username": "agent001",
      "email": "agent001@example.com",
      "role": "agent",
      "balance": 1000.00,
      "createdAt": "2023-01-01T12:00:00Z"
    }
  }
  ```

### 2.2 更新个人信息
- **接口**: `PUT /api/users/profile`
- **请求参数**: 
  ```json
  {
    "nickname": "string", // 昵称
    "avatar": "string",   // 头像URL
    "phone": "string"     // 手机号
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "更新成功",
    "data": {
      "updated": true
    }
  }
  ```

### 2.3 修改密码
- **接口**: `PUT /api/users/change-password`
- **请求参数**: 
  ```json
  {
    "oldPassword": "string", // 旧密码
    "newPassword": "string"  // 新密码
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "密码修改成功"
  }
  ```

### 2.4 改绑管理员
- **接口**: `PUT /api/users/change-parent`
- **请求参数**: 
  ```json
  {
    "userId": 123,      // 用户ID
    "newParentId": 456  // 新上级ID
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "改绑成功"
  }
  ```

### 2.5 查看额度
- **接口**: `GET /api/users/balance`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "balance": 1000.00,
      "frozenBalance": 0.00,
      "totalIncome": 2000.00,
      "totalExpense": 1000.00
    }
  }
  ```

### 2.6 获取代理列表
- **接口**: `GET /api/users/proxies`
- **请求参数**: 
  ```
  page: 1       // 页码
  size: 10      // 每页数量
  keyword: "agent" // 搜索关键词（可选）
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "total": 100,
      "list": [
        {
          "id": 123,
          "username": "agent001",
          "balance": 1000.00,
          "createdCards": 50,
          "lastLoginTime": "2023-01-01T12:00:00Z"
        },
        // 更多代理...
      ]
    }
  }
  ```

### 2.7 添加代理
- **接口**: `POST /api/users/proxy`
- **请求参数**: 
  ```json
  {
    "username": "string",       // 用户名
    "password": "string",       // 密码
    "email": "string",          // 邮箱
    "initialBalance": 100.00    // 初始额度
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "添加成功",
    "data": {
      "proxyId": 123
    }
  }
  ```

### 2.8 剔除代理
- **接口**: `DELETE /api/users/proxy/{id}`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "剔除成功"
  }
  ```

## 3. 项目管理

### 3.1 获取项目列表
- **接口**: `GET /api/projects`
- **请求参数**: 
  ```
  page: 1       // 页码
  size: 10      // 每页数量
  keyword: "test" // 搜索关键词（可选）
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "total": 50,
      "list": [
        {
          "id": 1,
          "name": "测试项目",
          "description": "这是一个测试项目",
          "version": "1.0.0",
          "status": "active"
        },
        // 更多项目...
      ]
    }
  }
  ```

### 3.2 获取项目详情
- **接口**: `GET /api/projects/{id}`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "id": 1,
      "name": "测试项目",
      "description": "这是一个测试项目",
      "version": "1.0.0",
      "apiKey": "ak_123456",
      "secretKey": "sk_123456",
      "callbackUrl": "https://example.com/callback",
      "status": "active",
      "createdAt": "2023-01-01T12:00:00Z",
      "updatedAt": "2023-01-02T12:00:00Z"
    }
  }
  ```

### 3.3 创建项目
- **接口**: `POST /api/projects`
- **请求参数**: 
  ```json
  {
    "name": "string",        // 项目名称
    "description": "string", // 项目描述
    "version": "string",     // 版本号
    "callbackUrl": "string"  // 回调URL
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "创建成功",
    "data": {
      "projectId": 1,
      "apiKey": "ak_123456",
      "secretKey": "sk_123456"
    }
  }
  ```

### 3.4 更新项目
- **接口**: `PUT /api/projects/{id}`
- **请求参数**: 
  ```json
  {
    "name": "string",        // 项目名称
    "description": "string", // 项目描述
    "version": "string",     // 版本号
    "callbackUrl": "string", // 回调URL
    "status": "string"       // 状态：active/suspended
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "更新成功"
  }
  ```

### 3.5 删除项目
- **接口**: `DELETE /api/projects/{id}`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "删除成功"
  }
  ```

### 3.6 授权项目
- **接口**: `POST /api/projects/{id}/authorize`
- **请求参数**: 
  ```json
  {
    "userId": 123,                         // 用户ID
    "expirationDate": "2023-12-31T23:59:59Z" // 过期时间
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "授权成功"
  }
  ```

### 3.7 申请项目
- **接口**: `POST /api/projects/{id}/apply`
- **请求参数**: 
  ```json
  {
    "reason": "string" // 申请理由
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "申请成功",
    "data": {
      "applicationId": 1
    }
  }
  ```

### 3.8 获取项目统计数据
- **接口**: `GET /api/projects/{id}/statistics`
- **请求参数**: 
  ```
  startDate: "2023-01-01" // 开始日期
  endDate: "2023-01-31"   // 结束日期
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "totalCards": 1000,
      "activeCards": 800,
      "expiredCards": 200,
      "dailyActivation": [
        {
          "date": "2023-01-01",
          "count": 10
        },
        // 更多日期...
      ]
    }
  }
  ```

## 4. 卡密管理

### 4.1 获取卡密列表
- **接口**: `GET /api/cards`
- **请求参数**: 
  ```
  page: 1           // 页码
  size: 10          // 每页数量
  projectId: 1      // 项目ID（可选）
  status: "unused"  // 状态（可选）：unused/used/expired/disabled
  keyword: "test"   // 搜索关键词（可选）
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "total": 1000,
      "list": [
        {
          "id": 1,
          "cardNo": "ABCD-1234-5678-EFGH",
          "projectId": 1,
          "projectName": "测试项目",
          "status": "unused",
          "createdAt": "2023-01-01T12:00:00Z",
          "expireAt": "2023-12-31T23:59:59Z",
          "activatedAt": null
        },
        // 更多卡密...
      ]
    }
  }
  ```

### 4.2 生成卡密
- **接口**: `POST /api/cards/generate`
- **请求参数**: 
  ```json
  {
    "projectId": 1,     // 项目ID
    "count": 10,        // 生成数量
    "duration": 30,     // 有效期（天）
    "remark": "string"  // 备注
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "生成成功",
    "data": {
      "batchId": "B20230101001",
      "cardCount": 10,
      "costBalance": 100.00
    }
  }
  ```

### 4.3 验证卡密
- **接口**: `POST /api/cards/verify`
- **请求参数**: 
  ```json
  {
    "projectId": 1,           // 项目ID
    "cardNo": "string",       // 卡密
    "deviceId": "string",     // 设备ID
    "deviceInfo": "string"    // 设备信息（JSON格式）
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "验证成功",
    "data": {
      "valid": true,
      "expireAt": "2023-12-31T23:59:59Z",
      "projectInfo": {
        "name": "测试项目",
        "version": "1.0.0"
      }
    }
  }
  ```

### 4.4 解绑卡密
- **接口**: `POST /api/cards/unbind`
- **请求参数**: 
  ```json
  {
    "cardNo": "string", // 卡密
    "reason": "string"  // 解绑原因
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "解绑成功"
  }
  ```

### 4.5 导出卡密
- **接口**: `GET /api/cards/export`
- **请求参数**: 
  ```
  batchId: "B20230101001" // 批次ID（可选）
  projectId: 1           // 项目ID（可选）
  ```
- **响应**: 文件下载

### 4.6 获取卡密详情
- **接口**: `GET /api/cards/{id}`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "id": 1,
      "cardNo": "ABCD-1234-5678-EFGH",
      "projectId": 1,
      "projectName": "测试项目",
      "status": "used",
      "createdAt": "2023-01-01T12:00:00Z",
      "expireAt": "2023-12-31T23:59:59Z",
      "activatedAt": "2023-01-02T12:00:00Z",
      "deviceId": "D123456",
      "deviceInfo": "{\"os\":\"Windows\",\"version\":\"10\",\"mac\":\"00:11:22:33:44:55\"}",
      "activationHistory": [
        {
          "time": "2023-01-02T12:00:00Z",
          "ip": "192.168.1.1",
          "deviceId": "D123456"
        }
      ]
    }
  }
  ```

## 5. 额度管理

### 5.1 额度变动记录
- **接口**: `GET /api/balance/history`
- **请求参数**: 
  ```
  page: 1                // 页码
  size: 10               // 每页数量
  type: "recharge"       // 类型（可选）：recharge/transfer_in/transfer_out/consume/refund
  startDate: "2023-01-01" // 开始日期（可选）
  endDate: "2023-01-31"   // 结束日期（可选）
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "total": 100,
      "list": [
        {
          "id": 1,
          "type": "recharge",
          "amount": 1000.00,
          "balance": 1000.00,
          "description": "管理员充值",
          "createdAt": "2023-01-01T12:00:00Z"
        },
        // 更多记录...
      ]
    }
  }
  ```

### 5.2 转账额度
- **接口**: `POST /api/balance/transfer`
- **请求参数**: 
  ```json
  {
    "targetUserId": 123,  // 目标用户ID
    "amount": 100.00,     // 金额
    "remark": "string"    // 备注
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "转账成功",
    "data": {
      "transactionId": 1,
      "currentBalance": 900.00
    }
  }
  ```

### 5.3 充值额度
- **接口**: `POST /api/balance/recharge`
- **请求参数**: 
  ```json
  {
    "userId": 123,      // 用户ID
    "amount": 1000.00,  // 金额
    "remark": "string"  // 备注
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "充值成功",
    "data": {
      "transactionId": 1
    }
  }
  ```

### 5.4 提现申请
- **接口**: `POST /api/balance/withdraw`
- **请求参数**: 
  ```json
  {
    "amount": 100.00,                // 金额
    "accountInfo": "{\"type\":\"alipay\",\"account\":\"example@alipay.com\"}" // 账户信息（JSON格式）
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "申请成功",
    "data": {
      "withdrawId": 1
    }
  }
  ```

## 6. 系统管理

### 6.1 获取系统配置
- **接口**: `GET /api/system/configs`
- **请求参数**: 
  ```
  configType: "system" // 配置类型（可选）：system/payment/notification
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "configs": {
        "site_name": "小白菜网络验证系统",
        "site_logo": "https://example.com/logo.png",
        "card_price": "10.00"
      }
    }
  }
  ```

### 6.2 更新系统配置
- **接口**: `PUT /api/system/configs`
- **请求参数**: 
  ```json
  {
    "configs": {
      "site_name": "小白菜网络验证系统",
      "site_logo": "https://example.com/logo.png",
      "card_price": "10.00"
    }
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "更新成功"
  }
  ```

### 6.3 获取轮播图
- **接口**: `GET /api/system/banners`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "banners": [
        {
          "id": 1,
          "imageUrl": "https://example.com/banner1.jpg",
          "linkUrl": "https://example.com/page1",
          "order": 1
        },
        // 更多轮播图...
      ]
    }
  }
  ```

### 6.4 添加轮播图
- **接口**: `POST /api/system/banners`
- **请求参数**: 
  ```json
  {
    "imageUrl": "string", // 图片URL
    "linkUrl": "string",  // 链接URL
    "order": 1            // 排序
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "添加成功",
    "data": {
      "bannerId": 1
    }
  }
  ```

### 6.5 更新轮播图
- **接口**: `PUT /api/system/banners/{id}`
- **请求参数**: 
  ```json
  {
    "imageUrl": "string", // 图片URL
    "linkUrl": "string",  // 链接URL
    "order": 1            // 排序
  }
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "更新成功"
  }
  ```

### 6.6 删除轮播图
- **接口**: `DELETE /api/system/banners/{id}`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "删除成功"
  }
  ```

### 6.7 获取系统日志
- **接口**: `GET /api/system/logs`
- **请求参数**: 
  ```
  page: 1                // 页码
  size: 10               // 每页数量
  level: "error"         // 日志级别（可选）：info/warning/error
  module: "auth"         // 模块（可选）
  startDate: "2023-01-01" // 开始日期（可选）
  endDate: "2023-01-31"   // 结束日期（可选）
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "total": 1000,
      "list": [
        {
          "id": 1,
          "level": "error",
          "module": "auth",
          "content": "登录失败：密码错误",
          "ip": "192.168.1.1",
          "userId": 123,
          "createdAt": "2023-01-01T12:00:00Z"
        },
        // 更多日志...
      ]
    }
  }
  ```

## 7. 统计数据

### 7.1 概览统计
- **接口**: `GET /api/statistics/overview`
- **请求参数**: 无
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "userCount": 1000,
      "projectCount": 50,
      "cardCount": 10000,
      "todayActivation": 100,
      "weekActivation": 500,
      "monthActivation": 2000
    }
  }
  ```

### 7.2 卡密统计
- **接口**: `GET /api/statistics/cards`
- **请求参数**: 
  ```
  projectId: 1           // 项目ID（可选）
  startDate: "2023-01-01" // 开始日期
  endDate: "2023-01-31"   // 结束日期
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "total": 1000,
      "active": 800,
      "expired": 200,
      "daily": [
        {
          "date": "2023-01-01",
          "generated": 100,
          "activated": 80
        },
        // 更多日期...
      ]
    }
  }
  ```

### 7.3 用户统计
- **接口**: `GET /api/statistics/users`
- **请求参数**: 
  ```
  startDate: "2023-01-01" // 开始日期
  endDate: "2023-01-31"   // 结束日期
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "totalUsers": 1000,
      "newUsers": 100,
      "activeUsers": 500,
      "userGrowth": [
        {
          "date": "2023-01-01",
          "count": 10
        },
        // 更多日期...
      ]
    }
  }
  ```

### 7.4 收入统计
- **接口**: `GET /api/statistics/income`
- **请求参数**: 
  ```
  startDate: "2023-01-01" // 开始日期
  endDate: "2023-01-31"   // 结束日期
  ```
- **响应**: 
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": {
      "totalIncome": 10000.00,
      "monthlyIncome": 2000.00,
      "dailyIncome": [
        {
          "date": "2023-01-01",
          "amount": 100.00
        },
        // 更多日期...
      ]
    }
  }
  ```

## 8. 错误码说明

| 错误码 | 说明 |
|-------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 409 | 资源冲突 |
| 429 | 请求过于频繁 |
| 500 | 服务器内部错误 |

## 9. 接口认证

除了登录、注册等公开接口外，其他接口都需要在请求头中携带JWT Token进行认证：

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 10. 接口版本控制

接口支持版本控制，可以通过以下两种方式指定API版本：

1. URL路径：`/api/v1/users/profile`
2. 请求头：`Accept: application/json; version=1.0`

默认使用最新版本的API。