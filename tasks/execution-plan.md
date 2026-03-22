# Vibe Knowledge Overhaul 执行计划

## 1. 目标摘要

本轮重构目标是把当前以 `workspace` 为中心的执行型产品，演进为以知识与问题推进为中心的产品：

- `project -> branch -> session -> note`
- `global note` 作为跨 session 的独立知识对象
- `workspace` 继续保留为底层运行时概念，但不在 UI 中暴露
- 正文内容统一落到仓库外 Markdown 文件，由外部编辑器接管编辑，产品内只做只读预览
- branch 之间要支持阻塞、派生、回流、提升到更全局位置的流转

## 2. 设计定稿

### 2.1 核心对象

- `project`
  - 稳定容器
  - 第一版只保留名称和主仓库
  - 不进入状态流转

- `branch`
  - 对应真实 Git branch
  - 对应一个隐藏 `workspace`
  - 必填字段：目的、本次涉及内容、前置条件、预期结果
  - 同时有 Markdown 正文文件
  - 状态：进行中、被阻塞、待深挖、已解决、已归档

- `session`
  - branch 下的一个完整问题处理单元
  - 原则上一个 session 只处理一个主题
  - 允许多轮问答
  - 必填字段与 branch 相同
  - 同时有 Markdown 正文文件

- `note`
  - 归属 session
  - 轻量对象
  - 必填：标题、正文
  - 可选：标签
  - 无状态

- `global note`
  - 独立对象
  - 可由多个 session note 提炼而来
  - 与原始 note 不是同一个对象
  - 保留来源关系
  - 第一版无状态、无独立 history

### 2.2 正文存储

- 所有 `branch / session / note / global note` 正文都存为 Markdown 文件
- 文件放在仓库外专用目录
- 每个 project 一个文件夹
- project 文件夹下采用平铺结构
- 层级与关系通过数据库和 frontmatter 管理
- Markdown 文件命名与 frontmatter v1 规范见 `tasks/markdown-frontmatter-spec.md`

### 2.3 编辑与预览

- 正文编辑统一交给外部编辑器
- 第一版优先走“打开外部编辑器”能力
- 产品内只展示只读预览

### 2.4 关系模型

第一版固定关系类型：

- `来源于`
- `相关`
- `补充`
- `依赖`
- `冲突`
- `启发`

关系既要参与展示，也要参与筛选和搜索。

## 3. OpenCLI 测试模块

## 3.1 定位

`opencli` 不替代当前仓库已有的 Preview/Dev Server 能力，而是新增一个“浏览器自动化测试模块”：

- 现有 Preview 负责人工查看页面、调试、点击选上下文
- `opencli` 负责把浏览器操作脚本化、CLI 化、可编排化

从 `opencli` README 可确认的关键信息：

- 它是统一 CLI Hub，可把网站、桌面应用、本地 CLI 暴露为命令
- 支持 AI 通过 `opencli list` 发现可用工具
- 浏览器接入依赖 Chrome Extension + daemon
- 通过 `opencli doctor` 做诊断

来源：

- <https://github.com/jackwener/opencli>
- README 关键命令：`opencli list`、`opencli doctor`

## 3.2 第一版接入策略

第一版不把 `opencli` 深度嵌进 agent 自动执行链，而是先做成一个独立测试模块：

- 后端新增 `browser_testing` 服务
- 支持配置 `opencli` 可执行路径
- 支持执行预定义测试场景
- 支持保存测试计划与执行结果
- 前端只做一个基础入口，不做复杂测试编辑器

## 3.3 第一版最小能力

- 检查 `opencli` 是否可用
- 运行 `opencli doctor`
- 运行一个预定义浏览器测试命令
- 保存测试记录到数据库
- 把测试记录挂到 branch/session 以便追踪

## 3.4 适合延后的能力

- 让 agent 自动生成 opencli 测试计划
- 可视化录制浏览器动作
- 丰富的测试模板编辑器
- 多环境、多浏览器矩阵执行

## 4. 可并行执行的任务批次

下面这些批次彼此依赖较少，适合多开分支并行推进。

### 批次 A：基础数据层与类型

目标：

- migrations
- Rust models
- shared types 生成

适合独立分支：

- `feat/branch-schema`
- `feat/session-note-schema`
- `feat/relation-schema`

### 批次 B：文件与外部编辑器能力

目标：

- 仓库外 Markdown 路径解析
- 文件命名与 frontmatter
- 只读预览
- 打开外部编辑器

适合独立分支：

- `feat/markdown-storage`
- `feat/external-editor-open`

### 批次 C：Branch / Session 后端 API

目标：

- branch CRUD
- branch block/unblock/derive
- session detail / create / update

适合独立分支：

- `feat/branch-api`
- `feat/session-domain-api`

### 批次 D：Session Note / Global Note / Search / Relation API

目标：

- note CRUD
- global note CRUD
- source mapping
- relation API
- knowledge search

适合独立分支：

- `feat/session-note-api`
- `feat/global-note-api`
- `feat/knowledge-search`

### 批次 E：前端导航与页面骨架

目标：

- 新路由
- 新 AppNavigation
- branch/session/knowledge/canvas 页面骨架

适合独立分支：

- `feat/project-knowledge-routes`
- `feat/branch-page-shell`
- `feat/session-page-shell`

### 批次 F：Session 页面与 Note 工作区

目标：

- 左 note、右 chat
- note 列表
- note 搜索与标签筛选
- note 只读预览

适合独立分支：

- `feat/session-note-panel`

### 批次 G：Knowledge Workbench

目标：

- global note 列表
- 来源区块
- 关系区块

适合独立分支：

- `feat/knowledge-workbench`

### 批次 H：Canvas

目标：

- 节点模型
- 关系创建
- 侧边栏筛选管理

适合独立分支：

- `feat/canvas-foundation`
- `feat/canvas-relations`

### 批次 I：OpenCLI 浏览器测试模块

目标：

- opencli 可用性检测
- 测试命令执行
- 结果记录

适合独立分支：

- `feat/opencli-browser-testing`

## 5. 依赖关系

最重要的依赖顺序：

1. 批次 A 必须先于 C、D、E
2. 批次 B 可与 A 并行，但会被 F、G 依赖
3. 批次 C、D 完成后，E/F/G 才能真正联调
4. 批次 H 依赖 D 的关系模型
5. 批次 I 可相对独立，最早在 A 之后就能启动

## 6. 下一会话建议直接启动的第一个任务

建议从这个最小执行单元开始：

- 任务名：`新增笔记根目录配置项与默认路径策略`

原因：

- 它是后续 Markdown 文件体系的根
- 依赖最少
- 可以非常快完成
- 验收简单明确

完成后下一步紧接：

- `明确 Markdown 文件命名规则与 frontmatter 规范`
- `设计 projects 单仓库化改造方案`
- `设计 branches 表字段与约束`

## 7. 相关文件清单

### 已有计划文件

- [tasks/todo.md](/home/lizhe/CodeOri/vibe-kanban-overview/tasks/todo.md)
- [tasks/lessons.md](/home/lizhe/CodeOri/vibe-kanban-overview/tasks/lessons.md)
- [tasks/execution-plan.md](/home/lizhe/CodeOri/vibe-kanban-overview/tasks/execution-plan.md)
- `tasks/markdown-frontmatter-spec.md`

### 当前最关键的现有后端文件

- [crates/db/src/models/project.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/db/src/models/project.rs)
- [crates/db/src/models/workspace.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/db/src/models/workspace.rs)
- [crates/db/src/models/session.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/db/src/models/session.rs)
- [crates/db/src/models/workspace_repo.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/db/src/models/workspace_repo.rs)
- [crates/db/src/models/scratch.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/db/src/models/scratch.rs)
- [crates/server/src/routes/mod.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/server/src/routes/mod.rs)
- [crates/server/src/routes/workspaces/mod.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/server/src/routes/workspaces/mod.rs)
- [crates/server/src/routes/sessions/mod.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/server/src/routes/sessions/mod.rs)
- [crates/server/src/bin/generate_types.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/server/src/bin/generate_types.rs)

### 当前最关键的现有前端文件

- [packages/web-core/src/shared/providers/WorkspaceProvider.tsx](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/shared/providers/WorkspaceProvider.tsx)
- [packages/web-core/src/shared/hooks/useWorkspaceContext.ts](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/shared/hooks/useWorkspaceContext.ts)
- [packages/web-core/src/shared/lib/routes/appNavigation.ts](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/shared/lib/routes/appNavigation.ts)
- [packages/local-web/src/app/navigation/AppNavigation.ts](/home/lizhe/CodeOri/vibe-kanban-overview/packages/local-web/src/app/navigation/AppNavigation.ts)
- [packages/web-core/src/pages/workspaces/WorkspacesLayout.tsx](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/pages/workspaces/WorkspacesLayout.tsx)
- [packages/web-core/src/pages/workspaces/WorkspacesMainContainer.tsx](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/pages/workspaces/WorkspacesMainContainer.tsx)
- [packages/web-core/src/pages/workspaces/WorkspaceNotesContainer.tsx](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/pages/workspaces/WorkspaceNotesContainer.tsx)
- [packages/web-core/src/shared/lib/api.ts](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/shared/lib/api.ts)
- [packages/web-core/src/shared/hooks/useOpenInEditor.ts](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/shared/hooks/useOpenInEditor.ts)

### 与现有预览/浏览器测试最相关的文件

- [docs/browser-testing.mdx](/home/lizhe/CodeOri/vibe-kanban-overview/docs/browser-testing.mdx)
- [docs/core-features/testing-your-application.mdx](/home/lizhe/CodeOri/vibe-kanban-overview/docs/core-features/testing-your-application.mdx)
- [packages/web-core/src/pages/workspaces/PreviewBrowserContainer.tsx](/home/lizhe/CodeOri/vibe-kanban-overview/packages/web-core/src/pages/workspaces/PreviewBrowserContainer.tsx)
- [packages/ui/src/components/PreviewBrowser.tsx](/home/lizhe/CodeOri/vibe-kanban-overview/packages/ui/src/components/PreviewBrowser.tsx)
- [crates/preview-proxy/src/lib.rs](/home/lizhe/CodeOri/vibe-kanban-overview/crates/preview-proxy/src/lib.rs)

### 预计新增的后端文件

- `crates/db/src/models/branch.rs`
- `crates/db/src/models/session_note.rs`
- `crates/db/src/models/global_note.rs`
- `crates/db/src/models/entity_relation.rs`
- `crates/server/src/routes/branches.rs`
- `crates/server/src/routes/session_notes.rs`
- `crates/server/src/routes/global_notes.rs`
- `crates/server/src/routes/relations.rs`
- `crates/server/src/routes/knowledge_search.rs`
- `crates/server/src/routes/documents.rs`
- `crates/server/src/routes/browser_testing.rs`
- `crates/services/src/services/markdown_store.rs`
- `crates/services/src/services/browser_testing.rs`

### 预计新增的前端文件

- `packages/local-web/src/routes/_app.projects.$projectId_.branches.tsx`
- `packages/local-web/src/routes/_app.projects.$projectId_.branches.create.tsx`
- `packages/local-web/src/routes/_app.projects.$projectId_.branches.$branchId.tsx`
- `packages/local-web/src/routes/_app.projects.$projectId_.sessions.$sessionId.tsx`
- `packages/local-web/src/routes/_app.projects.$projectId_.knowledge.tsx`
- `packages/local-web/src/routes/_app.projects.$projectId_.canvas.tsx`
- `packages/web-core/src/pages/branches/ProjectBranchesPage.tsx`
- `packages/web-core/src/pages/branches/BranchDetailPage.tsx`
- `packages/web-core/src/pages/sessions/SessionDetailPage.tsx`
- `packages/web-core/src/pages/knowledge/KnowledgeWorkbenchPage.tsx`
- `packages/web-core/src/pages/canvas/KnowledgeCanvasPage.tsx`
