# 系统架构设计

## 1. 架构概述

### 1.1 架构设计原则

小白菜网络验证卡密系统的架构设计遵循以下核心原则：

- **高内聚低耦合**：系统各模块功能内聚，接口清晰，降低模块间依赖
- **可扩展性**：支持水平扩展和垂直扩展，满足业务增长需求
- **高可用性**：消除单点故障，确保系统7*24小时稳定运行
- **安全性**：多层次安全防护，保障数据和业务安全
- **可维护性**：统一规范和标准，便于系统维护和升级
- **性能优化**：合理设计数据结构和算法，优化系统性能

### 1.2 整体架构

系统采用现代化的微服务架构，主要分为以下几层：

```
+------------------+    +------------------+    +------------------+
|                  |    |                  |    |                  |
|  客户端层        |    |  接入层          |    |  API网关层       |
|                  |    |                  |    |                  |
+------------------+    +------------------+    +------------------+
         |                       |                       |
         v                       v                       v
+------------------+    +------------------+    +------------------+
|                  |    |                  |    |                  |
|  微服务层        |    |  数据层          |    |  基础设施层      |
|                  |    |                  |    |                  |
+------------------+    +------------------+    +------------------+
```

## 2. 技术栈选型

### 2.1 后端技术栈

| 类别 | 技术选型 | 说明 |
|------|---------|------|
| 开发语言 | Java 17 | 稳定、成熟、生态丰富 |
| 开发框架 | Spring Boot 3.x | 简化开发，提高效率 |
| 微服务框架 | Spring Cloud Alibaba | 提供完整的微服务解决方案 |
| 服务注册与发现 | Nacos | 动态服务发现、配置管理 |
| 服务网关 | Spring Cloud Gateway | 高性能API网关 |
| 负载均衡 | Spring Cloud LoadBalancer | 客户端负载均衡 |
| 服务熔断 | Sentinel | 流量控制、熔断降级 |
| 分布式事务 | Seata | 微服务分布式事务解决方案 |
| 消息队列 | RocketMQ | 高吞吐量、可靠性强 |
| 数据库 | PostgreSQL 15 | 强大的关系型数据库 |
| 缓存 | Redis 7.x | 高性能分布式缓存 |
| 搜索引擎 | Elasticsearch 8.x | 全文搜索和分析 |
| ORM框架 | MyBatis-Plus | 增强的ORM框架 |
| API文档 | Knife4j (Swagger) | API文档自动生成 |
| 安全框架 | Spring Security | 认证和授权 |
| 定时任务 | XXL-Job | 分布式任务调度平台 |
| 监控 | Prometheus + Grafana | 系统监控和可视化 |
| 日志 | ELK Stack | 日志收集、分析和展示 |
| 容器化 | Docker | 应用容器化 |
| 容器编排 | Kubernetes | 容器编排和管理 |

### 2.2 前端技术栈

| 类别 | 技术选型 | 说明 |
|------|---------|------|
| 开发框架 | Vue 3 | 渐进式JavaScript框架 |
| 构建工具 | Vite | 下一代前端构建工具 |
| 状态管理 | Pinia | 轻量级状态管理库 |
| UI组件库 | Element Plus | 基于Vue 3的组件库 |
| HTTP客户端 | Axios | 基于Promise的HTTP客户端 |
| 路由 | Vue Router | Vue官方路由 |
| 图表 | ECharts | 功能丰富的图表库 |
| 工具库 | lodash | 实用工具库 |
| CSS预处理器 | SCSS | 增强CSS功能 |
| 代码规范 | ESLint + Prettier | 代码质量和格式化 |

## 3. 系统分层设计

### 3.1 客户端层

客户端层包括各种用户交互界面和接入系统的客户端应用：

- **Web前端**：基于Vue 3的响应式Web应用，适配PC和移动端
- **移动应用**：未来可扩展的iOS和Android原生应用
- **桌面客户端**：基于Electron的跨平台桌面应用
- **第三方系统**：通过API接口集成的外部系统

### 3.2 接入层

接入层负责处理来自各种客户端的请求，提供统一的接入点：

- **负载均衡**：使用Nginx实现请求分发和负载均衡
- **静态资源**：CDN加速静态资源分发
- **DDoS防护**：流量清洗和攻击防护
- **SSL终结**：处理HTTPS加密和解密

### 3.3 API网关层

API网关层是所有API请求的统一入口，提供以下功能：

- **路由转发**：将请求路由到相应的微服务
- **认证授权**：统一的认证和授权处理
- **限流熔断**：流量控制和服务保护
- **请求转换**：协议转换和请求/响应转换
- **日志监控**：请求日志记录和监控

### 3.4 微服务层

微服务层是系统的核心业务逻辑层，按照业务领域划分为多个微服务：

#### 3.4.1 认证服务 (auth-service)

负责用户认证、授权和会话管理：

- 用户登录和注册
- 令牌生成和验证
- 多因素认证
- 权限管理
- 会话管理

#### 3.4.2 用户服务 (user-service)

负责用户信息管理和用户关系管理：

- 用户信息CRUD
- 用户角色管理
- 代理层级关系
- 用户配置管理
- 用户日志记录

#### 3.4.3 项目服务 (project-service)

负责项目管理和授权管理：

- 项目信息CRUD
- 项目授权管理
- 项目申请处理
- 项目统计分析
- API密钥管理

#### 3.4.4 卡密服务 (card-service)

负责卡密生成、验证和管理：

- 卡密生成和批量管理
- 卡密验证和激活
- 卡密状态管理
- 设备绑定和解绑
- 卡密统计分析

#### 3.4.5 额度服务 (balance-service)

负责额度管理和交易处理：

- 额度充值和消费
- 额度转账和冻结
- 提现申请和处理
- 交易记录管理
- 额度统计分析

#### 3.4.6 通知服务 (notification-service)

负责系统通知和消息推送：

- 系统公告管理
- 邮件通知
- 短信通知
- 站内消息
- 消息模板管理

#### 3.4.7 统计服务 (statistics-service)

负责数据统计和分析：

- 实时数据统计
- 历史数据分析
- 报表生成
- 数据导出
- 数据可视化

#### 3.4.8 系统服务 (system-service)

负责系统配置和管理：

- 系统参数配置
- 轮播图管理
- 日志管理
- 定时任务
- 系统监控

### 3.5 数据层

数据层负责数据存储和管理：

- **关系型数据库**：PostgreSQL存储结构化业务数据
- **缓存**：Redis用于缓存和分布式锁
- **消息队列**：RocketMQ用于异步消息处理
- **搜索引擎**：Elasticsearch用于全文搜索和日志分析
- **对象存储**：MinIO用于存储文件和图片

### 3.6 基础设施层

基础设施层提供系统运行的基础环境：

- **容器化**：Docker容器化应用部署
- **容器编排**：Kubernetes管理容器集群
- **服务网格**：Istio提供服务间通信和治理
- **CI/CD**：Jenkins实现持续集成和部署
- **监控告警**：Prometheus和Grafana实现监控和可视化

## 4. 微服务详细设计

### 4.1 认证服务 (auth-service)

#### 4.1.1 功能模块

- **认证模块**：处理用户登录、注销和会话管理
- **授权模块**：处理权限验证和访问控制
- **令牌模块**：生成和验证JWT令牌
- **安全模块**：处理密码加密和安全策略

#### 4.1.2 接口设计

```java
// 认证接口
public interface AuthenticationService {
    AuthResult login(LoginRequest request);
    void logout(String token);
    AuthResult refreshToken(String refreshToken);
    boolean validateToken(String token);
    AuthResult register(RegisterRequest request);
    void sendVerificationCode(String email, VerificationType type);
    boolean verifyCode(String email, String code, VerificationType type);
}

// 授权接口
public interface AuthorizationService {
    boolean hasPermission(String userId, String resource, String action);
    List<Permission> getUserPermissions(String userId);
    void grantPermission(String userId, String resource, String action);
    void revokePermission(String userId, String resource, String action);
}
```

#### 4.1.3 数据模型

```java
// 用户认证信息
public class UserAuth {
    private String userId;
    private String username;
    private String passwordHash;
    private String email;
    private String phone;
    private UserStatus status;
    private Date lastLoginTime;
    private String lastLoginIp;
    private List<String> roles;
    private List<String> permissions;
    // getters and setters
}

// 认证令牌
public class AuthToken {
    private String userId;
    private String accessToken;
    private String refreshToken;
    private Date issuedAt;
    private Date expiresAt;
    private String clientIp;
    private String deviceInfo;
    // getters and setters
}
```

### 4.2 用户服务 (user-service)

#### 4.2.1 功能模块

- **用户管理模块**：处理用户信息的CRUD操作
- **角色管理模块**：处理用户角色分配和管理
- **代理关系模块**：管理代理层级关系
- **用户配置模块**：管理用户个性化配置

#### 4.2.2 接口设计

```java
// 用户管理接口
public interface UserService {
    UserDTO createUser(UserCreateRequest request);
    UserDTO updateUser(String userId, UserUpdateRequest request);
    UserDTO getUserById(String userId);
    UserDTO getUserByUsername(String username);
    Page<UserDTO> queryUsers(UserQueryRequest request, Pageable pageable);
    void deleteUser(String userId);
    void changePassword(String userId, PasswordChangeRequest request);
    void updateUserStatus(String userId, UserStatus status);
}

// 代理关系接口
public interface AgentRelationService {
    void addSubAgent(String agentId, String subAgentId);
    void removeSubAgent(String agentId, String subAgentId);
    List<UserDTO> getSubAgents(String agentId, Pageable pageable);
    UserDTO getParentAgent(String agentId);
    List<UserDTO> getAgentChain(String agentId);
}
```

#### 4.2.3 数据模型

```java
// 用户信息
public class User {
    private String id;
    private String username;
    private String email;
    private String phone;
    private String role;
    private String parentId;
    private BigDecimal balance;
    private BigDecimal frozenBalance;
    private UserStatus status;
    private Date lastLoginTime;
    private String lastLoginIp;
    private Date createdAt;
    private Date updatedAt;
    // getters and setters
}

// 用户配置
public class UserConfig {
    private String id;
    private String userId;
    private String configKey;
    private String configValue;
    private Date createdAt;
    private Date updatedAt;
    // getters and setters
}
```

### 4.3 项目服务 (project-service)

#### 4.3.1 功能模块

- **项目管理模块**：处理项目信息的CRUD操作
- **授权管理模块**：管理项目授权关系
- **申请处理模块**：处理项目授权申请
- **API密钥模块**：管理项目API密钥

#### 4.3.2 接口设计

```java
// 项目管理接口
public interface ProjectService {
    ProjectDTO createProject(ProjectCreateRequest request);
    ProjectDTO updateProject(String projectId, ProjectUpdateRequest request);
    ProjectDTO getProjectById(String projectId);
    Page<ProjectDTO> queryProjects(ProjectQueryRequest request, Pageable pageable);
    void deleteProject(String projectId);
    void updateProjectStatus(String projectId, ProjectStatus status);
    ApiKeyPair generateApiKey(String projectId);
    void revokeApiKey(String projectId);
}

// 项目授权接口
public interface ProjectAuthorizationService {
    void authorizeAgent(ProjectAuthRequest request);
    void revokeAuthorization(String projectId, String agentId);
    List<ProjectAuthDTO> getAgentAuthorizations(String agentId);
    List<AgentAuthDTO> getProjectAuthorizations(String projectId);
    void processAuthApplication(String applicationId, AuthApplicationResult result);
    Page<AuthApplicationDTO> getAuthApplications(AuthApplicationQueryRequest request, Pageable pageable);
}
```

#### 4.3.3 数据模型

```java
// 项目信息
public class Project {
    private String id;
    private String name;
    private String description;
    private String version;
    private String apiKey;
    private String secretKey;
    private String callbackUrl;
    private ProjectStatus status;
    private String createdBy;
    private Date createdAt;
    private Date updatedAt;
    // getters and setters
}

// 项目授权
public class ProjectAuthorization {
    private String id;
    private String projectId;
    private String userId;
    private String authorizedBy;
    private Date expirationDate;
    private AuthorizationStatus status;
    private Date createdAt;
    private Date updatedAt;
    // getters and setters
}
```

### 4.4 卡密服务 (card-service)

#### 4.4.1 功能模块

- **卡密生成模块**：处理卡密生成和批量管理
- **卡密验证模块**：处理卡密验证和激活
- **设备绑定模块**：管理卡密与设备的绑定关系
- **卡密查询模块**：提供卡密查询和统计功能

#### 4.4.2 接口设计

```java
// 卡密管理接口
public interface CardService {
    BatchGenerateResult generateCards(CardGenerateRequest request);
    CardDTO getCardById(String cardId);
    CardDTO getCardByCardNo(String cardNo);
    Page<CardDTO> queryCards(CardQueryRequest request, Pageable pageable);
    void updateCardStatus(String cardId, CardStatus status);
    void unbindDevice(String cardId);
    ExportResult exportCards(CardExportRequest request);
}

// 卡密验证接口
public interface CardVerificationService {
    VerificationResult verifyCard(CardVerificationRequest request);
    VerificationResult activateCard(CardActivationRequest request);
    DeviceBindResult bindDevice(String cardNo, DeviceBindRequest request);
    List<CardDTO> getDeviceCards(String deviceId, String projectId);
}
```

#### 4.4.3 数据模型

```java
// 卡密信息
public class Card {
    private String id;
    private String cardNo;
    private String projectId;
    private String batchId;
    private String createdBy;
    private CardStatus status;
    private Integer duration;
    private String deviceId;
    private String deviceInfo;
    private String remark;
    private Date createdAt;
    private Date activatedAt;
    private Date expireAt;
    // getters and setters
}

// 卡密批次
public class CardBatch {
    private String id;
    private String batchId;
    private String projectId;
    private String createdBy;
    private Integer cardCount;
    private Integer duration;
    private BigDecimal costBalance;
    private String remark;
    private Date createdAt;
    // getters and setters
}
```

### 4.5 额度服务 (balance-service)

#### 4.5.1 功能模块

- **额度管理模块**：处理额度变动和查询
- **交易处理模块**：处理充值、消费、转账等交易
- **提现管理模块**：处理提现申请和审核
- **额度统计模块**：提供额度统计和分析功能

#### 4.5.2 接口设计

```java
// 额度管理接口
public interface BalanceService {
    BigDecimal getUserBalance(String userId);
    BigDecimal getUserFrozenBalance(String userId);
    BalanceChangeResult changeBalance(BalanceChangeRequest request);
    Page<BalanceRecordDTO> getBalanceRecords(String userId, BalanceRecordQueryRequest request, Pageable pageable);
    BalanceStatisticsDTO getBalanceStatistics(String userId, DateRange dateRange);
}

// 交易处理接口
public interface TransactionService {
    TransactionResult recharge(RechargeRequest request);
    TransactionResult transfer(TransferRequest request);
    TransactionResult consume(ConsumeRequest request);
    TransactionResult refund(RefundRequest request);
    WithdrawalResult applyWithdrawal(WithdrawalRequest request);
    void processWithdrawal(String withdrawalId, WithdrawalProcessRequest request);
    Page<WithdrawalDTO> queryWithdrawals(WithdrawalQueryRequest request, Pageable pageable);
}
```

#### 4.5.3 数据模型

```java
// 额度记录
public class BalanceRecord {
    private String id;
    private String userId;
    private BalanceChangeType type;
    private BigDecimal amount;
    private BigDecimal balance;
    private String relatedUserId;
    private String relatedId;
    private String description;
    private Date createdAt;
    // getters and setters
}

// 提现申请
public class WithdrawalRequest {
    private String id;
    private String userId;
    private BigDecimal amount;
    private String accountInfo;
    private WithdrawalStatus status;
    private String processedBy;
    private Date processedAt;
    private String remark;
    private Date createdAt;
    private Date updatedAt;
    // getters and setters
}
```

## 5. 数据库设计

### 5.1 数据库架构

系统采用分库分表设计，提高数据库性能和可扩展性：

- **水平分库**：按业务领域划分为多个数据库
- **垂直分表**：将大表按功能拆分为多个表
- **水平分表**：对高频访问大表按ID哈希分片

数据库架构如下：

```
+-------------------+    +-------------------+    +-------------------+
|                   |    |                   |    |                   |
|  auth_db          |    |  user_db          |    |  project_db       |
|  (认证相关数据)    |    |  (用户相关数据)    |    |  (项目相关数据)    |
|                   |    |                   |    |                   |
+-------------------+    +-------------------+    +-------------------+

+-------------------+    +-------------------+    +-------------------+
|                   |    |                   |    |                   |
|  card_db          |    |  balance_db       |    |  system_db        |
|  (卡密相关数据)    |    |  (额度相关数据)    |    |  (系统相关数据)    |
|                   |    |                   |    |                   |
+-------------------+    +-------------------+    +-------------------+
```

### 5.2 读写分离

为提高数据库性能，系统实现读写分离：

- **主库**：处理所有写操作和核心读操作
- **从库**：处理非核心读操作和报表查询
- **路由策略**：基于SQL解析的读写分离路由

### 5.3 数据库高可用

采用以下策略确保数据库高可用：

- **主从复制**：实时数据同步
- **故障自动切换**：主库故障时自动切换到从库
- **数据库集群**：多节点部署，避免单点故障
- **定期备份**：全量备份和增量备份结合

## 6. 缓存设计

### 6.1 缓存架构

系统采用多级缓存架构，提高系统性能：

- **本地缓存**：应用内存缓存，适用于静态数据
- **分布式缓存**：Redis集群，适用于共享数据
- **数据库缓存**：数据库查询缓存

### 6.2 缓存策略

- **缓存穿透防护**：布隆过滤器过滤无效请求
- **缓存击穿防护**：热点数据永不过期策略
- **缓存雪崩防护**：过期时间随机化
- **缓存更新策略**：先更新数据库，再删除缓存

### 6.3 缓存数据

主要缓存以下数据：

- **用户会话**：用户登录状态和权限信息
- **配置信息**：系统配置和参数
- **卡密验证结果**：高频验证卡密的结果
- **统计数据**：预计算的统计结果
- **热点数据**：高频访问的业务数据

## 7. 消息队列设计

### 7.1 消息队列架构

系统使用RocketMQ作为消息队列中间件，实现异步处理和系统解耦：

- **生产者**：各微服务模块
- **消费者**：专门的消息处理服务
- **主题**：按业务领域划分

### 7.2 消息类型

- **事务消息**：确保分布式事务一致性
- **延时消息**：处理定时任务和超时处理
- **普通消息**：异步处理非关键业务
- **顺序消息**：保证消息处理顺序

### 7.3 应用场景

- **异步通知**：卡密激活、额度变动等通知
- **数据同步**：跨服务数据一致性维护
- **任务调度**：定时任务和批量处理
- **日志收集**：业务日志和审计日志收集
- **流量削峰**：处理突发高流量

## 8. 接口设计

### 8.1 API设计原则

- **RESTful风格**：资源化的URL设计
- **版本控制**：API版本在URL中明确标识
- **统一响应**：标准化的响应格式
- **参数验证**：严格的参数验证和错误处理
- **幂等性**：确保重复请求安全

### 8.2 API文档

使用Swagger自动生成API文档，包含以下内容：

- **接口描述**：功能说明和使用场景
- **请求参数**：参数名、类型、是否必须、示例值
- **响应结果**：状态码、响应结构、示例数据
- **错误码**：可能的错误码和错误信息
- **接口测试**：在线测试功能

### 8.3 接口安全

- **认证**：JWT令牌认证
- **授权**：基于角色和权限的访问控制
- **加密**：敏感数据传输加密
- **签名**：请求参数签名验证
- **限流**：基于用户和IP的请求限制

## 9. 部署架构

### 9.1 环境规划

系统划分为以下环境：

- **开发环境**：开发人员日常开发和单元测试
- **测试环境**：QA团队功能测试和集成测试
- **预发布环境**：与生产环境配置一致，用于最终验证
- **生产环境**：面向最终用户的正式环境

### 9.2 容器化部署

系统采用Docker容器化部署，Kubernetes编排管理：

- **容器镜像**：基于Alpine的轻量级镜像
- **容器编排**：Kubernetes管理容器生命周期
- **服务发现**：Kubernetes Service和DNS
- **配置管理**：ConfigMap和Secret管理配置
- **存储管理**：PersistentVolume管理持久化数据

### 9.3 高可用部署

- **多副本部署**：每个服务部署多个实例
- **跨区域部署**：跨可用区部署，避免单区故障
- **自动扩缩容**：根据负载自动调整实例数量
- **健康检查**：定期检查服务健康状态
- **自动恢复**：故障实例自动重启或重建

## 10. 监控与运维

### 10.1 监控系统

采用Prometheus + Grafana构建监控系统：

- **基础设施监控**：服务器、网络、存储等
- **容器监控**：容器资源使用和健康状态
- **应用监控**：JVM、线程、GC等
- **业务监控**：接口调用、成功率、响应时间等
- **日志监控**：错误日志和异常监控

### 10.2 告警系统

- **告警规则**：基于阈值和异常检测的告警规则
- **告警级别**：分为严重、警告、提示三级
- **告警渠道**：邮件、短信、钉钉等多渠道通知
- **告警抑制**：避免告警风暴
- **值班轮询**：告警自动分配给值班人员

### 10.3 日志管理

使用ELK Stack构建日志管理系统：

- **日志收集**：Filebeat收集应用日志
- **日志处理**：Logstash处理和转换日志
- **日志存储**：Elasticsearch存储和索引日志
- **日志展示**：Kibana可视化展示和查询
- **日志分析**：日志异常检测和趋势分析

### 10.4 CI/CD流水线

使用Jenkins构建CI/CD流水线：

- **代码检出**：从Git仓库检出代码
- **代码检查**：SonarQube代码质量检查
- **单元测试**：运行单元测试和覆盖率检查
- **构建打包**：构建应用和Docker镜像
- **自动部署**：部署到目标环境
- **自动测试**：运行自动化测试
- **发布确认**：人工确认正式发布

## 11. 系统扩展性

### 11.1 水平扩展

- **无状态服务**：所有微服务设计为无状态，支持水平扩展
- **数据分片**：数据库和缓存支持分片扩展
- **负载均衡**：请求自动分发到多个实例

### 11.2 垂直扩展

- **资源隔离**：核心服务独立部署，资源独占
- **性能优化**：针对性能瓶颈进行代码和架构优化
- **硬件升级**：关键节点硬件配置升级

### 11.3 功能扩展

- **插件机制**：支持功能插件扩展
- **API网关**：支持新服务无缝接入
- **配置中心**：动态配置，无需重启
- **服务发现**：自动发现和注册新服务

## 12. 灾备与恢复

### 12.1 灾备策略

- **数据备份**：定期全量备份和实时增量备份
- **异地多活**：核心系统跨区域部署
- **故障演练**：定期进行灾难恢复演练
- **应急预案**：详细的灾难应对流程

### 12.2 恢复流程

- **故障检测**：自动检测系统故障
- **故障隔离**：隔离故障组件，避免扩散
- **服务降级**：非核心功能自动降级
- **快速恢复**：按预案快速恢复核心功能
- **全面恢复**：逐步恢复所有功能

### 12.3 业务连续性

- **RTO(恢复时间目标)**：核心业务2小时内恢复
- **RPO(恢复点目标)**：数据丢失不超过5分钟
- **降级方案**：核心功能保证可用的降级方案
- **应急通道**：关键业务的备用处理通道