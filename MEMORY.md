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
- 框架: Vue 3 + TypeScript
- 构建: Vite
- UI: Element Plus (后台) / 自定义组件 (C端)
- 状态管理: Pinia
- 路由: Vue Router 4
- HTTP: Axios
- 目录: `src/frontend/`

### 后端
- 框架: Java + Spring Boot 3.x
- ORM: MyBatis-Plus
- 数据库: MySQL 8.0
- 缓存: Redis
- 认证: Spring Security + JWT
- 目录: `src/backend/`

### 共享
- API 类型定义: `src/shared/`
- 文档: `docs/`

## 项目结构

```
/data/ecommerce-shop/
├── src/
│   ├── frontend/          # 前端代码 (Vue 3)
│   │   ├── src/
│   │   │   ├── views/     # 页面组件
│   │   │   ├── components/# 通用组件
│   │   │   ├── composables/ # 组合式函数
│   │   │   ├── stores/    # Pinia Store
│   │   │   ├── api/       # API 调用
│   │   │   ├── utils/     # 工具函数
│   │   │   ├── router/    # 路由配置
│   │   │   └── types/     # TypeScript 类型
│   │   └── package.json
│   ├── backend/           # 后端代码 (Spring Boot)
│   │   ├── src/main/java/
│   │   │   ├── controller/  # 控制器层
│   │   │   ├── service/     # 业务逻辑层
│   │   │   ├── mapper/      # 数据访问层 (MyBatis-Plus)
│   │   │   ├── entity/      # 实体类
│   │   │   ├── dto/         # 数据传输对象
│   │   │   ├── vo/          # 视图对象
│   │   │   ├── config/      # 配置类
│   │   │   └── util/        # 工具类
│   │   ├── src/main/resources/
│   │   │   ├── mapper/      # MyBatis XML 映射
│   │   │   └── application.yml
│   │   └── pom.xml
│   └── shared/            # 前后端共享类型
├── tests/                 # 测试
├── docs/                  # 文档
├── scripts/               # 脚本
└── .context/              # 上下文（决策、任务、通信）
```

## 核心模块

| 模块 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 用户认证 | 注册/登录/JWT | 登录页、注册页 | AuthController + Spring Security + JWT |
| 商品管理 | 商品 CRUD、分类、搜索 | 商品列表/详情页 | ProductController + Service + MyBatis-Plus |
| 购物车 | 添加/删除/数量修改 | 购物车页面 | CartController + Redis 缓存 |
| 订单系统 | 下单/支付/状态流转 | 订单页面 | OrderController + 状态机 |
| 支付 | 支付集成 | 收银台页面 | PaymentController |
| 后台管理 | 商品/订单/用户管理 | Admin 页面 | AdminController |

## 工作规范

1. 收到需求后先确认理解，再动手编码
2. 前端代码写到 `src/frontend/`，后端代码写到 `src/backend/`
3. 新增 API 时前后端同步定义类型到 `src/shared/`
4. 写完代码后运行测试确认通过
5. 代码变更后 `git add && git commit` 记录变更
6. 遇到不确定的架构决策，记录到 `.context/decisions/` 并在群里讨论
7. 完成任务后在群里汇报进度

## 编码规范

- Java: 遵循阿里巴巴 Java 开发手册，使用 Checkstyle
- Vue 3: 使用 Composition API，组件使用 Setup 语法糖
- TypeScript: 严格模式
- API: RESTful 风格，统一返回格式 `{ code, message, data }`
- 错误处理: 后端用全局异常处理器，前端用统一错误提示
- 命名:
  - Java: 类名用 PascalCase，方法和变量用 camelCase
  - Vue: 组件用 PascalCase，文件用 kebab-case
  - 数据库: 表名和字段用 snake_case

## Claude Code 使用方式

### 调用命令
```bash
# 前台执行（快速任务）- 自动接受编辑，无需确认
claude --permission-mode acceptEdits --print "具体任务描述"

# 后台执行（长时间任务）
claude --permission-mode acceptEdits --print "任务描述" &
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

### 2026-03-17 - 确定技术栈：Vue 3 + Spring Boot
- **决策**: 前端使用 Vue 3，后端使用 Java Spring Boot
- **原因**: 根据项目需求调整
- **前端**: Vue 3 + TypeScript + Vite + Element Plus + Pinia
- **后端**: Java 17+ + Spring Boot 3.x + MyBatis-Plus + Redis + MySQL

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
