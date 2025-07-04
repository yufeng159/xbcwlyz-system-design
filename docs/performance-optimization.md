# 性能优化设计

## 1. 性能优化概述

### 1.1 性能优化目标

小白菜网络验证卡密系统的性能优化目标如下：

- **响应时间**：核心API接口响应时间<200ms，非核心接口<500ms
- **并发能力**：系统支持至少5000并发用户
- **吞吐量**：系统每秒处理交易请求>1000TPS
- **可用性**：系统可用性达到99.9%以上
- **资源利用率**：CPU平均利用率<70%，内存使用率<80%

### 1.2 性能挑战分析

系统面临的主要性能挑战：

1. **高并发验证**：卡密验证是高频操作，需要支持大量并发请求
2. **数据量大**：卡密、交易记录等数据量大且增长快
3. **复杂查询**：统计分析涉及复杂查询和计算
4. **多级代理**：多级代理结构增加了数据查询复杂度
5. **安全校验**：安全验证增加了请求处理时间

## 2. 架构层面优化

### 2.1 分层架构优化

- **微服务拆分**：按业务领域合理拆分微服务，避免服务过大或过小
- **无状态设计**：所有服务设计为无状态，便于水平扩展
- **异步处理**：非核心流程采用异步处理，提高响应速度
- **边缘计算**：将部分计算下沉到边缘节点，减轻中心服务负担

### 2.2 负载均衡优化

- **多级负载均衡**：CDN -> 网关 -> 服务集群的多级负载均衡
- **动态权重**：根据服务器负载动态调整权重
- **会话亲和性**：相关请求路由到同一服务实例，减少跨实例调用
- **预热机制**：新服务实例预热后再加入负载均衡池

### 2.3 服务治理优化

- **服务熔断**：及时熔断故障服务，避免级联故障
- **服务降级**：高负载时自动降级非核心功能
- **限流保护**：多级限流策略保护系统
- **弹性伸缩**：根据负载自动扩缩服务实例

## 3. 数据库优化

### 3.1 数据库架构优化

- **读写分离**：主库负责写操作，从库负责读操作
- **分库分表**：按业务和数据量水平拆分数据库
- **连接池优化**：合理配置连接池参数，避免连接资源不足
- **数据库集群**：关键业务数据库采用集群部署

```sql
-- 读写分离配置示例（基于MyCat）
CREATE TABLE t_card (
    id VARCHAR(32) NOT NULL,
    card_no VARCHAR(64) NOT NULL,
    project_id VARCHAR(32) NOT NULL,
    batch_id VARCHAR(32) NOT NULL,
    created_by VARCHAR(32) NOT NULL,
    status TINYINT NOT NULL,
    duration INT NOT NULL,
    device_id VARCHAR(64),
    device_info TEXT,
    remark VARCHAR(255),
    created_at TIMESTAMP NOT NULL,
    activated_at TIMESTAMP,
    expire_at TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_card_no (card_no),
    KEY idx_project_id (project_id),
    KEY idx_batch_id (batch_id),
    KEY idx_status (status),
    KEY idx_device_id (device_id),
    KEY idx_created_at (created_at),
    KEY idx_expire_at (expire_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
dbpartition by hash(id) dbpartitions 4 tbpartition by hash(id) tbpartitions 8;
```

### 3.2 SQL优化

- **索引优化**：合理设计索引，避免索引失效
- **查询优化**：优化SQL语句，避免全表扫描和笛卡尔积
- **分页优化**：使用游标分页代替传统分页
- **批量操作**：使用批量插入和更新代替单条操作

```sql
-- 优化前
SELECT * FROM t_card WHERE project_id = ? AND status = ? ORDER BY created_at DESC LIMIT ?, ?;

-- 优化后
SELECT * FROM t_card WHERE project_id = ? AND status = ? AND created_at < ? ORDER BY created_at DESC LIMIT ?;
```

### 3.3 数据访问层优化

- **ORM优化**：合理使用ORM，避免N+1查询问题
- **批量查询**：使用IN语句代替多次单条查询
- **延迟加载**：按需加载关联数据
- **结果集映射**：只映射需要的字段，减少数据传输量

```java
// 优化前
public List<CardDTO> getCardsByProjectId(String projectId) {
    List<Card> cards = cardRepository.findByProjectId(projectId);
    return cards.stream().map(this::convertToDTO).collect(Collectors.toList());
}

// 优化后
public List<CardDTO> getCardsByProjectId(String projectId) {
    return cardRepository.findCardDTOsByProjectId(projectId);
}

@Query("SELECT new com.xbc.card.dto.CardDTO(c.id, c.cardNo, c.status, c.createdAt, c.expireAt) " +
       "FROM Card c WHERE c.projectId = :projectId")
List<CardDTO> findCardDTOsByProjectId(@Param("projectId") String projectId);
```

## 4. 缓存优化

### 4.1 多级缓存策略

- **本地缓存**：使用Caffeine缓存热点数据
- **分布式缓存**：使用Redis缓存共享数据
- **二级缓存**：本地缓存+分布式缓存的二级缓存架构
- **全页缓存**：对静态内容进行全页缓存

```java
@Configuration
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        List<Cache> caches = new ArrayList<>();
        
        // 本地缓存配置
        caches.add(new CaffeineCache("localCardCache", 
            Caffeine.newBuilder()
                .expireAfterWrite(5, TimeUnit.MINUTES)
                .maximumSize(10000)
                .build()));
        
        // Redis缓存配置
        caches.add(new RedisCache("redisCardCache", 
            redisTemplate, 
            Duration.ofMinutes(30)));
        
        cacheManager.setCaches(caches);
        return cacheManager;
    }
}
```

### 4.2 缓存策略优化

- **热点数据缓存**：识别并缓存热点数据
- **预加载缓存**：系统启动时预加载常用数据
- **缓存更新策略**：采用更新数据库+删除缓存的策略
- **缓存穿透防护**：使用布隆过滤器过滤无效请求

```java
@Service
public class CardService {
    
    private BloomFilter<String> cardNoFilter;
    
    @PostConstruct
    public void initBloomFilter() {
        cardNoFilter = BloomFilter.create(
            Funnels.stringFunnel(Charset.defaultCharset()),
            10000000,  // 预计元素数量
            0.001);    // 误判率
        
        // 加载所有卡号到布隆过滤器
        List<String> allCardNos = cardRepository.findAllCardNos();
        for (String cardNo : allCardNos) {
            cardNoFilter.put(cardNo);
        }
    }
    
    public CardDTO getCardByCardNo(String cardNo) {
        // 布隆过滤器判断卡号是否存在
        if (!cardNoFilter.mightContain(cardNo)) {
            return null;
        }
        
        // 查询缓存
        CardDTO cardDTO = cardCache.get(cardNo);
        if (cardDTO != null) {
            return cardDTO;
        }
        
        // 查询数据库
        Card card = cardRepository.findByCardNo(cardNo);
        if (card == null) {
            return null;
        }
        
        // 更新缓存
        cardDTO = convertToDTO(card);
        cardCache.put(cardNo, cardDTO);
        
        return cardDTO;
    }
}
```

### 4.3 缓存监控与优化

- **缓存命中率监控**：实时监控缓存命中率
- **缓存容量优化**：根据数据访问模式优化缓存容量
- **缓存过期策略**：根据数据更新频率设置过期时间
- **缓存预热**：系统启动时预热缓存

## 5. 接口优化

### 5.1 接口设计优化

- **接口合并**：合并频繁一起调用的接口
- **数据精简**：只返回必要的数据字段
- **分页优化**：支持游标分页和按需加载
- **批量接口**：提供批量操作接口

```java
// 优化前：单个查询接口
@GetMapping("/cards/{cardId}")
public ResponseEntity<CardDTO> getCard(@PathVariable String cardId) {
    CardDTO card = cardService.getCardById(cardId);
    return ResponseEntity.ok(card);
}

// 优化后：批量查询接口
@PostMapping("/cards/batch")
public ResponseEntity<List<CardDTO>> getCards(@RequestBody List<String> cardIds) {
    List<CardDTO> cards = cardService.getCardsByIds(cardIds);
    return ResponseEntity.ok(cards);
}
```

### 5.2 接口压缩与序列化

- **响应压缩**：启用GZIP压缩减少传输数据量
- **序列化优化**：使用高效的序列化方式（如Protocol Buffers）
- **增量更新**：只传输变更的数据
- **二进制传输**：关键接口使用二进制协议

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // 添加Protocol Buffers转换器
        converters.add(new ProtobufHttpMessageConverter());
        
        // 优化JSON转换器
        MappingJackson2HttpMessageConverter jsonConverter = new MappingJackson2HttpMessageConverter();
        ObjectMapper objectMapper = jsonConverter.getObjectMapper();
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        converters.add(jsonConverter);
    }
    
    @Bean
    public Filter compressionFilter() {
        CompressingFilter compressingFilter = new CompressingFilter();
        return compressingFilter;
    }
}
```

### 5.3 异步接口

- **异步处理**：耗时操作异步处理
- **WebSocket**：实时通知使用WebSocket
- **服务器推送**：使用Server-Sent Events推送更新
- **长轮询**：替代传统轮询减少请求次数

```java
@RestController
@RequestMapping("/api/v1/cards")
public class CardController {
    
    @Async
    @PostMapping("/generate")
    public CompletableFuture<ResponseEntity<BatchGenerateResult>> generateCardsAsync(
            @RequestBody CardGenerateRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            BatchGenerateResult result = cardService.generateCards(request);
            return ResponseEntity.ok(result);
        });
    }
    
    @GetMapping("/events")
    public SseEmitter subscribeToCardEvents() {
        SseEmitter emitter = new SseEmitter(Long.MAX_VALUE);
        cardEventService.addEmitter(emitter);
        emitter.onCompletion(() -> cardEventService.removeEmitter(emitter));
        emitter.onTimeout(() -> cardEventService.removeEmitter(emitter));
        return emitter;
    }
}
```

## 6. 前端优化

### 6.1 资源优化

- **资源压缩**：压缩JS、CSS和图片
- **资源合并**：合并小文件减少请求数
- **懒加载**：按需加载组件和资源
- **CDN加速**：使用CDN分发静态资源

```javascript
// vue.config.js
module.exports = {
  productionSourceMap: false,
  chainWebpack: config => {
    config.optimization.splitChunks({
      chunks: 'all',
      cacheGroups: {
        vendors: {
          name: 'chunk-vendors',
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          chunks: 'initial'
        },
        common: {
          name: 'chunk-common',
          minChunks: 2,
          priority: -20,
          chunks: 'initial',
          reuseExistingChunk: true
        }
      }
    });
    
    if (process.env.NODE_ENV === 'production') {
      config.plugin('compression').use(require('compression-webpack-plugin'), [{
        test: /\.js$|\.html$|\.css$|\.svg$/,
        threshold: 10240,
        deleteOriginalAssets: false
      }]);
    }
  },
  configureWebpack: {
    externals: {
      'vue': 'Vue',
      'element-plus': 'ElementPlus',
      'echarts': 'echarts'
    }
  }
};
```

### 6.2 渲染优化

- **虚拟列表**：长列表使用虚拟滚动
- **懒渲染**：分批渲染大量DOM
- **骨架屏**：提升加载体验
- **预渲染**：预渲染静态内容

```vue
<template>
  <div class="card-list">
    <el-skeleton :loading="loading" animated>
      <template #template>
        <div v-for="i in 10" :key="i" class="skeleton-item">
          <el-skeleton-item variant="p" style="width: 100%" />
        </div>
      </template>
      
      <template #default>
        <virtual-list
          style="height: 500px; overflow-y: auto;"
          :data-key="'id'"
          :data-sources="cards"
          :data-component="cardComponent"
          :estimate-size="60"
        />
      </template>
    </el-skeleton>
  </div>
</template>

<script setup>
import { ref, defineComponent } from 'vue';
import { VirtualList } from 'vue-virtual-scroll-list';

const loading = ref(true);
const cards = ref([]);

const cardComponent = defineComponent({
  props: ['source'],
  template: `
    <div class="card-item">
      <div class="card-no">{{ source.cardNo }}</div>
      <div class="card-status">{{ formatStatus(source.status) }}</div>
      <div class="card-date">{{ formatDate(source.createdAt) }}</div>
    </div>
  `,
  methods: {
    formatStatus(status) {
      // 格式化状态
    },
    formatDate(date) {
      // 格式化日期
    }
  }
});

// 分批加载数据
const loadCards = async (page = 1, pageSize = 100) => {
  try {
    const response = await fetch(`/api/v1/cards?page=${page}&size=${pageSize}`);
    const data = await response.json();
    
    if (page === 1) {
      cards.value = data.content;
    } else {
      cards.value = [...cards.value, ...data.content];
    }
    
    loading.value = false;
    
    if (data.hasNext && page < 5) { // 预加载5页
      setTimeout(() => loadCards(page + 1, pageSize), 100);
    }
  } catch (error) {
    console.error('Failed to load cards:', error);
    loading.value = false;
  }
};

onMounted(() => {
  loadCards();
});
</script>
```

### 6.3 网络优化

- **请求合并**：合并多个API请求
- **请求缓存**：缓存请求结果
- **预加载**：预加载可能需要的资源
- **断点续传**：大文件支持断点续传

```javascript
// api.js
import axios from 'axios';
import { useCache } from './cache';

const cache = useCache();

// 创建axios实例
const api = axios.create({
  baseURL: '/api/v1',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// 请求拦截器
api.interceptors.request.use(
  config => {
    // 添加token
    const token = localStorage.getItem('token');
    if (token) {
      config.headers['Authorization'] = `Bearer ${token}`;
    }
    
    // 缓存GET请求
    if (config.method === 'get' && config.cache !== false) {
      const cacheKey = `${config.url}${JSON.stringify(config.params || {})}`;
      const cachedData = cache.get(cacheKey);
      
      if (cachedData) {
        // 返回缓存数据
        return {
          ...config,
          adapter: () => Promise.resolve({
            data: cachedData,
            status: 200,
            statusText: 'OK',
            headers: {},
            config,
            request: {}
          }),
          __fromCache: true
        };
      }
      
      // 标记需要缓存
      config.__cacheKey = cacheKey;
    }
    
    return config;
  },
  error => Promise.reject(error)
);

// 响应拦截器
api.interceptors.response.use(
  response => {
    // 缓存响应数据
    if (response.config.__cacheKey && !response.config.__fromCache) {
      const cacheTime = response.config.cacheTime || 5 * 60 * 1000; // 默认缓存5分钟
      cache.set(response.config.__cacheKey, response.data, cacheTime);
    }
    
    return response;
  },
  error => Promise.reject(error)
);

// 批量请求
export const batchRequest = requests => {
  return axios.all(requests.map(req => api(req)))
    .then(axios.spread((...responses) => responses.map(res => res.data)));
};

export default api;
```

## 7. JVM优化

### 7.1 JVM参数优化

- **内存配置**：合理配置堆内存和非堆内存
- **垃圾回收器**：选择合适的垃圾回收器
- **GC参数**：优化GC参数减少停顿时间
- **线程池配置**：合理配置线程池大小

```bash
# JVM优化参数示例
JAVA_OPTS="-server \
  -Xms4g -Xmx4g \
  -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=100 \
  -XX:+ParallelRefProcEnabled \
  -XX:ErrorFile=/var/log/java_error.log \
  -XX:HeapDumpPath=/var/log/java_heapdump.hprof \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:+DisableExplicitGC \
  -XX:+PrintGCDetails \
  -XX:+PrintGCDateStamps \
  -Xloggc:/var/log/gc.log \
  -XX:+UseGCLogFileRotation \
  -XX:NumberOfGCLogFiles=10 \
  -XX:GCLogFileSize=100m"
```

### 7.2 代码层优化

- **对象池化**：复用对象减少GC压力
- **内存泄漏检测**：定期检测内存泄漏
- **锁优化**：减少锁竞争和锁粒度
- **异常处理优化**：避免频繁创建异常对象

```java
// 对象池化示例
public class CardVerifier {
    
    private static final GenericObjectPool<VerificationContext> contextPool = 
        new GenericObjectPool<>(new VerificationContextFactory());
    
    public VerificationResult verify(CardVerificationRequest request) {
        VerificationContext context = null;
        try {
            // 从对象池获取上下文对象
            context = contextPool.borrowObject();
            context.setRequest(request);
            
            // 执行验证逻辑
            VerificationResult result = doVerify(context);
            
            return result;
        } catch (Exception e) {
            log.error("Verification failed", e);
            return VerificationResult.failed("系统错误");
        } finally {
            if (context != null) {
                try {
                    // 重置上下文并归还到对象池
                    context.reset();
                    contextPool.returnObject(context);
                } catch (Exception e) {
                    log.warn("Failed to return context to pool", e);
                }
            }
        }
    }
    
    private VerificationResult doVerify(VerificationContext context) {
        // 验证逻辑
    }
    
    // 上下文工厂
    private static class VerificationContextFactory extends BasePooledObjectFactory<VerificationContext> {
        @Override
        public VerificationContext create() {
            return new VerificationContext();
        }
        
        @Override
        public PooledObject<VerificationContext> wrap(VerificationContext context) {
            return new DefaultPooledObject<>(context);
        }
    }
}
```

### 7.3 线程优化

- **线程池隔离**：不同业务使用不同线程池
- **任务优先级**：支持任务优先级调度
- **线程监控**：监控线程状态和线程泄漏
- **异步处理**：耗时操作异步处理

```java
@Configuration
public class ThreadPoolConfig {
    
    @Bean("cardVerificationExecutor")
    public ThreadPoolTaskExecutor cardVerificationExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(20);
        executor.setMaxPoolSize(100);
        executor.setQueueCapacity(1000);
        executor.setThreadNamePrefix("card-verify-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
    
    @Bean("cardGenerationExecutor")
    public ThreadPoolTaskExecutor cardGenerationExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("card-gen-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
        executor.initialize();
        return executor;
    }
    
    @Bean("statisticsExecutor")
    public ThreadPoolTaskExecutor statisticsExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("stats-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardOldestPolicy());
        executor.initialize();
        return executor;
    }
}
```

## 8. 网络优化

### 8.1 网络架构优化

- **多CDN部署**：使用多个CDN提供商
- **就近接入**：用户就近接入节点
- **专线接入**：核心业务使用专线接入
- **多链路冗余**：多条网络链路保障可用性

### 8.2 协议优化

- **HTTP/2**：启用HTTP/2减少连接数
- **TLS优化**：优化TLS配置提高安全性和性能
- **长连接**：使用长连接减少连接建立开销
- **连接复用**：复用已建立的连接

```nginx
# Nginx HTTP/2和TLS优化配置
server {
    listen 443 ssl http2;
    server_name api.example.com;
    
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    
    # TLS优化
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    
    # 启用HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    
    # 连接优化
    keepalive_timeout 65;
    keepalive_requests 1000;
    
    # 压缩配置
    gzip on;
    gzip_comp_level 6;
    gzip_min_length 1000;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # 缓存控制
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 30d;
        add_header Cache-Control "public, max-age=2592000";
    }
    
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 8.3 网络安全优化

- **DDoS防护**：部署DDoS防护措施
- **WAF**：部署Web应用防火墙
- **API网关限流**：API网关层实现限流
- **异常流量检测**：实时检测异常流量

## 9. 监控与调优

### 9.1 全链路监控

- **APM工具**：使用SkyWalking等APM工具
- **调用链跟踪**：跟踪请求全链路
- **性能指标**：监控关键性能指标
- **异常监控**：实时监控系统异常

```java
@Configuration
public class TracingConfig {
    
    @Bean
    public Filter tracingFilter() {
        return new OncePerRequestFilter() {
            @Override
            protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) 
                    throws ServletException, IOException {
                
                String traceId = request.getHeader("X-Trace-ID");
                if (traceId == null) {
                    traceId = UUID.randomUUID().toString();
                }
                
                MDC.put("traceId", traceId);
                response.setHeader("X-Trace-ID", traceId);
                
                try {
                    long startTime = System.currentTimeMillis();
                    filterChain.doFilter(request, response);
                    long duration = System.currentTimeMillis() - startTime;
                    
                    // 记录请求处理时间
                    if (duration > 500) { // 记录慢请求
                        log.warn("Slow request: {} {} took {}ms", request.getMethod(), request.getRequestURI(), duration);
                    }
                } finally {
                    MDC.remove("traceId");
                }
            }
        };
    }
}
```

### 9.2 性能测试

- **负载测试**：模拟正常负载场景
- **压力测试**：模拟高负载场景
- **稳定性测试**：长时间运行测试
- **容量规划**：基于测试结果进行容量规划

### 9.3 自动化调优

- **自适应参数**：系统自动调整参数
- **智能限流**：基于系统负载智能限流
- **资源弹性伸缩**：自动扩缩资源
- **热点识别**：自动识别并优化热点数据

```java
@Service
public class AdaptiveRateLimiter {
    
    private final LoadingCache<String, RateLimiter> rateLimiters;
    private final SystemLoadMonitor loadMonitor;
    
    public AdaptiveRateLimiter(SystemLoadMonitor loadMonitor) {
        this.loadMonitor = loadMonitor;
        
        rateLimiters = CacheBuilder.newBuilder()
            .expireAfterAccess(10, TimeUnit.MINUTES)
            .build(new CacheLoader<String, RateLimiter>() {
                @Override
                public RateLimiter load(String key) {
                    return RateLimiter.create(getInitialRate(key));
                }
            });
    }
    
    public boolean tryAcquire(String key) {
        RateLimiter limiter = rateLimiters.getUnchecked(key);
        
        // 根据系统负载动态调整速率
        double systemLoad = loadMonitor.getCurrentLoad();
        double currentRate = limiter.getRate();
        double targetRate = calculateTargetRate(key, systemLoad);
        
        if (Math.abs(currentRate - targetRate) / currentRate > 0.1) { // 变化超过10%才调整
            limiter.setRate(targetRate);
        }
        
        return limiter.tryAcquire();
    }
    
    private double getInitialRate(String key) {
        // 根据不同的资源类型设置初始速率
        if (key.startsWith("api:verify")) {
            return 1000.0; // 验证接口每秒1000次
        } else if (key.startsWith("api:generate")) {
            return 50.0; // 生成接口每秒50次
        } else {
            return 100.0; // 默认每秒100次
        }
    }
    
    private double calculateTargetRate(String key, double systemLoad) {
        double baseRate = getInitialRate(key);
        
        // 根据系统负载调整速率
        if (systemLoad > 0.8) { // 高负载
            return baseRate * 0.5; // 降低到50%
        } else if (systemLoad > 0.6) { // 中等负载
            return baseRate * 0.8; // 降低到80%
        } else { // 低负载
            return baseRate; // 保持原速率
        }
    }
}
```

## 10. 性能优化最佳实践

### 10.1 开发阶段

- **代码审查**：关注性能相关的代码审查
- **单元测试**：编写性能单元测试
- **性能基准**：建立性能基准和目标
- **持续集成**：集成性能测试到CI/CD流程

### 10.2 测试阶段

- **性能测试**：定期进行全面性能测试
- **瓶颈分析**：识别和解决性能瓶颈
- **回归测试**：确保优化不引入新问题
- **模拟生产**：在类生产环境进行测试

### 10.3 运维阶段

- **性能监控**：实时监控系统性能
- **容量规划**：定期进行容量规划
- **故障演练**：进行性能故障演练
- **持续优化**：根据实际运行情况持续优化

## 11. 性能优化路线图

### 11.1 短期优化计划（1-3个月）

1. **基础优化**：JVM参数调优、数据库索引优化
2. **缓存引入**：引入多级缓存系统
3. **代码优化**：优化关键业务代码
4. **监控建设**：建立基础监控系统

### 11.2 中期优化计划（3-6个月）

1. **架构优化**：引入读写分离、分库分表
2. **缓存深化**：完善缓存策略和预热机制
3. **异步改造**：非核心流程异步化
4. **全链路监控**：建立全链路监控系统

### 11.3 长期优化计划（6-12个月）

1. **微服务优化**：服务拆分和治理优化
2. **自动化调优**：引入自适应参数调整
3. **智能运维**：AI辅助性能分析和优化
4. **全球化部署**：多区域部署和流量调度

## 12. 性能优化成果与效益

### 12.1 预期性能提升

- **响应时间**：核心接口响应时间降低50%
- **并发能力**：系统并发能力提升200%
- **资源利用率**：资源利用率提高30%
- **系统稳定性**：系统可用性达到99.99%

### 12.2 业务价值

- **用户体验提升**：系统响应更快，用户满意度提高
- **业务容量增长**：支持业务规模扩大
- **运营成本降低**：资源利用率提高，降低硬件成本
- **系统可靠性**：减少故障和宕机时间

### 12.3 持续优化机制

- **性能评审**：定期进行性能评审会议
- **优化迭代**：将性能优化纳入常规迭代
- **知识沉淀**：建立性能优化知识库
- **团队能力**：提升团队性能优化能力