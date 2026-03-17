# OpenViking 多机器人协作集成方案

## 背景分析

### 当前问题
在电商购物系统开发中，存在多个机器人：
1. **开发机器人** (DevBot) - 负责代码实现和单元测试
2. **测试机器人** (TestBot) - 负责集成测试和联调测试
3. **未来可能的机器人** - 部署机器人、监控机器人等

**挑战**：
- ❌ 上下文不共享：每个机器人独立运行，不了解其他机器人的工作
- ❌ 决策不同步：架构决策、技术选型无法实时同步
- ❌ 状态不一致：项目状态、进度跟踪各自维护
- ❌ 重复工作：可能重复读取相同的文件或执行相同的分析

## OpenViking 集成方案

### 方案概述

OpenViking 是一个多机器人协作框架，可以实现：
- ✅ 上下文共享（Shared Context）
- ✅ 消息传递（Message Passing）
- ✅ 状态同步（State Synchronization）
- ✅ 任务编排（Task Orchestration）

### 架构设计

```
┌─────────────────────────────────────────────────────────┐
│                   OpenViking Core                        │
│  (Context Manager | Message Bus | State Sync)            │
└─────────────────────────────────────────────────────────┘
           │                  │                  │
           ▼                  ▼                  ▼
    ┌──────────┐       ┌──────────┐       ┌──────────┐
    │ DevBot   │       │ TestBot  │       │  其他Bot  │
    │ 开发机器人│       │ 测试机器人│       │          │
    └──────────┘       └──────────┘       └──────────┘
           │                  │                  │
           └──────────────────┴──────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ Shared Storage  │
                    │ - MEMORY.md     │
                    │ - decisions/    │
                    │ - tasks/        │
                    │ - comms/        │
                    └─────────────────┘
```

### 实现方案

#### 1. 共享上下文层（Shared Context）

**位置**: `.context/` 目录

**结构**:
```
.context/
├── shared/
│   ├── context.json          # 全局上下文
│   ├── decisions.json        # 决策记录
│   ├── tasks.json            # 任务队列
│   └── state.json            # 项目状态
├── devbot/
│   └── local-state.json      # DevBot 本地状态
├── testbot/
│   └── local-state.json      # TestBot 本地状态
└── communications/           # 机器人间通信记录
    ├── dev-to-test.json
    └── test-to-dev.json
```

**上下文内容**:
```json
{
  "project": {
    "name": "ecommerce-shop",
    "techStack": {
      "frontend": "Vue 3 + TypeScript",
      "backend": "Spring Boot 3.x"
    },
    "status": "development"
  },
  "agents": {
    "devbot": {
      "role": "development",
      "status": "active",
      "currentTask": "implement user auth"
    },
    "testbot": {
      "role": "testing",
      "status": "idle",
      "lastTest": "2026-03-17T16:00:00Z"
    }
  },
  "decisions": [
    {
      "id": "2026-03-17-001",
      "topic": "技术栈选型",
      "decision": "Vue 3 + Spring Boot",
      "by": "devbot",
      "timestamp": "2026-03-17T15:00:00Z"
    }
  ],
  "tasks": [
    {
      "id": "task-001",
      "title": "实现用户认证",
      "assignee": "devbot",
      "status": "in_progress",
      "dependencies": []
    }
  ]
}
```

#### 2. 消息传递机制（Message Passing）

**实现方式**: 文件系统 + 轮询

**消息格式**:
```json
{
  "id": "msg-001",
  "from": "devbot",
  "to": "testbot",
  "type": "task_complete",
  "payload": {
    "task": "实现用户登录功能",
    "files": ["src/backend/.../AuthController.java"],
    "commit": "abc123"
  },
  "timestamp": "2026-03-17T16:30:00Z",
  "status": "unread"
}
```

**通信流程**:
```
DevBot 完成开发
    ↓
1. 更新本地状态
2. 写入消息到 .context/communications/dev-to-test.json
3. 更新共享上下文 .context/shared/tasks.json
    ↓
TestBot 轮询检测
    ↓
1. 读取 .context/communications/dev-to-test.json
2. 标记消息为 "read"
3. 开始集成测试
4. 写入测试结果到 .context/communications/test-to-dev.json
    ↓
DevBot 读取反馈
    ↓
1. 读取 .context/communications/test-to-dev.json
2. 根据反馈修复 Bug 或继续下一个任务
```

#### 3. 状态同步（State Synchronization）

**同步策略**:

1. **事件驱动同步**
   - 每次重要操作后触发同步
   - Git 提交、任务完成、决策更新

2. **定时同步**
   - 每分钟检查一次共享上下文
   - 检测其他机器人的更新

3. **冲突解决**
   - 使用时间戳
   - 最后写入胜出（Last Write Wins）
   - 关键冲突需要人工介入

#### 4. 任务编排（Task Orchestration）

**任务队列**:
```json
{
  "queue": [
    {
      "id": "task-001",
      "title": "实现用户认证",
      "assignee": "devbot",
      "status": "completed",
      "dependencies": [],
      "output": {
        "files": ["..."],
        "commit": "abc123"
      }
    },
    {
      "id": "task-002",
      "title": "用户认证集成测试",
      "assignee": "testbot",
      "status": "pending",
      "dependencies": ["task-001"],
      "trigger": "auto"
    }
  ]
}
```

**触发机制**:
```javascript
// 伪代码
if (task-001.status === "completed" && task-002.dependencies.includes("task-001")) {
  triggerTask("task-002");
}
```

### 具体实现步骤

#### 阶段 1: 基础设施搭建

1. **创建共享上下文结构**
```bash
mkdir -p .context/{shared,devbot,testbot,communications}
```

2. **定义上下文 Schema**
```json
// .context/shared/schema.json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Shared Context Schema",
  "type": "object",
  "properties": {
    "project": { "type": "object" },
    "agents": { "type": "object" },
    "decisions": { "type": "array" },
    "tasks": { "type": "array" }
  }
}
```

3. **实现消息传递工具**
```python
# tools/context_manager.py
class ContextManager:
    def read_shared_context(self):
        """读取共享上下文"""
        pass

    def update_shared_context(self, updates):
        """更新共享上下文"""
        pass

    def send_message(self, to_agent, message):
        """发送消息给其他机器人"""
        pass

    def read_messages(self):
        """读取发给自己的消息"""
        pass
```

#### 阶段 2: 机器人集成

1. **DevBot 集成**
   - 在 MEMORY.md 中添加上下文管理说明
   - 每次任务完成后更新共享上下文
   - 定期检查来自 TestBot 的消息

2. **TestBot 集成**
   - 定期检查共享上下文中的任务完成
   - 自动触发集成测试
   - 反馈测试结果

#### 阶段 3: 工作流自动化

1. **自动化触发**
```yaml
# .context/workflows/task-handoff.yml
name: Task Handoff
on:
  - event: task_completed
    from: devbot
steps:
  - name: Notify testbot
    action: send_message
    to: testbot
    message: "User auth feature ready for testing"
  - name: Create test task
    action: create_task
    assignee: testbot
    title: "Test user auth feature"
```

2. **状态监控**
```bash
# 实时查看所有机器人状态
cat .context/shared/context.json | jq '.agents'
```

### 优势分析

#### ✅ 优势
1. **上下文共享**: 所有机器人看到相同的项目状态
2. **避免重复**: 不需要重复读取文件或分析代码
3. **实时同步**: 决策和状态变更立即共享
4. **任务编排**: 自动化任务流转
5. **可追溯**: 所有通信和决策都有记录

#### ⚠️ 挑战
1. **并发控制**: 多个机器人同时写入需要锁机制
2. **数据一致性**: 确保上下文不被污染
3. **性能考虑**: 频繁的文件 I/O 可能影响性能
4. **错误处理**: 机器人故障时的恢复机制

### 替代方案

如果 OpenViking 不可用，可以使用：

1. **Git 作为同步机制**
   - 每个机器人提交到独立分支
   - 通过 PR/MR 进行通信
   - 利用 Git 的冲突解决机制

2. **Redis 作为消息队列**
   - 使用 Redis Pub/Sub
   - 轻量级消息传递
   - 已经在 Docker 环境中运行

3. **简单的文件共享**
   - 就像本方案描述的
   - 无需额外依赖
   - 容易实现和调试

## 推荐方案

**对于当前项目，推荐使用 "简单的文件共享" 方案**：

### 理由
1. ✅ 无需额外依赖（OpenViking 可能不存在或太复杂）
2. ✅ 易于实现和调试
3. ✅ Git 自动跟踪所有变更
4. ✅ 可以逐步演进到更复杂的系统

### 下一步行动
1. 创建 `.context/shared/` 结构
2. 定义共享上下文格式
3. 在 MEMORY.md 中添加协作规范
4. DevBot 和 TestBot 都遵循相同的上下文访问协议

---

**结论**: 不依赖特定的 OpenViking 框架，使用文件系统 + Git 实现轻量级的多机器人协作机制。
