简化版电商秒杀系统架构设计

📋 项目亮点（适合简历描述）

• 实现高并发秒杀场景，模拟QPS 1000+的抢购请求

• 采用Redis缓存预减库存，避免数据库直接穿透

• 使用消息队列RabbitMQ异步处理订单，提升系统吞吐量

• 接口层添加令牌桶限流与验证码防刷机制

• 通过JMeter压力测试验证系统性能，完成性能调优

🏗️ 技术栈建议

层级 技术选型 说明

后端框架 Spring Boot 3.x 快速构建REST API

数据层 MyBatis-Plus 简化数据库操作

缓存 Redis 7.x 库存预热、分布式锁

消息队列 RabbitMQ 异步削峰、解耦

数据库 MySQL 8.x 主从分离（可选）

测试工具 JMeter 5.x 压力测试

部署 Docker + Nginx 容器化部署

🧩 核心模块设计


1. 用户服务
   - 注册/登录（JWT鉴权）
   
2. 商品服务
   - 商品列表/详情
   - 秒杀商品预热到Redis
   
3. 秒杀服务（核心）
   - 获取秒杀路径（隐藏接口地址）
   - Redis预减库存 + Lua脚本保证原子性
   - 生成订单ID → 发送MQ消息
   - 查询秒杀结果
   
4. 订单服务
   - 消费MQ消息，生成完整订单
   - 订单状态查询


⚡ 关键技术实现点

1. 防超卖方案

// Redis Lua脚本保证原子性
String script = "if redis.call('get', KEYS[1]) >= ARGV[1] then " +
                "    return redis.call('decrby', KEYS[1], ARGV[1]) " +
                "else " +
                "    return -1 " +
                "end";


2. 接口限流

• 使用Guava RateLimiter做令牌桶限流

• 对同一用户ID做访问频率限制

3. 异步下单流程


用户请求 → 校验 → 预减库存 → 生成订单号 → 发送MQ → 返回"处理中"
                              ↓
                          消费者处理 → 写数据库 → 更新缓存


📊 性能优化点

1. CDN缓存静态页面
2. Redis集群分担热点key压力
3. 库存分段：将商品库存拆分到多个Redis key
4. 数据库优化：索引、分库分表（商品/订单分表）

📁 项目结构示例


miaosha/
├── src/main/java/com/example/miaosha/
│   ├── controller/     # 接口层
│   ├── service/       # 业务逻辑
│   ├── mapper/        # 数据访问
│   ├── entity/        # 实体类
│   ├── config/        # 配置类
│   ├── mq/            # 消息队列
│   └── utils/         # 工具类
├── sql/               # 数据库脚本
├── jmeter/            # 压测脚本
└── README.md          # 项目说明


🧪 学习价值

1. 并发编程：线程安全、分布式锁
2. 系统设计：高可用、可扩展架构
3. 问题排查：JMeter压测 → 定位瓶颈 → 优化
4. 简历亮点：完整项目经历，涵盖主流中间件

🚀 进阶扩展（加分项）

• 实现分布式唯一ID生成（雪花算法）

• 添加Sentinel流控与降级规则

• 使用Seata实现分布式事务

• 接入Prometheus + Grafana监控

这个项目能全面展示你对高并发场景的理解，建议在GitHub上完整实现并写好文档，面试时会有很大优势。
