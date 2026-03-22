# Markdown 文件命名规则与 Frontmatter 规范

## 1. 目标

本规范用于定稿第一版知识体系中文档文件的落盘规则，覆盖以下四类实体：

- `branch`
- `session`
- `note`
- `global_note`

目标是同时满足下面几件事：

- 文件路径稳定，不因标题修改频繁重命名
- 项目目录内部保持平铺，不引入 `project/branch/session/note` 多层目录
- 外部编辑器中可读、可搜索、可人工排查
- 后续 `markdown_store` 服务可以低成本实现创建、解析、同步与修复
- 明确数据库与 frontmatter 的职责边界，避免双写冲突

## 2. 已知约束

- 所有正文文件都放在仓库外的 `notes_root_dir`
- 每个 `project` 对应 `notes_root_dir` 下的一个独立目录
- `project` 目录内部采用平铺结构
- 层级关系与关系图谱不靠目录层级表达，而靠数据库与 frontmatter 表达
- 第一版产品内只做只读预览，正文编辑交给外部编辑器
- 当前仓库主流稳定 ID 为 `UUID v4`

## 3. 路径与文件命名规则

### 3.1 目录结构

统一目录结构：

```text
{notes_root_dir}/
  project--{project_slug}--{project_id}/
    branch--{created_at_compact}--{slug}--{entity_id}.md
    session--{created_at_compact}--{slug}--{entity_id}.md
    note--{created_at_compact}--{slug}--{entity_id}.md
    global-note--{created_at_compact}--{slug}--{entity_id}.md
```

说明：

- `project_slug` 来源于 `project.name` 的首次创建快照
- `project_id` 使用数据库中的完整 UUID
- `project` 目录名创建后不因 project 改名而自动重命名
- 路径的“稳定锚点”是 `project_id` 与 `entity_id`，不是 slug

### 3.2 project 目录命名

固定格式：

```text
project--{project_slug}--{project_id}
```

示例：

```text
project--vibe-kanban--8c5a3d4b-6d61-4e1d-9fd3-9d93f3bfa1f1
```

选择这个格式的原因：

- 目录名直接可读，便于人在 Finder、Explorer、终端里识别
- 同名 project 也不会冲突，因为 `project_id` 保证唯一
- project 改名后无需搬迁整目录，避免路径抖动

### 3.3 文件命名格式

固定格式：

```text
{entity_prefix}--{created_at_compact}--{slug}--{entity_id}.md
```

其中：

- `entity_prefix`
  - `branch`
  - `session`
  - `note`
  - `global-note`
- `created_at_compact`
  - 使用 UTC 时间
  - 格式固定为 `YYYYMMDDTHHmmssZ`
  - 例：`20260322T084100Z`
- `slug`
  - 来自标题或分支名的创建时快照
  - 只用于可读性，不作为定位依据
- `entity_id`
  - 使用数据库中的完整 UUID

示例：

```text
branch--20260322T084100Z--feat-markdown-frontmatter-spec--550e8400-e29b-41d4-a716-446655440000.md
session--20260322T090500Z--frontmatter-boundary-analysis--9d8cb1a2-0d2b-4baf-8c80-2c4f51d91f27.md
note--20260322T091200Z--db-vs-file-authority--f3f2b8d2-e097-45db-a022-0f4b68ff7584.md
global-note--20260322T093000Z--markdown-metadata-schema--a2e6ef0a-f267-4b5e-98cc-6bd5a9f7933e.md
```

### 3.4 slug 生成规则

统一规则：

- 输入来源
  - `branch`：`git_branch_name`
  - 其他实体：`title`
- 转为小写 ASCII
- 空白、下划线、斜杠与连续标点统一折叠为单个 `-`
- 去掉首尾 `-`
- 最大长度 48 个字符
- 清洗后为空时使用 `untitled`

补充约束：

- slug 只在文件创建时生成一次
- 标题后续变更不会触发自动改名
- 文件真实定位依赖数据库记录的 `file_path` 与 frontmatter 中的 `entity_id`

这样设计的原因：

- 避免“改一个标题导致路径变化、外部索引失效、同步器误判新文件”
- 保留足够的人类可读性
- 后续若要支持文件重命名，可以作为显式迁移动作，而不是默认副作用

### 3.5 冲突规避策略

- 正常情况下不依赖 slug 去重，因为 `entity_id` 已保证唯一
- 如果同一 `entity_id` 的目标路径已存在，则视为同一实体已落盘，不再生成新文件
- 如果数据库记录的 `file_path` 与磁盘实际文件名不一致，以数据库记录为准；修复逻辑在后续 `markdown_store` 中补
- 第一版不支持用户通过手工改文件名来“重命名实体”

### 3.6 为什么不用纯 slug 或纯时间戳

- 不用纯 slug：
  - 标题重复时必然冲突
  - 标题变更会导致路径不稳定
- 不用纯时间戳：
  - 人无法快速判断文件语义
- 不用只有 UUID：
  - 稳定但可读性太差，排查成本高

因此 v1 采用：

- 类型前缀
- 创建时间快照
- 可读 slug
- 完整 UUID

四者组合。

## 4. Frontmatter 总体规则

### 4.1 格式

- 使用 YAML frontmatter
- 必须位于文件开头，以 `---` 开始、以 `---` 结束
- frontmatter 后保留一个空行，再进入 Markdown 正文
- 顶层字段统一使用 `snake_case`
- v1 不使用嵌套对象
- v1 只允许以下值类型：
  - 字符串
  - 字符串数组
- 可选字段为空时直接省略，不写 `null`

选择“扁平字段 + 少类型”的原因：

- Rust 解析实现更简单
- 与当前仓库的 `snake_case` 风格一致
- 降低手工编辑出错率
- 减少 YAML 嵌套结构带来的 merge 噪音

### 4.2 公共字段

所有四类文档都必须包含以下字段：

| 字段          | 类型     | 必填 | authority            | 说明                                          |
| ------------- | -------- | ---- | -------------------- | --------------------------------------------- |
| `vk_schema`   | `string` | 是   | 数据库写入，文件镜像 | 固定值：`vk-markdown/v1`                      |
| `entity_type` | `string` | 是   | 数据库写入，文件镜像 | `branch` / `session` / `note` / `global_note` |
| `entity_id`   | `string` | 是   | 数据库               | 实体 UUID，不允许手工改                       |
| `project_id`  | `string` | 是   | 数据库               | 所属 project UUID，不允许手工改               |
| `title`       | `string` | 是   | 文件                 | 文档显示标题，允许外部编辑                    |
| `slug`        | `string` | 是   | 系统生成，文件镜像   | 创建时生成，不随标题自动刷新                  |
| `created_at`  | `string` | 是   | 数据库               | ISO 8601 UTC 时间                             |
| `updated_at`  | `string` | 是   | 系统同步             | 最近一次成功同步时间                          |

补充说明：

- `title` 是面向用户的主显示名称
- `slug` 不是主键，不参与关系计算
- `updated_at` 不是纯文件 mtime，而是系统确认并同步后的业务时间

### 4.3 branch 专属字段

| 字段                | 类型       | 必填 | authority | 说明                                                                    |
| ------------------- | ---------- | ---- | --------- | ----------------------------------------------------------------------- |
| `git_branch_name`   | `string`   | 是   | 数据库    | 对应真实 Git branch 名称                                                |
| `status`            | `string`   | 是   | 数据库    | `in_progress` / `blocked` / `needs_deep_dive` / `resolved` / `archived` |
| `purpose`           | `string`   | 是   | 文件      | 本次 branch 目的                                                        |
| `scope`             | `string`   | 是   | 文件      | 本次涉及范围                                                            |
| `prerequisites`     | `string[]` | 是   | 文件      | 前置条件列表                                                            |
| `expected_outcomes` | `string[]` | 是   | 文件      | 预期结果列表                                                            |

不放进 frontmatter 的字段：

- `workspace_id`
- 阻塞关系
- 派生关系
- PR 关联

这些都属于运行时或图关系数据，应该留在数据库里。

### 4.4 session 专属字段

| 字段                | 类型       | 必填 | authority | 说明                           |
| ------------------- | ---------- | ---- | --------- | ------------------------------ |
| `branch_id`         | `string`   | 是   | 数据库    | 所属 branch UUID，不允许手工改 |
| `purpose`           | `string`   | 是   | 文件      | 本次 session 要解决的问题      |
| `scope`             | `string`   | 是   | 文件      | 本次 session 涉及范围          |
| `prerequisites`     | `string[]` | 是   | 文件      | 前置条件列表                   |
| `expected_outcomes` | `string[]` | 是   | 文件      | 预期结果列表                   |

说明：

- v1 不给 `session` 单独定义状态字段
- session 与 branch 的层级关系通过 `branch_id` 明确表达，不靠目录层级

### 4.5 note 专属字段

| 字段         | 类型       | 必填 | authority | 说明              |
| ------------ | ---------- | ---- | --------- | ----------------- |
| `branch_id`  | `string`   | 是   | 数据库    | 所属 branch UUID  |
| `session_id` | `string`   | 是   | 数据库    | 所属 session UUID |
| `tags`       | `string[]` | 否   | 文件      | note 的轻量标签   |

说明：

- `note` 的正文就是核心内容，因此不再额外定义 `summary`
- `tags` 允许用户在外部编辑器中维护，数据库保存索引副本用于筛选与搜索

### 4.6 global_note 专属字段

| 字段              | 类型       | 必填 | authority | 说明                              |
| ----------------- | ---------- | ---- | --------- | --------------------------------- |
| `source_note_ids` | `string[]` | 否   | 数据库    | 来源 note UUID 列表，文件中做镜像 |
| `tags`            | `string[]` | 否   | 文件      | global note 标签                  |

说明：

- v1 不给 `global_note` 定义状态字段
- `source_note_ids` 用于增强文件自解释性，但真正的来源关系仍以数据库映射表为准

## 5. 数据库与 frontmatter 的职责边界

### 5.1 数据库 authoritative

以下信息统一以数据库为准：

- 实体身份
  - `entity_id`
  - `entity_type`
  - `project_id`
  - `branch_id`
  - `session_id`
- 运行时映射
  - `workspace_id`
  - 真实 `file_path`
  - Git branch 与 repo 映射
- 状态与流程
  - `branch.status`
  - 阻塞/解阻
  - 派生/回流
  - relation 图谱
- 来源与关联
  - `global_note_sources`
  - `entity_relations`
- 检索与排序
  - 索引字段
  - 搜索向量
  - 列表查询排序字段

原因：

- 这些字段需要强约束、索引、事务一致性
- 这些字段经常参与列表过滤、关系遍历和后端逻辑
- 它们不适合作为“任意可手改 YAML”来做唯一真源

### 5.2 文件 authoritative

以下信息以文件为准，数据库保存同步后的副本：

- `title`
- `purpose`
- `scope`
- `prerequisites`
- `expected_outcomes`
- `tags`
- Markdown 正文

原因：

- 这些字段属于文档内容的一部分
- 它们天然适合在外部编辑器中维护
- 未来若引入 Git 历史或文件同步，这些字段也更适合作为文档真源

### 5.3 文件中的系统镜像字段

以下字段写入 frontmatter，但不允许用户把它们当成可自由编辑字段：

- `vk_schema`
- `entity_type`
- `entity_id`
- `project_id`
- `branch_id`
- `session_id`
- `source_note_ids`
- `git_branch_name`
- `status`
- `slug`
- `created_at`
- `updated_at`

这些字段存在于文件中的目的：

- 让单个 Markdown 文件脱离数据库时仍然具备最小自解释性
- 便于未来做导入、修复、重建索引
- 便于人工排查“这份文件是谁、属于谁、来自哪里”

但它们不是 v1 的外部编辑入口。

## 6. 同步规则

创建文档时：

1. 先在数据库创建实体与关系记录
2. 生成 project 目录
3. 生成规范文件名
4. 写入 frontmatter 初始快照与空正文模板
5. 将相对路径写回数据库

读取文档时：

1. 以数据库中的 `file_path` 定位文件
2. 解析 frontmatter
3. 使用文件字段更新文档内容型元数据
4. 忽略或覆盖非法的系统镜像字段修改

外部编辑时：

- 允许修改正文
- 允许修改 `title`
- 允许修改 `purpose`
- 允许修改 `scope`
- 允许修改 `prerequisites`
- 允许修改 `expected_outcomes`
- 允许修改 `tags`
- 不支持把“手动改文件名”当作正式重命名机制
- 不支持手动改 `entity_id`、`project_id`、`branch_id`、`session_id`、`status`

## 7. v1 明确不做的事情

- 不通过目录层级表达父子关系
- 不在 frontmatter 中直接存整张关系图
- 不在 frontmatter 中暴露 `workspace_id`
- 不让 slug 或标题成为定位主键
- 不因为标题变化自动重命名文件
- 不支持任意手工改系统字段后自动迁移数据库

## 8. Frontmatter 示例

### 8.1 branch

```yaml
---
vk_schema: vk-markdown/v1
entity_type: branch
entity_id: 550e8400-e29b-41d4-a716-446655440000
project_id: 8c5a3d4b-6d61-4e1d-9fd3-9d93f3bfa1f1
title: 明确 Markdown 文件命名规则与 frontmatter 规范
slug: feat-markdown-frontmatter-spec
created_at: 2026-03-22T08:41:00Z
updated_at: 2026-03-22T09:45:00Z
git_branch_name: feat/markdown-frontmatter-spec
status: in_progress
purpose: 固化知识文档文件的稳定命名与元数据边界
scope: 覆盖 branch、session、note、global note 的 Markdown 规范
prerequisites:
  - 已完成 notes_root_dir 配置
  - 已确认 project 目录采用平铺结构
expected_outcomes:
  - 产出可直接实现的规格文档
  - 为后续 markdown_store 服务提供输入
---
```

### 8.2 session

```yaml
---
vk_schema: vk-markdown/v1
entity_type: session
entity_id: 9d8cb1a2-0d2b-4baf-8c80-2c4f51d91f27
project_id: 8c5a3d4b-6d61-4e1d-9fd3-9d93f3bfa1f1
title: 分析 frontmatter 与数据库的职责边界
slug: frontmatter-boundary-analysis
created_at: 2026-03-22T09:05:00Z
updated_at: 2026-03-22T09:30:00Z
branch_id: 550e8400-e29b-41d4-a716-446655440000
purpose: 决定哪些字段应该可被外部编辑
scope: 仅讨论 v1 同步边界，不展开实现细节
prerequisites:
  - 已有 branch 规格草案
expected_outcomes:
  - 确定数据库 authoritative 字段
  - 确定 frontmatter 可编辑字段
---
```

### 8.3 note

```yaml
---
vk_schema: vk-markdown/v1
entity_type: note
entity_id: f3f2b8d2-e097-45db-a022-0f4b68ff7584
project_id: 8c5a3d4b-6d61-4e1d-9fd3-9d93f3bfa1f1
title: DB 与文件 authority 划分
slug: db-vs-file-authority
created_at: 2026-03-22T09:12:00Z
updated_at: 2026-03-22T09:20:00Z
branch_id: 550e8400-e29b-41d4-a716-446655440000
session_id: 9d8cb1a2-0d2b-4baf-8c80-2c4f51d91f27
tags:
  - spec
  - sync
---
```

### 8.4 global_note

```yaml
---
vk_schema: vk-markdown/v1
entity_type: global_note
entity_id: a2e6ef0a-f267-4b5e-98cc-6bd5a9f7933e
project_id: 8c5a3d4b-6d61-4e1d-9fd3-9d93f3bfa1f1
title: Markdown 元数据设计原则
slug: markdown-metadata-schema
created_at: 2026-03-22T09:30:00Z
updated_at: 2026-03-22T10:00:00Z
source_note_ids:
  - f3f2b8d2-e097-45db-a022-0f4b68ff7584
tags:
  - knowledge
  - markdown
---
```

## 9. 后续实现建议

本规范定稿后，后续实现顺序建议如下：

1. 新增 `markdown_store` 服务，先只支持“按 DB 路径读写单文件”
2. 为公共 frontmatter 定义 Rust 结构体与 `entity_type` 分派
3. 先实现“系统字段校验 + 内容字段同步”，暂不做自动迁移
4. 再补“文件缺失修复”“前后端预览接口”“打开外部编辑器”
