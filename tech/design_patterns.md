# 设计模式详解与实战指南

## 目录
1. 策略模式
2. 责任链模式
3. 命令模式

---

## 策略模式

### 1.1 定义与目的
- **定义**：定义一系列算法，将它们分别封装起来，使它们可以互相替换。策略模式让算法的变化独立于使用算法的客户端。
- **目的**：
  - 避免冗长的 if-else 或 switch-case 分支判断
  - 将做什么（业务逻辑）与怎么做（具体算法）分离
  - 使得新增策略时无需修改 Existing Code（遵循开闭原则）

### 1.2 结构
```
客户端 ──┬── 策略接口（Strategy）
         │       ├── 具体策略A
         │       ├── 具体策略B
         │       └── 具体策略C
         │
         └── 策略上下文（Context）
                    └── 持有/选择策略
```

### 1.3 核心角色
- Strategy（策略）：声明所有支持的算法的公共接口
- ConcreteStrategy（具体策略）：实现 Strategy 接口的具体算法
- Context（上下文）：客户端通过它来访问策略，通常持有 Strategy 引用

### 1.4 代码结构
```java
// 1. 策略接口
public interface Strategy {
    boolean supports(String context);
    String execute(ContextData data);
}

// 2. 具体策略
@Component
public class ConcreteStrategyA implements Strategy {
    @Override
    public boolean supports(String resource) {
        return "A".equals(resource);
    }
    @Override
    public String execute(ContextData data) {
        return "Result A";
    }
}

// 3. 策略注册器/上下文
@Component
public class StrategyRegistry {
    private final List<Strategy> strategies;
    public String handle(String resource, ContextData data) {
        return strategies.stream()
            .filter(s -> s.supports(resource))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("无可用策略"))
            .execute(data);
    }
}
```

### 1.5 适用场景
- 多种算法切换：同一业务有多种实现方式，运行时决定
- 条件分支复杂：if-else 超过 3-5 个分支
- 避免代码重复：多个地方使用相同的条件判断
- 框架扩展点：提供 SPI 机制供第三方扩展

### 1.6 实际案例：日志格式化
**传统写法（不推荐）**：
```java
public String formatLog(String resource, String action, Object[] args) {
    switch (resource) {
        case "user":
            return String.format("用户操作: %s, 参数: %s", action, Arrays.toString(args));
        case "role":
            return String.format("角色操作: %s", action);
        case "permission":
            return String.format("权限操作: %s", action);
        default:
            return "未知操作";
    }
}
```

**策略模式重构后**：
```java
// Strategy 接口
public interface LogStrategy {
    boolean supports(String resource);
    String format(String action, Object[] args);
}

// 具体策略
@Component
public class UserLogStrategy implements LogStrategy {
    @Override
    public boolean supports(String resource) { return "user".equals(resource); }
    @Override
    public String format(String action, Object[] args) {
        return "用户" + getActionName(action) + "，参数：" + Arrays.toString(args);
    }
}

// 使用
@Service
public class LogService {
    private final List<LogStrategy> strategies;
    public String format(String resource, String action, Object[] args) {
        return strategies.stream()
            .filter(s -> s.supports(resource))
            .findFirst()
            .map(s -> s.format(action, args))
            .orElse("未知操作");
    }
}
```

### 1.7 注意事项
- 策略数量控制：超过 10 个策略建议使用工厂模式按类别分组
- 顺序依赖：如果策略有优先级，需实现 Comparable<Strategy> 或 @Order 注解
- 状态共享：策略应是无状态的，如需共享数据使用 Context 传递
- 性能考虑：策略查找可缓存结果（Map<resource, Strategy>），避免每次遍历

---

## 责任链模式

### 2.1 定义与目的
- **定义**：使多个对象都有机会处理请求，从而避免请求的发送者与接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。
- **目的**：
  - 解耦请求发送者与处理器
  - 动态组合处理流程
  - 符合单一职责原则（每个节点只做一件事）

### 2.2 结构
```
客户端 ──┬── 处理器1 ── 处理器2 ── 处理器3 ── ... ── 末端处理器
         │          │          │
         └──────────┴──────────┴── (链式调用)
```

### 2.3 核心角色
- Handler（处理器）：定义处理请求的接口，包含对下一个处理器的引用
- ConcreteHandler（具体处理器）：实现处理逻辑，决定是否传递请求
- Client（客户端）：创建链并提交请求

### 2.4 代码结构
**版本1：显式链（固定顺序）**
```java
// 处理器接口
public interface Handler {
    void handle(Request request);
    void setNext(Handler next);
}

// 具体处理器
public class HandlerA implements Handler {
    private Handler next;
    @Override
    public void handle(Request request) {
        if (shouldHandle(request)) {
            process(request);
        } else if (next != null) {
            next.handle(request);
        }
    }
}

// 客户端组装链
Handler chain = new HandlerA();
chain.setNext(new HandlerB());
chain.setNext(new HandlerC());
chain.handle(request);
```

**版本2：隐式链（自动扫描 + 顺序控制）**
```java
// 处理器接口（简化版）
public interface Validator {
    int order();
    void validate(Request request);
}

// 链管理器
@Component
public class ValidatorChain {
    private final List<Validator> validators;
    public void validate(Request request) {
        validators.stream()
            .sorted(Comparator.comparingInt(Validator::order))
            .forEach(v -> v.validate(request));
    }
}
```

### 2.5 适用场景
- 多层审批：申请按层级逐级审批
- 数据校验：多个校验规则依次执行
- 过滤器链：Tomcat Filter、Spring Interceptor
- 异常处理：逐级捕获、降级处理

### 2.6 实际案例：用户注册校验
**需求**：新用户注册需依次校验
1. 用户名格式（3-20 位字母/数字/下划线）
2. 用户名唯一性
3. 密码强度（含大小写字母+数字，≥8 位）
4. 邮箱格式

**传统写法（不推荐）**：
```java
public void register(UserDTO dto) {
    if (!dto.getUsername().matches("^[a-zA-Z0-9_]{3,20}$")) {
        throw new BizException("用户名格式错误");
    }
    if (userService.exists(dto.getUsername())) {
        throw new BizException("用户名已存在");
    }
    if (!isStrong(dto.getPassword())) {
        throw new BizException("密码强度不足");
    }
    if (!EmailValidator.getInstance().isValid(dto.getEmail())) {
        throw new BizException("邮箱格式错误");
    }
    // 业务逻辑...
}
```

**责任链模式重构后**：
```java
// 1. 校验器接口
public interface UserRegisterValidator {
    int order();
    void validate(UserDTO dto) throws ValidationException;
}

// 2. 实现类
@Component
public class UsernameFormatValidator implements UserRegisterValidator {
    @Override public int order() { return 1; }
    @Override
    public void validate(UserDTO dto) {
        if (!dto.getUsername().matches("^[a-zA-Z0-9_]{3,20}$")) {
            throw new ValidationException("用户名格式错误");
        }
    }
}

@Component
public class UsernameUniqueValidator implements UserRegisterValidator {
    @Override public int order() { return 2; }
    @Override
    public void validate(UserDTO dto) {
        if (userService.exists(dto.getUsername())) {
            throw new ValidationException("用户名已存在");
        }
    }
}

// 3. 链管理器（自动组装）
@Component
public class ValidatorChain {
    private final List<UserRegisterValidator> validators;
    public void validate(UserDTO dto) {
        validators.stream()
            .sorted(Comparator.comparingInt(UserRegisterValidator::order))
            .forEach(v -> v.validate(dto));
    }
}

// 4. 使用
@Service
public class UserService {
    private final ValidatorChain validatorChain;
    public void register(UserDTO dto) {
        validatorChain.validate(dto);  // 自动按顺序执行
        // 业务逻辑...
    }
}
```

### 2.7 注意事项
- 链条断裂：某个节点不调用 next.handle() 会终止链条，这是设计预期
- 异常处理：统一在链尾或外层捕获，不建议在中间节点吞掉异常
- 顺序敏感：order() 值必须唯一且连续，否则可能出现跳跃或死循环
- 状态共享：校验类应是无状态的，数据通过参数传递

---

## 命令模式

### 3.1 定义与目的
- **定义**：将请求封装为一个对象，从而使你可用不同的请求对客户端进行参数化，并支持请求的队列、记录日志、撤销等操作。
- **目的**：
  - 将调用者与执行者解耦
  - 支持撤销/重做（Undo/Redo）
  - 支持排队/延迟执行（如任务队列）
  - 支持命令日志（用于恢复、审计）

### 3.2 结构
```
调用者 (Invoker) ──┬── Command（命令接口）
                   │         ├── ConcreteCommandA
                   │         └── ConcreteCommandB
                   │
                   └──► 接收者 (Receiver)
                          ├── 方法A()
                          └── 方法B()
```

### 3.3 核心角色
- Command（命令）：声明执行操作的接口，通常有 execute() 和 undo()
- ConcreteCommand（具体命令）：将请求封装为对象，持有 Receiver 引用
- Receiver（接收者）：真正执行业务逻辑的对象
- Invoker（调用者）：持有命令对象，调用 execute() 并管理命令历史
- Client（客户端）：创建具体命令，设置 Receiver，提交给 Invoker

### 3.4 代码结构
**简单版本（内存栈）**
```java
// 1. 命令接口
public interface Command {
    void execute();
    void undo();
}

// 2. 具体命令
public class GrantPermissionsCommand implements Command {
    private final RoleService roleService;
    private final Long roleId;
    private final List<Long> permissionIds;
    public GrantPermissionsCommand(RoleService roleService, Long roleId, List<Long> permissionIds) {
        this.roleService = roleService;
        this.roleId = roleId;
        this.permissionIds = permissionIds;
    }
    @Override
    public void execute() {
        roleService.grant(roleId, permissionIds);
    }
    @Override
    public void undo() {
        roleService.revoke(roleId, permissionIds);
    }
}

// 3. 调用器（维护历史栈）
@Component
public class CommandInvoker {
    private final Deque<Command> history = new ArrayDeque<>();
    public void execute(Command command) {
        command.execute();
        history.push(command);
    }
    public void undo() {
        if (history.isEmpty()) throw new IllegalStateException("无历史记录");
        Command command = history.pop();
        command.undo();
    }
}
```

**持久化版本（数据库记录）**
```java
// 命令历史实体（用于持久化）
@Entity
public class CommandHistory {
    private Long id;
    private String type;           // "grant" / "revoke"
    private String payload;        // JSON 序列化的参数
    private String status;         // "done" / "undone"
}

// Invoker（存储到数据库）
@Component
public class CommandInvoker {
    private final CommandHistoryRepository historyRepo;
    public void execute(Command command, String type, List<Long> args) {
        command.execute();
        historyRepo.save(new CommandHistory(type, toJson(args), "done"));
    }
    public void undo() {
        CommandHistory last = historyRepo.findTopByStatusOrderByIdDesc("done");
        Command command = rebuild(record);  // 反序列化重建命令
        command.undo();
        record.setStatus("undone");
        historyRepo.save(record);
    }
}
```

### 3.5 适用场景
- 撤销/重做：GUI 操作、编辑器
- 事务回滚：分布式事务补偿
- 宏命令：多个命令组合为一个
- 延迟执行：任务队列、定时任务
- 日志与恢复：系统崩溃后恢复状态

### 3.6 实际案例：角色权限编辑
**需求**：
- 管理员给角色批量分配权限
- 可随时撤销上一步操作
- 撤销历史跨会话持久化（重启服务不丢失）

**传统写法（问题）**：
```java
public void grantPermissions(Long roleId, List<Long> permissionIds) {
    Role role = roleRepository.findById(roleId);
    rolePermissionRepository.deleteByRoleId(roleId);
    rolePermissionRepository.batchInsert(roleId, permissionIds);
}
//撤销？不知道之前是什么状态，无法回退
```

**命令模式解决方案**：
```java
// 1. 命令接口
public interface RolePermissionCommand {
    void execute();
    void undo();
}

// 2. 授权命令
public class GrantCommand implements RolePermissionCommand {
    private final RolePermissionService service;
    private final Long roleId;
    private final List<Long> permissionIds;
    @Override
    public void execute() {
        service.insert(roleId, permissionIds);
    }
    @Override
    public void undo() {
        service.delete(roleId, permissionIds);
    }
}

// 3. 撤权命令
public class RevokeCommand implements RolePermissionCommand {
    private final RolePermissionService service;
    private final Long roleId;
    private final List<Long> permissionIds;
    @Override
    public void execute() {
        service.delete(roleId, permissionIds);
    }
    @Override
    public void undo() {
        service.insert(roleId, permissionIds);
    }
}

// 4. 调用器（持久化版）
@Component
public class RolePermissionInvoker {
    private final CommandHistoryRepository historyRepo;
    private final RolePermissionService service;
    public void grant(Long roleId, List<Long> permissionIds) {
        GrantCommand cmd = new GrantCommand(service, roleId, permissionIds);
        executeAndRecord(cmd, roleId, "grant", permissionIds);
    }
    public void undo(Long roleId) {
        CommandHistory last = historyRepo.findTopByRoleIdAndStatusOrderByIdDesc(
            roleId, "done");
        if (last == null) throw new IllegalStateException("无历史记录");
        RolePermissionCommand cmd = rebuildFromHistory(last);
        cmd.undo();
        last.setStatus("undone");
        historyRepo.save(last);
    }
    private void executeAndRecord(RolePermissionCommand cmd, Long roleId,
                                   String type, List<Long> permissionIds) {
        cmd.execute();
        historyRepo.save(new CommandHistory(roleId, type,
            toJson(permissionIds), "done"));
    }
}
```

### 3.7 注意事项
- 命令粒度：一个命令只做一个原子操作，避免 execute() 太长
- 幂等性：execute() 和 undo() 应可重复调用而不出错（常用于补偿）
- 资源释放：如果命令持有连接、文件等资源，需在 finally 中释放
- 循环依赖：命令持有 Receiver，Receiver 不应持有命令，否则循环引用
- 序列化：如需持久化，命令对象应实现 Serializable，参数需可序列化

---

### 对比总结

| 维度 | 策略模式 | 责任链模式 | 命令模式 |
|------|----------|------------|----------|
| 核心思想 | 封装算法，可互换 | 请求沿链传递，直到被处理 | 请求封装为对象 |
| 耦合关系 | 客户端 → 策略接口 | 客户端 → 链头（不感知后续） | 调用者 → 命令 → 接收者 |
| 对象间的联系 | 策略之间相互独立 | 有明确的父子链 | 命令持有接收者引用 |
| 典型场景 | 多种算法切换 | 多层审批、过滤器链 | 撤销重做、事务补偿 |
| 参数传递 | 上下文对象 | 沿链向下传递 | 构造函数注入命令对象 |

---