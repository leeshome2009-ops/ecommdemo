# Claude-Mem 和 OpenViking 分析

## 核心发现

根据腾讯云开发者社区的文章，这两个工具都是专门为 **OpenClaw** 和 **Claude Code** 设计的，用于解决 Token 消耗过快的问题。

---

## 1. Claude-Mem

### 是什么
专门为 OpenClaw 和 Claude Code 打造的**记忆压缩系统**。

### 核心功能
- ✅ **持久化内存** - 上下文可以跨会话保存
- ✅ **渐进式披露** - 分层内存检索，可见 token 成本
- ✅ **基于技能的搜索** - 使用 mem-search 技能查询项目历史
- ✅ **Web 查看器界面** - 在 http://localhost:37777 实时查看内存流
- ✅ **隐私控制** - 使用 `<private>` 标签排除敏感内容
- ✅ **上下文配置** - 精细控制注入的上下文内容
- ✅ **自动运行** - 无需手动干预

### 工作原理
1. 自动捕获操作记录
2. 生成语义摘要
3. 在需要时提供给 AI
4. 跨会话复用，节省 Token

### 快速安装

#### Claude Code 安装
```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

#### OpenClaw 一键安装
```bash
curl -fsSL https://install.cmem.ai/openclaw.sh | bash
```

### GitHub
https://github.com/thedotmack/claude-mem

---

## 2. OpenViking

### 是什么
由**火山引擎开源**的专为 AI Agent 设计的**上下文数据库** (Context Database)。

采用"文件系统范式"统一管理 Agent 的记忆、资源和技能。

### 解决的核心问题

❌ **传统方案的痛点**:
1. **上下文碎片化** - 记忆在代码里、资源在向量数据库、技能分散各处
2. **上下文需求激增** - Agent 长时间运行产生大量上下文
3. **检索效果差** - 传统 RAG 使用扁平存储，缺乏全局视角
4. **不可观测** - 传统 RAG 的隐式检索链路像黑盒
5. **记忆迭代有限** - 仅记录用户交互，缺少 Agent 任务记忆

✅ **OpenViking 的解决方案**:
1. **文件系统管理范式** - 统一管理记忆、资源和技能
2. **分层上下文加载** - L0/L1/L2 三层结构，按需加载
3. **目录递归检索** - 结合目录定位与语义搜索
4. **可视化检索轨迹** - 上下文可观测
5. **自动会话管理** - 自动压缩对话、提取长期记忆

### 核心特点

#### 1. 文件系统管理范式
```
像管理本地文件一样构建 Agent 的"大脑"

/agent/memory/
├── project/           # 项目记忆
├── decisions/         # 决策记录
├── tasks/            # 任务历史
└── conversations/    # 对话记录
```

#### 2. 分层上下文加载 (L0/L1/L2)
- **L0**: 核心上下文（必须加载）
- **L1**: 任务相关上下文（按需加载）
- **L2**: 历史记录（需要时加载）

**节省 Token**: 60-95%

#### 3. 目录递归检索
- 支持原生文件系统检索
- 结合目录定位 + 语义搜索
- 递归精准的上下文获取

#### 4. 可视化检索轨迹
- 清晰观察问题根源
- 指导检索逻辑优化

#### 5. 自动会话管理
- 自动压缩对话内容
- 提取长期记忆
- 让 Agent 越用越聪明

### 技术要求

#### 环境要求
- **Python**: 3.10+
- **操作系统**: Linux, macOS, Windows
- **网络**: 需要稳定连接

#### 模型依赖
- **VLM 模型**: 用于图像和内容理解
- **Embedding 模型**: 用于向量化和语义检索

### 安装方式

#### Python SDK
```bash
pip install openviking
```

#### CLI 工具
```bash
curl -fsSL https://raw.githubusercontent.com/volcengine/OpenViking/main/crates/ov_cli/install.sh | bash
```

或从源码构建:
```bash
cargo install --git https://github.com/volcengine/OpenViking ov_cli
```

### 配置示例

创建配置文件 `~/.openviking/ov.conf`:

```json
{
  "embedding": {
    "dense": {
      "api_base": "<api-endpoint>",
      "api_key": "<your-api-key>",
      "provider": "volcengine",
      "dimension": 1024,
      "model": "doubao-embedding-vision-250615"
    }
  },
  "vlm": {
    "api_base": "<api-endpoint>",
    "api_key": "<your-api-key>",
    "provider": "volcengine",
    "model": "doubao-seed-1-8-251228"
  }
}
```

---

## 3. 对比分析

### Claude-Mem vs OpenViking

| 特性 | Claude-Mem | OpenViking |
|------|-----------|------------|
| **定位** | 记忆压缩系统 | 上下文数据库 |
| **管理方式** | 自动捕获 + 语义摘要 | 文件系统范式 |
| **检索** | 技能搜索 (mem-search) | 目录递归 + 语义搜索 |
| **分层** | 渐进式披露 | L0/L1/L2 三层结构 |
| **可视化** | Web UI (localhost:37777) | 检索轨迹可视化 |
| **自动运行** | ✅ 是 | ✅ 是 |
| **适用场景** | Claude Code, OpenClaw | 通用 AI Agent |
| **开源** | ✅ GitHub | ✅ 火山引擎开源 |

### Token 节省效果

两者都能节省 **60-95%** 的 Token 消耗。

---

## 4. 推荐方案

### 对于当前电商项目

#### 方案 A: 使用 Claude-Mem（推荐）⭐⭐⭐

**理由**:
1. ✅ 专门为 Claude Code 和 OpenClaw 设计
2. ✅ 一键安装，配置简单
3. ✅ 自动运行，无需手动干预
4. ✅ 提供可视化的 Web UI
5. ✅ 我们当前就在 Claude Code/OpenClaw 环境中

**安装命令**:
```bash
# OpenClaw 网关一键安装
curl -fsSL https://install.cmem.ai/openclaw.sh | bash

# 或在 Claude Code 中
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

#### 方案 B: 使用 OpenViking

**理由**:
1. ✅ 更强大的上下文管理能力
2. ✅ 文件系统范式，易于理解
3. ✅ 分层加载更精细
4. ✅ 适合多智能体协作

**安装命令**:
```bash
pip install openviking
```

#### 方案 C: 两者结合（最佳）⭐⭐⭐⭐⭐

**组合使用**:
- **Claude-Mem**: 处理会话级记忆和快速检索
- **OpenViking**: 管理项目级上下文和多智能体协作

---

## 5. 实施建议

### 立即行动

1. **安装 Claude-Mem**
   ```bash
   # 在 OpenClaw 网关所在机器执行
   curl -fsSL https://install.cmem.ai/openclaw.sh | bash
   ```

2. **验证安装**
   - 访问 http://localhost:37777 查看 Web UI
   - 检查内存流是否正常

3. **配置 .gitignore**
   ```bash
   echo ".claude-mem/" >> .gitignore
   ```

4. **更新文档**
   - 在 MEMORY.md 中记录使用 Claude-Mem
   - 说明如何使用 mem-search 技能

### 后续优化

1. **集成 OpenViking**（如果需要更强的上下文管理）
2. **配置分层加载策略**
3. **设置隐私标签**保护敏感信息
4. **定期检查内存使用情况**

---

## 6. 总结

### 关键要点

1. **Claude-Mem 和 OpenViking 都是好东西** ✅
   - 都能节省 60-95% Token
   - 都是为 OpenClaw/Claude Code 设计
   - 都支持自动运行

2. **Claude-Mem 更轻量** ⚡
   - 一键安装
   - 开箱即用
   - 适合快速上手

3. **OpenViking 更强大** 🚀
   - 文件系统范式
   - 分层加载
   - 适合复杂场景

### 推荐决策

**对于当前的电商项目**:

✅ **优先使用 Claude-Mem**
- 简单高效
- 无需复杂配置
- 满足当前需求

⏭️ **根据需要考虑 OpenViking**
- 如果项目规模扩大
- 如果需要更强的上下文管理
- 如果需要多智能体协作

---

**结论**: 两个工具都很好！Claude-Mem 更适合快速上手，OpenViking 更适合复杂场景。建议先安装 Claude-Mem，体验效果后再决定是否需要 OpenViking。
