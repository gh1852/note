### Java 8 + Spring：相同逻辑枚举优化（共享 Handler，避免重复）

### 文档概述
本文档介绍在Java 8 + Spring环境中，当多个枚举值共享相同处理逻辑时（如PENDING和DRAFT都校验库存），如何避免重复创建@Component Bean的优化方案。核心思路是通过Spring注入所有Handler，然后使用ApplicationRunner构建共享的Map<Enum, Handler>，实现同一个Handler实例服务多个枚举值，达到O(1)时间复杂度的分发和高效的内存利用。文档详细对比了5种方案，重点推荐ApplicationRunner共享Map和Enum Group两种实现方式，并提供了完整的代码示例、性能对比和最佳实践。

### 使用场景
- **订单状态处理**：PENDING和DRAFT都需要校验库存，SHIPPED需要更新物流信息
- **支付状态流转**：INIT和PENDING都需要风控检查，SUCCESS需要发送通知
- **审批流程**：DRAFT和REVIEW都需要权限校验，APPROVED需要执行业务逻辑
- **工作流引擎**：多个中间状态共享相同的前置条件检查逻辑
- **状态机实现**：当状态转换规则中存在多个状态使用相同处理器时

### Java 8 + Spring：相同逻辑枚举优化（共享 Handler，避免重复）

当**多个枚举值共享相同处理逻辑**（如 `PENDING` 和 `DRAFT` 都校验库存），**避免重复 @Component Bean**（代码冗余、维护差）。**核心优化：Spring 注入所有 Handler → ApplicationRunner 构建共享 Map<Enum, Handler>**，同一个 Handler 实例服务多枚举（O(1) 分发、内存高效）。

- **优势**：
 - **零重复**：共享逻辑一处定义。
 - **OCP**：新枚举复用现有 Handler，或加新 `@Component`。
 - **Spring 友好**：自动注入 + 手动分发表（启动时 O(n) 构建，运行 O(1)）。
 - **类型安全**：Handler 接口统一，运行时验证覆盖率。

#### 1. **方案对比**
| 方案 | 简洁度 | 共享支持 | 注入支持 | 扩展性 | 内存/性能 | 适用场景 |
|-----------------------------|--------|----------|----------|--------|-----------|---------------------------|
| **重复 @Component** | 低 | ❌ | ✅ | 低 | 高 | 不推荐 |
| **Handler 带 Enum param** | 中 | ✅ | ✅ | 中 | 中 | 逻辑微差（不推荐耦合） |
| **共享 Map 构建 (Runner)** | **高**| **✅** | **✅** | **高**| **低** | **推荐1：多对多** |
| **Enum Group 字段** | 高 | ✅ | ✅ | 高 | 低 | **推荐2：分组固定** |
| **@Conditional 多 Qualifier**| 中 | ✅ | ✅ | 中 | 中 | Bean 少 |

#### 2. **推荐实现：ApplicationRunner 共享 Map**
1. **定义 Handler**（如前，不变）。
2. **共享 Handler**：一处 `@Component`，覆盖多枚举。
3. **Initializer**：注入所有 Handler，构建 `EnumMap<OrderStatus, StatusHandler>`（共享赋值）。

##### **步骤1：Handler（共享示例）**
```java
// 共享校验库存逻辑（PENDING + DRAFT）
@Component("stockValidator")  // BeanName 标识共享
public class StockValidatorHandler implements StatusHandler {
    
    private final StockRepository stockRepo;
    
    public StockValidatorHandler(StockRepository stockRepo) {
        this.stockRepo = stockRepo;
    }
    
    @Override
    public void process(Order order) {
        int available = stockRepo.getStock(order.getProductId());
        if (available < order.getQuantity()) {
            throw new RuntimeException("库存不足");
        }
        System.out.println("StockValidator: 校验通过");
    }
}

// 独享逻辑（SHIPPED）
@Component("logisticsUpdater")
public class LogisticsUpdaterHandler implements StatusHandler {
    // 如前...
}
```

##### **步骤2：扩展枚举（新增 DRAFT 共享）**
```java
public enum OrderStatus {
    PENDING,      // 共享 stockValidator
    DRAFT,        // ← 新增，共享相同逻辑
    SHIPPED,      // 独享 logisticsUpdater
    DELIVERED;
}
```

##### **步骤3：共享 Map 构建（ApplicationRunner）**
```java
import java.util.EnumMap;
import java.util.Map;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public class HandlerRegistry implements ApplicationRunner {
    
    private final Map<String, StatusHandler> allHandlers;  // Spring 注入所有 @Component("...")
    private final OrderStatusProcessor processor;         // 注入 Processor，暴露共享Map
    
    public HandlerRegistry(
            Map<String, StatusHandler> allHandlers,     // key=BeanName
            OrderStatusProcessor processor) {
        this.allHandlers = allHandlers;
        this.processor = processor;
    }
    
    @Override
    public void run(ApplicationArguments args) {
        // 构建共享 EnumMap：O(n)，启动一次
        EnumMap<OrderStatus, StatusHandler> dispatchMap = new EnumMap<>(OrderStatus.class);
        
        // 共享规则：配置化（易改）
        Map<OrderStatus, String> mappingRules = Map.of(
            OrderStatus.PENDING, "stockValidator",
            OrderStatus.DRAFT,   "stockValidator",   // ← 共享！
            OrderStatus.SHIPPED, "logisticsUpdater",
            OrderStatus.DELIVERED, "notificationSender"
        );
        
        for (Map.Entry<OrderStatus, String> rule : mappingRules.entrySet()) {
            StatusHandler handler = allHandlers.get(rule.getValue());
            if (handler == null) {
                throw new IllegalStateException("Missing handler: " + rule.getValue());
            }
            dispatchMap.put(rule.getKey(), handler);  // 同一实例多用
        }
        
        // 暴露给 Processor（字段注入或 setter）
        processor.setHandlers(dispatchMap);
        
        System.out.println("Handler registry built: " + dispatchMap.size() + "/" + OrderStatus.values().length);
    }
}
```

##### **步骤4：Processor 更新（支持共享 Map）**
```java
@Service
public class OrderStatusProcessor {
    
    private volatile Map<OrderStatus, StatusHandler> handlers;  // volatile 安全发布
    
    // 构造时空（Runner 后 set）
    public OrderStatusProcessor() {}
    
    public void setHandlers(Map<OrderStatus, StatusHandler> handlers) {
        this.handlers = handlers;
    }
    
    public void process(OrderStatus status, Order order) {
        StatusHandler handler = handlers.get(status);
        if (handler == null) throw new IllegalArgumentException("No handler: " + status);
        handler.process(order);
    }
}
```

##### **步骤5：使用（不变）**
```java
// Controller 中
processor.process(OrderStatus.DRAFT, order);  // 用 stockValidator（共享）！
processor.process(OrderStatus.PENDING, order); // 同上
```

#### 3. **备选：Enum Group（静态分组，超简洁）**
枚举内置 `group` 字段，Map 按组分发（零运行时开销）。

```java
public enum OrderStatus {
    PENDING("stock"), DRAFT("stock"),    // 共享 group
    SHIPPED("logistics"), DELIVERED("notify");

    private final String group;
    OrderStatus(String group) { this.group = group; }
    public String getGroup() { return group; }
}

// Processor
@Service
public class OrderStatusProcessor {
    private final Map<String, StatusHandler> groupHandlers;  // @Autowired Map<String, ...>
    
    public void process(OrderStatus status, Order order) {
        String group = status.getGroup();
        groupHandlers.get(group).process(order);  // O(1)，共享！
    }
}
```
- **BeanName=group**：`@Component("stock") StockValidatorHandler`。

#### 4. **选择 Checklist**
- ✅ **Runner 共享 Map**：动态规则、多对多（默认）。
- ✅ **Enum Group**：逻辑严格分组、配置少。
- ✅ **性能**：EnumMap > HashMap（序数优化）；启动验证 100% 覆盖。
- ⚠️ **测试**：
```java
@SpringBootTest
class Test {
 @Autowired HandlerRegistry registry;
 @Test void sharedLogic() {
 // Mock allHandlers，验证 dispatchMap.get(DRAFT) == get(PENDING)
 }
}
```
- ⚠️ **配置化**：mappingRules 移 `@ConfigurationProperties`，热加载。
- **扩展**：新共享组 → 加 enum + rules，无改 Handler。

#### 5. **最佳实践**
- **规则源**：`@ConfigurationProperties(prefix="handlers")` YAML 配置。
- **监控**：Actuator + Custom Metrics（handler 调用计数）。
- **参考**：Spring `HandlerMethodMapping` 共享适配器。
- **Lombok**：`@AllArgsConstructor` + `@Value` 简化。

提供你的共享规则/枚举细节（如哪些共享），我给出精确 YAML+代码！

---

[← 返回文档首页](../index.md)
