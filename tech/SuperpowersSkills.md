下面我用「**高效使用案例**」的方式，帮你把 [obra/superpowers](https://github.com/obra/superpowers) 这套“技能化工作流”真正用起来——不止告诉你有啥命令，而是给你**可复制的实战套路、提示词模板、以及什么时候用哪条技能链**。Superpowers 的核心是：让编码代理别“上来就写代码”，而是按**设计 → 计划 → TDD 执行 → 评审 → 收尾**的纪律化流程跑。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 0) 先把“高效”前提打好：安装 & 你会看到什么

### Claude Code（插件市场安装）

```text
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
/help   # 你应能看到 /superpowers:brainstorm /write-plan /execute-plan
```

这些步骤与可用命令在仓库 README 有明确说明。 [\[github.com\]](https://github.com/obra/superpowers)

> Codex / OpenCode 属于“手动接入”，README 也给了对应 INSTALL 文档入口。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 1) 案例一：做新功能（从“想法”到“可连续自主执行”）

### 适用场景

*   新增模块/接口/页面
*   需求不清晰、有分歧、要做技术选型

### 高效打法（3 步）

1.  **/brainstorm：把需求“逼”清楚**
2.  **/write-plan：拆成 2–5 分钟的小任务**
3.  **/execute-plan 或 subagent-driven-development：按计划执行 + 两阶段评审**

这就是 README 里定义的基本工作流骨架：brainstorming → worktrees → writing-plans →（subagent-driven-development 或 executing-plans）→ TDD → code review → finishing branch。 [\[github.com\]](https://github.com/obra/superpowers)

### 你可以直接复制的对话模板

```text
/superpowers:brainstorm
我要做一个「OAuth 登录 + 绑定已有账号」功能。
约束：
- 后端：Node/Express
- 数据库：Postgres
- 现有登录：邮箱 + 密码
目标：
- 支持 Google OAuth
- 已有邮箱用户首次 OAuth 登录时要提示绑定
非目标：
- 暂不做多提供商
请你用提问把需求澄清，并给出可评审的设计分块输出。
```

> Superpowers 的 brainstorming 会在写代码前激活，通过提问澄清需求、比较方案并分块展示设计供你确认。 [\[github.com\]](https://github.com/obra/superpowers)

然后：

```text
/superpowers:write-plan
基于刚刚确认的设计，写一个可执行计划：
- 任务颗粒度 2-5 分钟
- 每个任务包含精确文件路径、完整代码位置、验证步骤/命令
- 全程按真·TDD（red/green/refactor）
```

> README 明确写了计划拆分粒度、每任务要包含文件路径/代码/验证步骤，并强调 TDD、YAGNI、DRY。 [\[github.com\]](https://github.com/obra/superpowers)

最后：

```text
/superpowers:execute-plan
按计划批量执行，设置人工检查点：
- 每完成 3 个任务停一下汇报
- 遇到阻塞立即停下并按系统化调试流程处理
```

> README 提到执行阶段可走 subagent-driven-development（按任务派发子代理并两阶段评审）或 executing-plans（分批执行并人工检查点）。 [\[github.com\]](https://github.com/obra/superpowers)

✅ **效率收益点**：你把“不可控的生成”变成“可审计的工程流水线”，代理能连续跑更久、偏航更少（README 也强调可在不偏离计划情况下自主工作较长时间）。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 2) 案例二：改动风险高？用 Git Worktrees 做“隔离实验室”

### 适用场景

*   大重构、依赖升级、跨模块修改
*   你不想污染主分支、也不想把工作区搞乱

### 高效打法

*   设计通过后，让技能 **using-git-worktrees** 自动创建隔离工作区：新分支 + 项目 setup + 先跑一遍“干净基线测试”。 [\[github.com\]](https://github.com/obra/superpowers)

你可以这样要求：

```text
在开始实现前，请按 using-git-worktrees：
- 创建隔离 worktree 分支
- 运行项目初始化
- 确认修改前测试全绿（baseline）
```

> 这三点在 README 对 using-git-worktrees 的描述里是明确的激活行为。 [\[github.com\]](https://github.com/obra/superpowers)

✅ **效率收益点**：把“尝试性改动”从主工作区剥离出来，回滚/丢弃成本极低；也天然适合并行推进多个任务（见案例六）。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 3) 案例三：把代理“锁”进真·TDD：先红再绿，不许偷跑

### 适用场景

*   你发现 AI 常见问题：先写实现、补测试、甚至不测
*   你想要更稳的可维护代码

### 高效打法

Superpowers 的 **test-driven-development** 会强制 RED-GREEN-REFACTOR：

*   先写失败测试并“亲眼看到失败”
*   写最小实现让测试通过
*   再重构  
    甚至强调：**测试前写的代码可能会被删除**。 [\[github.com\]](https://github.com/obra/superpowers)

可复制指令：

```text
接下来严格按 test-driven-development：
1) 先写一个失败测试（要能表达需求）
2) 运行测试并贴出失败结果
3) 写最小实现让它通过
4) 重构（保持全绿）
每一步都要给出执行的测试命令与输出摘要
```

✅ **效率收益点**：你用“流程约束”换“质量与回归安全”，后面迭代速度反而更快（少返工）。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 4) 案例四：线上/复杂 Bug：别“试一试”，走系统化调试 + 完成前验证

### 适用场景

*   偶发 500、竞态、缓存不一致、随机登出、某些环境才复现
*   你不希望 AI 胡乱改动造成更多不确定性

### 高效打法（两条技能链）

1.  **systematic-debugging**：4 阶段根因分析流程 [\[github.com\]](https://github.com/obra/superpowers)
2.  **verification-before-completion**：在宣布修复前必须验证“确实修好了” [\[github.com\]](https://github.com/obra/superpowers)

可复制输入：

```text
请启用 systematic-debugging：
- 先定义问题与最小复现路径
- 收集证据（日志、调用栈、配置差异、请求样本）
- 提出多个假设并逐个设计实验验证
- 只在证据支持时修改代码

修复后必须按 verification-before-completion：
- 给出能证明修复有效的验证步骤（含回归测试）
- 说明还可能失败的边界与监控建议
```

✅ **效率收益点**：把“玄学调试”变成“实验驱动”，减少引入新 bug 的概率。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 5) 案例五：让代理帮你做“可执行”的代码评审（而不是泛泛而谈）

### 适用场景

*   你要合并 PR、或代理已完成一批任务
*   你想要“按严重级别分类”的审查结论

Superpowers 的 **requesting-code-review** 会在任务之间激活，按计划逐项审、按严重度报告，严重问题会阻塞推进。  
而 **receiving-code-review** 则用于结构化地回应反馈（分类、改动策略、复测）。 [\[github.com\]](https://github.com/obra/superpowers)

可复制请求：

```text
请按 requesting-code-review：
- 对照 write-plan 的每个任务验收
- 问题按 Critical/Major/Minor 分类
- Critical 必须给出具体修复建议与验证方式

如果我给出评审意见，请按 receiving-code-review：
- 逐条归类
- 明确接受/拒绝及理由
- 给出修复顺序与回归测试清单
```

✅ **效率收益点**：你得到的是“能落地的审查清单”，不是“看起来很懂”的评论。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 6) 案例六：并行推进多个子任务：并发子代理 + Worktrees

### 适用场景

*   你有多个相互独立的小任务（UI、API、测试、文档）
*   想加速但又怕上下文互相污染

README 里列出的 **dispatching-parallel-agents** 与 **subagent-driven-development** 就是为并行与子代理任务执行设计的；同时用 worktrees 保持隔离。 [\[github.com\]](https://github.com/obra/superpowers)

一个很实用的“并行拆分法”：

```text
把工作分成 3 条并行线：
A) 测试与用例补齐
B) 最小实现
C) 文档/示例与边界条件
请用 dispatching-parallel-agents 分派子代理，并确保每条线都遵循 TDD 与验证步骤。
```

✅ **效率收益点**：并行带来的吞吐提升，靠“隔离 + 计划 + 验证”来守住质量。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 7) 案例七：收尾阶段自动化：合并/提 PR/清理工作区，一次做对

当计划任务都完成后，**finishing-a-development-branch** 会验证测试、给你 merge/PR/保留/丢弃选项，并清理 worktree。 [\[github.com\]](https://github.com/obra/superpowers)

你可以这样“卡口”：

```text
在 finishing-a-development-branch 阶段：
- 先跑全量测试（提供命令与结果摘要）
- 输出变更摘要（按模块）
- 给出 merge / 开 PR / 丢弃 的推荐与理由
- 清理 worktree
```

✅ **效率收益点**：避免“功能做完了但分支乱、测试没跑、PR 没写”的尾部摩擦。 [\[github.com\]](https://github.com/obra/superpowers)

***

## 8) 案例八：团队/个人流程升级：写自己的 Skill，让“最佳实践”固化

Superpowers 不只是用现成技能，还鼓励你按 **writing-skills** 的规范新增技能并贡献。  
README 也明确了贡献路径：fork → 建分支 → 跟随 writing-skills → PR。 [\[github.com\]](https://github.com/obra/superpowers)

适用场景：

*   你团队有固定规范（提交信息、目录结构、日志字段、错误码、ADR）
*   你希望代理每次都自动遵循，而不是每次提醒

可复制请求：

```text
请按 writing-skills：
- 为“我们的 API 错误码与日志规范”创建一个技能
- 包含：触发条件、强制规则、检查清单、示例
- 给出如何测试该技能在实际任务中会被正确触发
```

***

## 9) 三条“最值钱”的高效心法（用对了会明显提速）

### 心法 A：你越“具体”，/write-plan 越像项目经理

因为计划技能要求每任务都要有**精确文件路径、完整代码、验证步骤**。  
👉 所以你输入时要给：目录结构、测试命令、框架版本、CI 约束。 [\[github.com\]](https://github.com/obra/superpowers)

### 心法 B：用“检查点”控制 autonomy

executing-plans 支持分批执行并人工检查点；subagent-driven-development 支持每任务两阶段评审。  
👉 赶进度用批量；高风险改动用两阶段评审。 [\[github.com\]](https://github.com/obra/superpowers)

### 心法 C：把“完成定义”写进验证步骤

Superpowers 强调“证据优于声明”，并提供 verification-before-completion。  
👉 你的 DoD（Definition of Done）要包含：测试、回归点、性能/日志验证、边界用例。 [\[github.com\]](https://github.com/obra/superpowers)

***