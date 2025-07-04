# 开发规范

## 1. 概述

本规范旨在统一团队的开发标准，提高代码质量、可读性和可维护性，降低协作成本。

## 2. 分支管理规范 (Git Flow)

- **main**: 主分支，用于存放对外发布的版本，任何时候都处于可发布状态。
- **develop**: 开发分支，用于日常开发，存放最新的开发版代码。
- **feature/**: 功能分支，用于开发新功能，以 `feature/` 开头，如 `feature/user-login`。
- **release/**: 发布分支，用于发布新版本，以 `release/` 开头，如 `release/v1.2.0`。
- **hotfix/**: 热修复分支，用于修复线上紧急Bug，以 `hotfix/` 开头，如 `hotfix/fix-login-bug`。

## 3. 代码风格规范

### 3.1 后端 (Java)

- **代码格式化**: 遵循 [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)。
- **命名规范**:
  - 类名：`UpperCamelCase`
  - 方法名、变量名：`lowerCamelCase`
  - 常量：`UPPER_SNAKE_CASE`
  - 包名：`lowercase`
- **注释规范**:
  - 公共方法和类必须有JavaDoc注释。
  - 复杂代码块需要有行内注释解释其逻辑。

### 3.2 前端 (Vue)

- **代码格式化**: 使用 Prettier 和 ESLint 自动格式化代码。
- **命名规范**:
  - 组件名：`PascalCase` (e.g., `UserProfile.vue`)
  - Props, data, methods: `camelCase`
- **组件化规范**:
  - 单文件组件 (`.vue`)。
  - 组件Props定义清晰，包含类型、默认值和是否必需。
  - 优先使用 `emit` 与父组件通信。

## 4. Commit Message 规范

采用 [AngularJS Git Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a150)。

格式: `<type>(<scope>): <subject>`

- **type**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- **scope**: 本次提交影响的范围 (可选)
- **subject**: 简短描述

示例:
- `feat(auth): add jwt authentication`
- `fix(card): resolve card verification issue`

## 5. API 设计规范

- 遵循本文档库中的 `api-design.md`。

## 6. 测试规范

- **单元测试**: 使用 JUnit (后端) 和 Vitest (前端) 编写单元测试，核心业务逻辑的测试覆盖率应达到80%以上。
- **集成测试**: 对涉及多个服务交互的业务流程进行集成测试。
- **端到端测试**: 使用 Cypress 或 Playwright 对关键用户流程进行端到端测试。

## 7. Code Review 规范

- **时机**: 功能开发完成后，提交Merge Request时进行。
- **要求**: 至少需要一名其他开发人员 `Approve` 后方可合并。
- **关注点**:
  - 是否符合设计文档和需求。
  - 是否遵循代码规范。
  - 是否存在潜在的Bug或性能问题。
  - 是否有必要的测试用例。

## 8. 文档规范

- **API文档**: 使用 Swagger/OpenAPI 自动生成并维护最新的API文档。
- **设计文档**: 重要的架构、数据库、安全等设计需要在 `docs` 目录中创建或更新相应的Markdown文档。
- **README.md**: 每个微服务项目都应有详细的 `README.md`，说明项目功能、如何启动、环境变量等。