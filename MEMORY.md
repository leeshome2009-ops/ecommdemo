# MEMORY.md - 开发机器人记忆库

## 角色定位

**身份**: 飞书智能开发机器人 - 全栈开发助手

**职责**:
1. 在飞书群聊中通过 @机器人 接收开发需求
2. 理解项目负责人描述的功能需求
3. 调用 Claude Code 进行代码实现
4. 监控开发进度并汇报
5. 管理 Git 仓库和代码提交

**工作流程**:
```
飞书群 @机器人 → 需求理解 → 调用 Claude Code → 监控进度 → 汇报结果
```

## 技术栈

### 前端
- 框架: React 18 + TypeScript
- 构建: Vite
- UI: Ant Design (后台) / 自定义组件 (C端)
- 状态管理: Zustand
- 路由: React Router v6
- HTTP: Axios
- 目录: `src/frontend/`

### 后端
- 框架: Go + Gin
- ORM: GORM
- 数据库: MySQL
- 缓存: Redis
- 认证: JWT
- 目录: `src/backend/`

### 共享
- API 类型定义: `src/shared/`
- 文档: `docs/`

## 项目结构

```
/data/ecommerce-shop/
├── src/
│   ├── frontend/          # 前端代码
│   │   ├── src/
│   │   │   ├── pages/     # 页面组件
│   │   │   ├── components/# 通用组件
│   │   │   ├── hooks/     # 自定义 Hooks
│   │   │   ├── stores/    # Zustand Store
│   │   │   ├── services/  # API 调用
│   │   │   ├── utils/     # 工具函数
│   │   │   └── types/     # TypeScript 类型
│   │   └── package.json
│   ├── backend/           # 后端代码
│   │   ├── cmd/           # 入口
│   │   ├── internal/
│   │   │   ├── handler/   # HTTP Handler
│   │   │   ├── service/   # 业务逻辑
│   │   │   ├── model/     # 数据模型
│   │   │   ├── repo/      # 数据访问层
│   │   │   └── middleware/ # 中间件
│   │   ├── pkg/           # 公共包
│   │   └── go.mod
│   └── shared/            # 前后端共享类型
├── tests/                 # 测试
├── docs/                  # 文档
├── scripts/               # 脚本
└── .context/              # 上下文（决策、任务、通信）
```

## 核心模块

| 模块 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 用户认证 | 注册/登录/JWT | 登录页、注册页 | auth handler + JWT middleware |
| 商品管理 | 商品 CRUD、分类、搜索 | 商品列表/详情页 | product handler + service |
| 购物车 | 添加/删除/数量修改 | 购物车页面 | cart handler + Redis 缓存 |
| 订单系统 | 下单/支付/状态流转 | 订单页面 | order handler + 状态机 |
| 支付 | 支付集成 | 收银台页面 | payment handler |
| 后台管理 | 商品/订单/用户管理 | Admin 页面 | admin handler |

## 工作规范

1. 收到需求后先确认理解，再动手编码
2. 前端代码写到 `src/frontend/`，后端代码写到 `src/backend/`
3. 新增 API 时前后端同步定义类型到 `src/shared/`
4. 写完代码后运行测试确认通过
5. 代码变更后 `git add && git commit` 记录变更
6. 遇到不确定的架构决策，记录到 `.context/decisions/` 并在群里讨论
7. 完成任务后在群里汇报进度

## 编码规范

- Go: 遵循 Effective Go，使用 gofmt 格式化
- TypeScript: 严格模式，组件使用函数式写法
- API: RESTful 风格，统一返回格式 `{ code, message, data }`
- 错误处理: 后端用统一错误码，前端用统一错误提示
- 命名: Go 用驼峰，TS 组件用 PascalCase，文件用 kebab-case

## Claude Code 使用方式

### 调用命令
```bash
# 前台执行（快速任务）
claude --print "具体任务描述"

# 后台执行（长时间任务）
claude --print "任务描述" &
```

### 工作目录
- 所有操作在 `/data/ecommerce-shop` 下进行
- 使用 `workdir` 参数指定工作目录

### 监控方式
- 使用 `process` 工具监控后台任务
- `process action:list` - 查看运行中的任务
- `process action:log` - 查看任务输出
- `process action:poll` - 等待任务完成

## 项目状态

**初始化时间**: 2026-03-17
**当前状态**: 项目骨架已搭建，代码目录为空
**Git 状态**: 初始 commit `init: ecommerce shop project structure`
**下一步**: 等待具体功能需求开始开发

## 重要决策记录

### 2026-03-17 - 确定使用 Claude Code 作为代码实现工具
- **决策**: 使用 Claude Code (claude CLI) 进行代码编写
- **原因**: 用户已经通过 Claude Code 完成项目初始化，希望延续使用
- **命令格式**: `claude --print "任务描述"`
- **工作目录**: `/data/ecommerce-shop`
- **监控方式**: 通过 process 工具监控执行进度

## 飞书集成信息

**机器人名称**: 开发机器人
**唤醒方式**: 群聊中 @机器人
**主要用户**: 项目负责人
**交互模式**: 
- 接收需求描述
- 确认理解
- 调用 Claude Code 实现
- 汇报进度和结果

## 上下文文件说明

- `AGENTS.md` - 开发 Agent 规范
- `IDENTITY.md` - 机器人身份定义
- `SOUL.md` - 机器人个性设定
- `USER.md` - 用户信息记录
- `TOOLS.md` - 工具使用说明
- `MEMORY.md` - 本文件，记忆库
- `HEARTBEAT.md` - 心跳任务（当前为空）
