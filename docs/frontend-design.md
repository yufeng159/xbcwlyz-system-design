# 前端设计

## 1. 技术栈选型

本系统前端采用以下技术栈：

- **框架**：Vue 3 + Vite
- **UI组件库**：Element Plus
- **状态管理**：Pinia
- **路由**：Vue Router
- **HTTP客户端**：Axios
- **图表库**：ECharts
- **CSS预处理器**：SCSS
- **代码规范**：ESLint + Prettier

选型理由：
- Vue 3提供了Composition API，更好的TypeScript支持和更小的打包体积
- Element Plus提供了丰富的企业级UI组件
- Pinia相比Vuex更加轻量和类型友好
- Vite提供了极速的开发体验和优化的构建性能

## 2. 项目结构

```
src/
├── api/                # API请求模块
│   ├── auth.js         # 认证相关API
│   ├── user.js         # 用户相关API
│   ├── project.js      # 项目相关API
│   ├── card.js         # 卡密相关API
│   ├── balance.js      # 额度相关API
│   └── system.js       # 系统相关API
├── assets/             # 静态资源
│   ├── images/         # 图片资源
│   ├── styles/         # 全局样式
│   └── icons/          # 图标资源
├── components/         # 公共组件
│   ├── common/         # 通用组件
│   ├── layout/         # 布局组件
│   └── business/       # 业务组件
├── composables/        # 组合式函数
├── config/             # 配置文件
├── directives/         # 自定义指令
├── hooks/              # 自定义钩子
├── router/             # 路由配置
│   ├── index.js        # 路由入口
│   ├── routes.js       # 路由定义
│   └── guards.js       # 路由守卫
├── store/              # 状态管理
│   ├── modules/        # 状态模块
│   └── index.js        # 状态入口
├── utils/              # 工具函数
│   ├── request.js      # 请求封装
│   ├── auth.js         # 认证工具
│   ├── validator.js    # 验证工具
│   └── formatter.js    # 格式化工具
├── views/              # 页面组件
│   ├── public/         # 公共页面
│   ├── agent/          # 代理页面
│   ├── admin/          # 管理员页面
│   └── super-admin/    # 超级管理员页面
├── App.vue             # 根组件
└── main.js             # 入口文件
```

## 3. 响应式设计

系统采用响应式设计，适配不同尺寸的设备：

- **桌面端**：优化宽屏显示，提供更多的数据展示和操作空间
- **平板端**：调整布局，确保关键功能易于访问
- **移动端**：简化界面，聚焦核心功能，优化触摸操作

响应式断点设计：

```scss
// 移动端优先设计
$breakpoints: (
  'sm': 576px,   // 小型设备
  'md': 768px,   // 中型设备
  'lg': 992px,   // 大型设备
  'xl': 1200px,  // 超大型设备
  'xxl': 1400px  // 特大型设备
);
```

## 4. 主题设计

### 4.1 色彩系统

主色调采用蓝色系，体现专业性和科技感：

```scss
// 主题色
$primary-color: #1890ff;       // 主色
$primary-color-light: #40a9ff; // 主色-浅色
$primary-color-dark: #096dd9;  // 主色-深色

// 功能色
$success-color: #52c41a;       // 成功色
$warning-color: #faad14;       // 警告色
$error-color: #f5222d;         // 错误色
$info-color: #1890ff;          // 信息色

// 中性色
$text-primary: rgba(0, 0, 0, 0.85);    // 主要文字
$text-secondary: rgba(0, 0, 0, 0.65);   // 次要文字
$text-disabled: rgba(0, 0, 0, 0.45);    // 禁用文字
$border-color: #d9d9d9;                 // 边框颜色
$background-color: #f0f2f5;             // 背景色
```

### 4.2 字体系统

```scss
// 字体家族
$font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'Noto Sans', sans-serif;

// 字体大小
$font-size-xs: 12px;    // 超小号
$font-size-sm: 14px;    // 小号
$font-size-md: 16px;    // 中号
$font-size-lg: 18px;    // 大号
$font-size-xl: 20px;    // 超大号
$font-size-xxl: 24px;   // 特大号

// 字重
$font-weight-light: 300;    // 细体
$font-weight-normal: 400;   // 常规
$font-weight-medium: 500;   // 中粗
$font-weight-bold: 600;     // 粗体
```

### 4.3 暗黑模式

系统支持明亮和暗黑两种模式，用户可以根据偏好切换：

```scss
// 暗黑模式色彩变量
[data-theme="dark"] {
  --primary-color: #177ddc;
  --primary-color-light: #1f6fbb;
  --primary-color-dark: #3c9ae8;
  
  --success-color: #49aa19;
  --warning-color: #d89614;
  --error-color: #d32029;
  --info-color: #177ddc;
  
  --text-primary: rgba(255, 255, 255, 0.85);
  --text-secondary: rgba(255, 255, 255, 0.65);
  --text-disabled: rgba(255, 255, 255, 0.45);
  --border-color: #434343;
  --background-color: #141414;
}
```

## 5. 页面设计

### 5.1 公共页面

#### 5.1.1 登录页

![登录页设计](https://placeholder-for-login-page-design.com)

登录页包含以下元素：
- 系统Logo和名称
- 登录表单（用户名/密码）
- 验证码
- 记住密码选项
- 忘记密码链接
- 注册入口（仅对代理开放）

#### 5.1.2 注册页

注册页包含以下元素：
- 系统Logo和名称
- 注册表单（用户名、密码、确认密码、邮箱）
- 验证码
- 服务条款同意选项
- 登录入口

#### 5.1.3 忘记密码页

忘记密码页包含以下元素：
- 系统Logo和名称
- 邮箱输入框
- 验证码
- 重置密码按钮
- 登录入口

### 5.2 代理页面

#### 5.2.1 代理首页/仪表盘

代理首页展示关键数据和快捷操作：

- 数据概览区
  - 当前余额
  - 今日消耗
  - 卡密总数
  - 已用卡密数
- 快捷操作区
  - 生成卡密
  - 查询卡密
  - 充值额度
  - 申请项目
- 项目列表区
  - 已授权项目卡片
  - 每个项目卡片显示项目名称、卡密数量、使用情况
- 最近操作区
  - 最近生成的卡密批次
  - 最近验证的卡密

#### 5.2.2 卡密管理

卡密管理页面包含以下功能：

- 卡密列表
  - 搜索和筛选（项目、状态、批次、时间范围）
  - 分页显示
  - 每行显示卡密号、项目、状态、创建时间、到期时间等信息
  - 操作按钮（查看详情、导出等）
- 批量操作
  - 批量导出
  - 批量删除
- 生成卡密
  - 选择项目
  - 设置数量
  - 设置有效期
  - 添加备注

#### 5.2.3 额度管理

额度管理页面包含以下功能：

- 额度概览
  - 当前余额
  - 冻结额度
  - 累计充值
  - 累计消费
- 额度记录
  - 搜索和筛选（类型、时间范围）
  - 分页显示
  - 每行显示类型、金额、余额、时间、描述等信息
- 额度操作
  - 充值额度
  - 转账额度
  - 提现申请

#### 5.2.4 项目管理

项目管理页面包含以下功能：

- 项目列表
  - 已授权项目
  - 每个项目显示名称、描述、版本、状态等信息
  - 操作按钮（生成卡密、查看卡密等）
- 项目申请
  - 可申请项目列表
  - 申请表单（项目选择、申请理由）
- 申请记录
  - 显示历史申请记录及状态

#### 5.2.5 个人中心

个人中心页面包含以下功能：

- 基本信息
  - 用户名
  - 邮箱
  - 角色
  - 注册时间
  - 最后登录时间
- 安全设置
  - 修改密码
  - 绑定邮箱
  - 设置安全问题
- 通知消息
  - 系统通知
  - 个人消息

### 5.3 代理管理员页面

#### 5.3.1 管理员首页/仪表盘

管理员首页除了包含代理首页的内容外，还增加了：

- 下级代理概览
  - 代理总数
  - 今日新增
  - 活跃代理数
- 下级代理额度统计
  - 总额度
  - 已用额度
  - 剩余额度
- 项目授权统计
  - 各项目授权代理数量

#### 5.3.2 代理管理

代理管理页面包含以下功能：

- 代理列表
  - 搜索和筛选（用户名、状态、时间范围）
  - 分页显示
  - 每行显示用户名、邮箱、余额、状态、创建时间等信息
  - 操作按钮（查看详情、充值、禁用等）
- 添加代理
  - 代理信息表单（用户名、密码、邮箱等）
  - 初始额度设置
- 代理详情
  - 基本信息
  - 额度记录
  - 卡密统计
  - 操作记录

#### 5.3.3 项目授权管理

项目授权管理页面包含以下功能：

- 授权列表
  - 搜索和筛选（项目、代理、状态）
  - 分页显示
  - 每行显示项目、代理、授权时间、到期时间、状态等信息
  - 操作按钮（延期、撤销等）
- 添加授权
  - 选择项目
  - 选择代理
  - 设置有效期
- 授权申请处理
  - 待处理申请列表
  - 处理操作（同意、拒绝）

### 5.4 超级管理员页面

#### 5.4.1 超级管理员首页/仪表盘

超级管理员首页包含全局数据概览：

- 系统概览
  - 用户总数（按角色分类）
  - 项目总数
  - 卡密总数（按状态分类）
  - 系统总额度
- 趋势图表
  - 用户增长趋势
  - 卡密生成趋势
  - 卡密使用趋势
  - 额度消耗趋势
- 项目概览
  - 各项目卡密数量对比
  - 各项目使用情况对比
- 系统状态
  - 服务器状态
  - 数据库状态
  - 缓存状态

#### 5.4.2 项目管理

项目管理页面包含以下功能：

- 项目列表
  - 搜索和筛选（名称、状态、创建时间）
  - 分页显示
  - 每行显示名称、描述、版本、创建者、状态等信息
  - 操作按钮（编辑、禁用、查看详情等）
- 添加项目
  - 项目信息表单（名称、描述、版本等）
  - API配置（回调URL等）
- 项目详情
  - 基本信息
  - API密钥管理
  - 授权代理列表
  - 卡密统计

#### 5.4.3 系统配置

系统配置页面包含以下功能：

- 基本配置
  - 系统名称
  - Logo设置
  - 联系方式
  - 版权信息
- 安全配置
  - 密码策略
  - 登录限制
  - IP白名单
- 额度配置
  - 卡密生成额度规则
  - 提现规则
- 通知配置
  - 邮件服务器设置
  - 通知模板管理
- 轮播图管理
  - 轮播图列表
  - 添加/编辑轮播图

#### 5.4.4 日志管理

日志管理页面包含以下功能：

- 操作日志
  - 搜索和筛选（用户、模块、级别、时间范围）
  - 分页显示
  - 每行显示时间、用户、IP、模块、操作内容等信息
- 登录日志
  - 搜索和筛选（用户、状态、时间范围）
  - 分页显示
  - 每行显示时间、用户、IP、设备、状态等信息
- 系统日志
  - 搜索和筛选（级别、模块、时间范围）
  - 分页显示
  - 每行显示时间、级别、模块、内容等信息

## 6. 组件设计

### 6.1 通用组件

#### 6.1.1 PageHeader 页头组件

```vue
<template>
  <div class="page-header">
    <div class="page-header__title">
      <slot name="title">{{ title }}</slot>
    </div>
    <div class="page-header__actions">
      <slot name="actions"></slot>
    </div>
  </div>
</template>

<script setup>
defineProps({
  title: {
    type: String,
    default: ''
  }
});
</script>
```

#### 6.1.2 SearchForm 搜索表单组件

```vue
<template>
  <el-form :model="formData" :inline="true" class="search-form">
    <slot :form-data="formData"></slot>
    <el-form-item>
      <el-button type="primary" @click="handleSearch">搜索</el-button>
      <el-button @click="handleReset">重置</el-button>
    </el-form-item>
  </el-form>
</template>

<script setup>
import { reactive } from 'vue';

const props = defineProps({
  initialValues: {
    type: Object,
    default: () => ({})
  }
});

const emit = defineEmits(['search', 'reset']);

const formData = reactive({ ...props.initialValues });

const handleSearch = () => {
  emit('search', { ...formData });
};

const handleReset = () => {
  Object.keys(formData).forEach(key => {
    formData[key] = props.initialValues[key] || undefined;
  });
  emit('reset', { ...formData });
};
</script>
```

#### 6.1.3 DataTable 数据表格组件

```vue
<template>
  <div class="data-table">
    <el-table
      v-loading="loading"
      :data="data"
      :border="border"
      :stripe="stripe"
      :row-key="rowKey"
      @selection-change="handleSelectionChange"
    >
      <el-table-column
        v-if="selectable"
        type="selection"
        width="55"
      />
      <slot></slot>
    </el-table>
    
    <div class="data-table__pagination" v-if="showPagination">
      <el-pagination
        :current-page="currentPage"
        :page-size="pageSize"
        :total="total"
        :page-sizes="pageSizes"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
  </div>
</template>

<script setup>
defineProps({
  data: {
    type: Array,
    required: true
  },
  loading: {
    type: Boolean,
    default: false
  },
  border: {
    type: Boolean,
    default: true
  },
  stripe: {
    type: Boolean,
    default: true
  },
  rowKey: {
    type: String,
    default: 'id'
  },
  selectable: {
    type: Boolean,
    default: false
  },
  showPagination: {
    type: Boolean,
    default: true
  },
  currentPage: {
    type: Number,
    default: 1
  },
  pageSize: {
    type: Number,
    default: 10
  },
  total: {
    type: Number,
    default: 0
  },
  pageSizes: {
    type: Array,
    default: () => [10, 20, 50, 100]
  }
});

const emit = defineEmits(['selection-change', 'size-change', 'current-change']);

const handleSelectionChange = (selection) => {
  emit('selection-change', selection);
};

const handleSizeChange = (size) => {
  emit('size-change', size);
};

const handleCurrentChange = (page) => {
  emit('current-change', page);
};
</script>
```

### 6.2 业务组件

#### 6.2.1 CardGenerator 卡密生成组件

```vue
<template>
  <el-form :model="formData" label-width="100px">
    <el-form-item label="项目" required>
      <el-select v-model="formData.projectId" placeholder="请选择项目">
        <el-option
          v-for="item in projects"
          :key="item.id"
          :label="item.name"
          :value="item.id"
        />
      </el-select>
    </el-form-item>
    
    <el-form-item label="数量" required>
      <el-input-number
        v-model="formData.count"
        :min="1"
        :max="maxCount"
      />
    </el-form-item>
    
    <el-form-item label="有效期(天)" required>
      <el-input-number
        v-model="formData.duration"
        :min="1"
      />
    </el-form-item>
    
    <el-form-item label="备注">
      <el-input
        v-model="formData.remark"
        type="textarea"
        placeholder="请输入备注信息"
      />
    </el-form-item>
    
    <el-form-item>
      <el-button type="primary" @click="handleSubmit" :loading="loading">
        生成卡密
      </el-button>
      <el-button @click="handleReset">重置</el-button>
    </el-form-item>
  </el-form>
</template>

<script setup>
import { reactive, ref } from 'vue';
import { ElMessage } from 'element-plus';
import { generateCards } from '@/api/card';

const props = defineProps({
  projects: {
    type: Array,
    required: true
  },
  maxCount: {
    type: Number,
    default: 1000
  }
});

const emit = defineEmits(['success']);

const formData = reactive({
  projectId: '',
  count: 1,
  duration: 30,
  remark: ''
});

const loading = ref(false);

const handleSubmit = async () => {
  if (!formData.projectId) {
    ElMessage.warning('请选择项目');
    return;
  }
  
  loading.value = true;
  try {
    const result = await generateCards(formData);
    ElMessage.success(`成功生成${formData.count}张卡密`);
    emit('success', result);
    handleReset();
  } catch (error) {
    console.error(error);
    ElMessage.error(error.message || '生成卡密失败');
  } finally {
    loading.value = false;
  }
};

const handleReset = () => {
  formData.projectId = '';
  formData.count = 1;
  formData.duration = 30;
  formData.remark = '';
};
</script>
```

#### 6.2.2 CardVerifier 卡密验证组件

```vue
<template>
  <div class="card-verifier">
    <el-form :model="formData" label-width="100px">
      <el-form-item label="项目" required>
        <el-select v-model="formData.projectId" placeholder="请选择项目">
          <el-option
            v-for="item in projects"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          />
        </el-select>
      </el-form-item>
      
      <el-form-item label="卡密" required>
        <el-input
          v-model="formData.cardNo"
          placeholder="请输入卡密"
        />
      </el-form-item>
      
      <el-form-item label="设备ID" required>
        <el-input
          v-model="formData.deviceId"
          placeholder="请输入设备ID"
        />
      </el-form-item>
      
      <el-form-item>
        <el-button type="primary" @click="handleVerify" :loading="loading">
          验证卡密
        </el-button>
        <el-button @click="handleReset">重置</el-button>
      </el-form-item>
    </el-form>
    
    <div v-if="result" class="card-verifier__result">
      <el-alert
        :title="result.success ? '验证成功' : '验证失败'"
        :type="result.success ? 'success' : 'error'"
        :description="result.message"
        show-icon
      />
      
      <div v-if="result.success" class="card-verifier__info">
        <p><strong>卡密:</strong> {{ result.data.cardNo }}</p>
        <p><strong>状态:</strong> {{ result.data.status }}</p>
        <p><strong>激活时间:</strong> {{ result.data.activatedAt }}</p>
        <p><strong>过期时间:</strong> {{ result.data.expireAt }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue';
import { ElMessage } from 'element-plus';
import { verifyCard } from '@/api/card';

const props = defineProps({
  projects: {
    type: Array,
    required: true
  }
});

const formData = reactive({
  projectId: '',
  cardNo: '',
  deviceId: ''
});

const loading = ref(false);
const result = ref(null);

const handleVerify = async () => {
  if (!formData.projectId || !formData.cardNo || !formData.deviceId) {
    ElMessage.warning('请填写完整信息');
    return;
  }
  
  loading.value = true;
  try {
    const res = await verifyCard(formData);
    result.value = res;
  } catch (error) {
    console.error(error);
    result.value = {
      success: false,
      message: error.message || '验证失败'
    };
  } finally {
    loading.value = false;
  }
};

const handleReset = () => {
  formData.projectId = '';
  formData.cardNo = '';
  formData.deviceId = '';
  result.value = null;
};
</script>
```

## 7. 交互设计

### 7.1 表单交互

- **即时验证**：用户输入时进行字段验证，提供即时反馈
- **提交验证**：表单提交前进行完整性和有效性验证
- **错误提示**：在输入框下方显示明确的错误信息
- **成功反馈**：操作成功后显示成功提示，并根据场景自动跳转或清空表单

### 7.2 列表交互

- **批量操作**：支持选择多条记录进行批量操作
- **快速筛选**：提供常用筛选条件的快捷按钮
- **排序**：支持点击表头进行排序
- **分页**：支持调整每页显示数量和页码跳转
- **行内操作**：常用操作直接显示在行内，次要操作放在下拉菜单中

### 7.3 加载状态

- **全局加载**：页面初始加载时显示全屏加载状态
- **局部加载**：组件加载数据时显示局部加载状态
- **按钮加载**：操作按钮点击后显示加载状态，防止重复提交
- **骨架屏**：数据加载前显示内容骨架，减少用户等待感

### 7.4 消息通知

- **操作反馈**：操作成功或失败时显示消息提示
- **系统通知**：重要系统消息通过通知中心和消息弹窗提醒
- **状态变更**：关键状态变更（如卡密激活、额度变动）通过消息通知

## 8. 性能优化

### 8.1 代码层面

- **组件懒加载**：路由组件使用动态导入，按需加载
- **虚拟列表**：大数据列表使用虚拟滚动技术
- **图片优化**：使用WebP格式，实现懒加载和预加载
- **代码分割**：按路由和功能模块分割代码，减小初始加载体积

### 8.2 构建优化

- **Tree Shaking**：移除未使用的代码
- **压缩资源**：压缩JS、CSS和图片
- **CDN加速**：静态资源使用CDN分发
- **缓存策略**：合理设置缓存策略，减少重复请求

### 8.3 用户体验优化

- **预加载**：预测用户行为，提前加载可能需要的资源
- **骨架屏**：数据加载前显示页面结构，减少白屏时间
- **离线缓存**：关键数据本地缓存，支持离线访问部分功能
- **渐进式加载**：先加载核心内容，再加载次要内容

## 9. 安全措施

### 9.1 前端安全

- **XSS防护**：输入输出过滤，使用安全的模板引擎
- **CSRF防护**：请求携带token，验证请求来源
- **敏感信息保护**：不在前端存储敏感信息，必要时加密存储
- **权限控制**：基于角色的访问控制，隐藏无权限的功能入口

### 9.2 API安全

- **请求签名**：API请求携带签名，防止篡改
- **请求限流**：限制单位时间内的请求次数，防止恶意请求
- **数据加密**：敏感数据传输加密，使用HTTPS协议

## 10. 兼容性

### 10.1 浏览器兼容性

支持以下主流浏览器的最新两个版本：
- Chrome
- Firefox
- Safari
- Edge

### 10.2 设备兼容性

- **桌面设备**：Windows、macOS、Linux
- **移动设备**：iOS 11+、Android 7.0+
- **平板设备**：iPad OS 13+、Android 7.0+

## 11. 前端部署

### 11.1 构建流程

```bash
# 安装依赖
npm install

# 开发环境构建
npm run dev

# 生产环境构建
npm run build

# 代码检查
npm run lint
```

### 11.2 部署策略

- **静态资源服务器**：Nginx或CDN
- **容器化部署**：Docker镜像构建和部署
- **CI/CD**：自动化测试和部署流程

### 11.3 环境配置

```javascript
// .env.development
VITE_API_BASE_URL=http://localhost:3000/api
VITE_APP_ENV=development

// .env.production
VITE_API_BASE_URL=/api
VITE_APP_ENV=production
```

## 12. 前端监控

### 12.1 性能监控

- **加载性能**：页面加载时间、资源加载时间
- **运行性能**：内存使用、CPU使用、帧率
- **网络性能**：API请求时间、成功率、错误率

### 12.2 错误监控

- **JS错误**：捕获并上报JS运行时错误
- **API错误**：捕获并上报API请求错误
- **资源加载错误**：捕获并上报资源加载失败

### 12.3 用户行为监控

- **PV/UV**：页面访问量和独立访问用户
- **停留时间**：用户在各页面的停留时间
- **点击行为**：用户点击和操作行为
- **转化路径**：用户从登录到关键操作的路径