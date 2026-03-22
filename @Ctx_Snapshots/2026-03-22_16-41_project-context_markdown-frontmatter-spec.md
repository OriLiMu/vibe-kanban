# Vibe-Kanban 项目上下文快照

- 记录时间：2026-03-22 16:41 +0800
- 记录目的：作为新会话继续 `markdown-frontmatter-spec` 的上下文起点
- 仓库路径：`/home/lizhe/CodeOri/vibe-kanban`

## 1. 当前 Git 状态

- 当前分支：`feat/markdown-frontmatter-spec`
- 跟踪分支：`origin/feat/markdown-frontmatter-spec`
- 当前工作树状态：干净，无未提交改动
- 当前唯一 worktree：`/home/lizhe/CodeOri/vibe-kanban`
- 之前用于 `feat/notes-root-config` 的 worktree `/home/lizhe/CodeOri/vibe-kanban-overview` 已删除
- `main` 与 `origin/main` 已同步到同一提交
- `feat/markdown-frontmatter-spec` 与 `origin/feat/markdown-frontmatter-spec` 已同步到同一提交

### 当前关键提交

- 当前 `HEAD` / `main` / `origin/main` / `origin/feat/markdown-frontmatter-spec`：
  - `3543944f` `add notes root directory config`
- 上一个关键提交：
  - `7e1e9334` `merge codex/codebase-overview`
- 规划文档相关提交：
  - `25203dbc` `document knowledge overhaul execution plan`

## 2. 已完成的关键工作

### 2.1 知识体系重构方案已定稿

执行计划已经明确以下核心方向：

- 产品主线从当前执行型 `workspace` 中心，转向知识推进型结构：
  - `project -> branch -> session -> note`
- `workspace` 继续保留为底层运行时概念，但不直接暴露在 UI
- `global note` 为跨 session 的独立知识对象
- `branch / session / note / global note` 的正文统一存为仓库外 Markdown 文件
- 产品内正文只做只读预览，编辑交给外部编辑器
- 关系模型第一版固定为：
  - `来源于`
  - `相关`
  - `补充`
  - `依赖`
  - `冲突`
  - `启发`

对应文档：

- `tasks/execution-plan.md`
- `tasks/todo.md`

### 2.2 `feat/notes-root-config` 已完成并已合入 `main`

该任务已经开发、验证、合并、推送完成，feature 分支和远端分支都已删除。

这次完成的内容：

- 配置系统升级到 `v9`
- 新增配置字段 `notes_root_dir`
- 当用户未配置 `notes_root_dir` 时，默认解析为应用数据目录下的 `notes`
- 启动时会预创建解析后的笔记根目录
- 更新配置时会先校验并创建该目录，避免无效路径进入配置
- 设置页新增“笔记根目录”输入框和目录选择入口
- 多语言设置文案已补齐
- 共享 TS 类型已重新生成

关键文件：

- `crates/services/src/services/config/versions/v9.rs`
- `crates/services/src/services/config/mod.rs`
- `crates/local-deployment/src/lib.rs`
- `crates/server/src/routes/config.rs`
- `packages/web-core/src/shared/dialogs/settings/settings/GeneralSettingsSection.tsx`
- `packages/web-core/src/i18n/locales/*/settings.json`
- `shared/types.ts`

## 3. 已完成验证

### 3.1 在 `feat/notes-root-config` 上已完成的验证

- `pnpm run generate-types`
  - 通过
- `cargo test -p services notes_root_dir`
  - 通过，3 个测试全部成功
- `git diff --check`
  - 通过

### 3.2 格式化相关的真实情况

- Rust 格式化：
  - `cargo fmt --all` 已执行
- 全仓 `pnpm run format` 未完全通过
  - 失败原因不是代码错误，而是本地缺少前端依赖中的 `prettier`
  - 报错本质：`packages/web-core` 目录下没有安装 `node_modules`
- 为完成本次实际变更的格式化，已经使用：
  - `pnpm dlx prettier@3.6.2 ...`

这意味着后续继续前端相关任务时，需要记住：

- 当前环境不是完整安装态
- 直接跑全仓前端格式化/某些 pnpm 子命令时，可能因为缺少本地依赖失败

## 4. 当前任务看板状态

`tasks/todo.md` 中，以下事项已经完成：

- 记录执行计划与相关文件清单到 `tasks/execution-plan.md`
- 新增笔记根目录配置项与默认路径策略
- `feat/notes-root-config`：定义笔记根目录默认路径策略与配置命名
- `feat/notes-root-config`：扩展 Config 版本迁移与共享类型
- `feat/notes-root-config`：在设置页增加笔记根目录选择入口与文案
- `feat/notes-root-config`：验证配置保存、类型生成与格式化

当前排在最前面的未完成任务是：

- `明确 Markdown 文件命名规则与 frontmatter 规范`

后续紧跟的高优先级任务是：

- `设计 projects 单仓库化改造方案`
- `设计 branches 表字段与约束`

## 5. 下一会话应直接开始的任务

- 分支名：`feat/markdown-frontmatter-spec`
- 当前状态：已创建并已推送远端，但尚未开始写代码或文档实现
- 目标任务：`明确 Markdown 文件命名规则与 frontmatter 规范`

### 5.1 这个任务建议解决的问题

新会话启动后，建议优先定稿以下内容：

- Markdown 文件的命名规则
  - `branch`
  - `session`
  - `note`
  - `global note`
- 是否使用稳定 ID、slug、时间戳，或三者组合
- 文件名冲突规避策略
- 项目级目录平铺下的分类前缀策略
- frontmatter 的公共字段
- frontmatter 的实体专属字段
- frontmatter 中数据库关系与文件系统关系的职责边界
- 哪些字段以数据库为准，哪些字段只用于文件侧冗余描述或索引

### 5.2 这个任务的输入约束

来自既有设计文档的约束：

- Markdown 文件放在仓库外
- 每个 `project` 一个文件夹
- `project` 文件夹内部采用平铺结构
- 层级与关系通过数据库和 frontmatter 管理
- 第一版产品内不做正文编辑，只做只读预览

这意味着下一任务不要把目录层级设计成：

- `project/branch/session/note/...`

因为当前计划已经明确：

- 每个 `project` 一个文件夹
- 文件夹内部采用平铺结构

## 6. 下一任务可直接参考的关键文件

### 6.1 计划与任务文件

- `tasks/execution-plan.md`
- `tasks/todo.md`
- `tasks/lessons.md`

### 6.2 现有配置与设置相关文件

- `crates/services/src/services/config/versions/v9.rs`
- `crates/services/src/services/config/mod.rs`
- `crates/server/src/routes/config.rs`
- `crates/local-deployment/src/lib.rs`
- `packages/web-core/src/shared/dialogs/settings/settings/GeneralSettingsSection.tsx`
- `shared/types.ts`

### 6.3 下一任务更可能会触达的文件

以下文件当前尚未实现 Markdown 存储服务，但很可能会在 `markdown-frontmatter-spec` 阶段被讨论或新增：

- `tasks/execution-plan.md`
- `crates/services/src/services/markdown_store.rs`（计划新增）
- `crates/server/src/routes/documents.rs`（计划新增）
- `packages/web-core/src/shared/components/SimpleMarkdown.tsx`
- `packages/web-core/src/shared/hooks/useOpenInEditor.ts`

### 6.4 可复用的现有 frontmatter 解析线索

当前仓库里，已存在一处 frontmatter 解析逻辑，可作为实现风格参考：

- `crates/executors/src/executors/claude/slash_commands.rs`

它已经有最小 frontmatter 读取能力，说明：

- 仓库内并不是完全没有 frontmatter 处理基础
- 下一任务可以优先统一 frontmatter 规范，再决定是否抽出公共解析工具

## 7. 重要环境事实

- 当前不存在 `@Ctx_Snapshots/` 目录历史内容；本文件是首次创建
- 当前不存在额外 worktree
- 本地和远端已经收敛一致
- `feat/notes-root-config` 已删除，无需再回到该分支
- `feat/markdown-frontmatter-spec` 当前只是新起点，不包含该任务本身的实现

## 8. 建议的新会话起手动作

建议新会话按下面顺序启动：

1. 读取本文件
2. 读取 `tasks/todo.md`
3. 读取 `tasks/execution-plan.md`
4. 明确 Markdown 文件命名规则草案
5. 明确 frontmatter 字段草案
6. 将规范写回计划或新增设计文档
7. 再决定是否直接进入 `markdown_store` 服务骨架实现

## 9. 一句话总结

当前项目已经完成知识体系重构的总体方案和笔记根目录配置闭环，`main` 与远端一致，下一步应在 `feat/markdown-frontmatter-spec` 上优先定稿 Markdown 文件命名规则与 frontmatter 规范。
