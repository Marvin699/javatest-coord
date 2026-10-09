# 分析任务包：I-002 部门详情页显示员工数量

> 你（开发侧 AI）的任务：按「分析规则」为下方 Intent 产出一份 Proposal JSON。
> 全部上下文已备齐（需求、历史经验、项目地图），输出要求见文末。

## 1. Intent（原始需求）

```json
{
  "id": "I-002",
  "created_by": "customer-ai",
  "created_at": "2026-10-09T03:01:13.171467Z",
  "title": "部门详情页显示员工数量",
  "description": "管理员打开部门详情页时，希望直接看到该部门当前有多少名员工，不用再去员工列表数。",
  "priority": "medium",
  "raw_quote": "部门详情那个页面能不能显示下这个部门有多少人",
  "customer_analysis": {
    "suspected_modules": [
      "部门详情",
      "员工数据"
    ],
    "suspected_files": [
      "src/main/java/com/hrsystem/controller/admin/DepartmentController.java",
      "src/main/resources/templates/admin/department-detail.html"
    ],
    "rationale": "详情页在 DepartmentController 的 detail/{id}，加个员工计数即可",
    "confidence": 0.8
  },
  "assumptions": [],
  "needs_clarification": [],
  "acceptance_criteria": [],
  "status": "analyzing",
  "history": [
    {
      "at": "2026-10-09T03:01:13.171760Z",
      "from_status": "draft",
      "to_status": "submitted",
      "by": "customer-ai",
      "note": ""
    },
    {
      "at": "2026-10-09T03:01:43.326046Z",
      "from_status": "submitted",
      "to_status": "analyzing",
      "by": "dev-ai",
      "note": ""
    }
  ]
}
```

## 2. 相关项目记忆（历史经验，按相关度排序，可能为空）

### 来自 [员工列表支持按部门筛选]（相关度 0.18）
- 实际工时: 2.5h
- 当时改动: src/main/resources/mapper/EmployeeMapper.xml, src/main/java/com/hrsystem/service/impl/EmployeeServiceImpl.java, src/main/java/com/hrsystem/controller/admin/EmployeeController.java, src/main/resources/templates/admin/employee-list.html
- 踩坑经验: 员工筛选类需求走 Controller→Service→Mapper→XML 四层，改签名要四层同步；查询已有 selectWithDept 片段可复用

## 3. 项目地图（脱敏认知，客户 AI 也在用这份）

### architecture.md

# 项目架构 — JavaTest 人事管理系统

> 由开发侧 AI 依据代码扫描填写（2026-10-09）。只收录代码中真实存在的内容。

## 技术栈

- Java / Spring Boot（入口：`HrSystemApplication.java`）
- MyBatis（XML mapper：`resources/mapper/*.xml`）
- Thymeleaf 服务端渲染（`resources/templates/`）
- Spring Security（`config/SecurityConfig.java`，角色：ADMIN / EMPLOYEE）
- MySQL（建表脚本：`sql/init.sql`，库名 hr_system）

## 模块结构

```
com.hrsystem/
├── controller/
│   ├── LoginController.java          # 登录
│   ├── admin/                        # 管理端页面（需 ADMIN 角色）
│   │   ├── AdminController.java      #   dashboard
│   │   ├── DepartmentController.java #   部门管理
│   │   └── EmployeeController.java   #   员工管理
│   └── employee/EmployeeController.java  # 员工端页面
├── service/（+ impl/）                # Department/Employee/Log/UserDetails
├── mapper/（XML 在 resources/mapper/）# Department/Employee/OperationLog
├── entity/                           # Department/Employee/LoginUser/OperationLog
├── aop/LogAspect.java                # 操作日志切面 → operation_log 表
└── config/SecurityConfig.java        # 安全与权限配置
```

## 关键链路

- **权限**：Spring Security 按角色拦截，ADMIN 走 `/admin/**`，EMPLOYEE 走员工页面
- **审计**：所有操作经 `LogAspect` 切面自动写入 operation_log 表（新增功能需注意是否触发切面）
- **页面流**：Controller 返回 Thymeleaf 模板，无前后端分离、无独立 REST API 层

## 禁区 / 注意事项

- 登录密码字段直接存于 employee 表（`password` 列），改动登录逻辑前先看 SecurityConfig
- mapper 修改必须同步 XML 文件，注解与 XML 混用会失效


### api.json

```json
{
  "apis": [
    { "method": "GET",  "path": "/",                        "desc": "根路径（重定向登录）", "module": "auth" },
    { "method": "GET",  "path": "/login",                   "desc": "登录页", "module": "auth" },
    { "method": "GET",  "path": "/403",                     "desc": "无权限页", "module": "auth" },
    { "method": "GET",  "path": "/admin/dashboard",         "desc": "管理端首页", "module": "admin" },
    { "method": "GET",  "path": "/admin/department/list",   "desc": "部门列表", "module": "department" },
    { "method": "GET",  "path": "/admin/department/detail/{id}", "desc": "部门详情", "module": "department" },
    { "method": "GET",  "path": "/admin/department/add",    "desc": "部门新增表单", "module": "department" },
    { "method": "POST", "path": "/admin/department/add",    "desc": "保存新部门", "module": "department" },
    { "method": "GET",  "path": "/admin/department/edit/{id}", "desc": "部门编辑表单", "module": "department" },
    { "method": "POST", "path": "/admin/department/edit",   "desc": "保存部门修改", "module": "department" },
    { "method": "GET",  "path": "/admin/department/delete/{id}", "desc": "删除部门", "module": "department" },
    { "method": "GET",  "path": "/admin/employee/list",     "desc": "员工列表", "module": "employee" },
    { "method": "GET",  "path": "/admin/employee/search",   "desc": "员工搜索", "module": "employee" },
    { "method": "GET",  "path": "/admin/employee/add",      "desc": "员工新增表单", "module": "employee" },
    { "method": "POST", "path": "/admin/employee/add",      "desc": "保存新员工", "module": "employee" },
    { "method": "GET",  "path": "/admin/employee/edit/{id}", "desc": "员工编辑表单", "module": "employee" },
    { "method": "POST", "path": "/admin/employee/edit",     "desc": "保存员工修改", "module": "employee" },
    { "method": "GET",  "path": "/admin/employee/delete/{id}", "desc": "删除员工", "module": "employee" },
    { "method": "GET",  "path": "/employee/dashboard",      "desc": "员工端首页", "module": "employee" }
  ],
  "_note": "Thymeleaf 服务端渲染，无独立 REST API。以上已逐一核对 Controller 注解（2026-10-09），与代码一致"
}

```

### schema.json

```json
{
  "database": "hr_system（MySQL，utf8mb4）",
  "tables": [
    {
      "name": "department",
      "desc": "部门表",
      "columns": [
        { "name": "id",          "type": "INT AUTO_INCREMENT", "pk": true },
        { "name": "name",        "type": "VARCHAR(50)",  "not_null": true },
        { "name": "code",        "type": "VARCHAR(20)",  "not_null": true, "unique": true },
        { "name": "create_time", "type": "DATETIME", "default": "CURRENT_TIMESTAMP" }
      ]
    },
    {
      "name": "employee",
      "desc": "员工表（兼登录账号：username/password/role）",
      "columns": [
        { "name": "id",            "type": "INT AUTO_INCREMENT", "pk": true },
        { "name": "name",          "type": "VARCHAR(50)", "not_null": true },
        { "name": "emp_no",        "type": "VARCHAR(20)", "not_null": true, "unique": true },
        { "name": "gender",        "type": "VARCHAR(4)",  "not_null": true },
        { "name": "position",      "type": "VARCHAR(50)", "not_null": true },
        { "name": "birth_date",    "type": "DATE" },
        { "name": "department_id", "type": "INT", "not_null": true, "fk": "department.id" },
        { "name": "username",      "type": "VARCHAR(50)", "not_null": true, "unique": true },
        { "name": "password",      "type": "VARCHAR(100)", "not_null": true, "note": "BCrypt" },
        { "name": "role",          "type": "VARCHAR(20)", "not_null": true, "default": "EMPLOYEE", "enum": ["ADMIN", "EMPLOYEE"] },
        { "name": "create_time",   "type": "DATETIME", "default": "CURRENT_TIMESTAMP" }
      ]
    },
    {
      "name": "operation_log",
      "desc": "操作日志表（由 LogAspect 切面自动写入）",
      "columns": [
        { "name": "id",          "type": "INT AUTO_INCREMENT", "pk": true },
        { "name": "username",    "type": "VARCHAR(50)", "not_null": true },
        { "name": "operation",   "type": "VARCHAR(100)", "not_null": true },
        { "name": "method",      "type": "VARCHAR(200)" },
        { "name": "params",      "type": "TEXT" },
        { "name": "ip",          "type": "VARCHAR(50)" },
        { "name": "create_time", "type": "DATETIME", "default": "CURRENT_TIMESTAMP" }
      ]
    }
  ],
  "_note": "依据 sql/init.sql 逐列录入，未做字段增删推断"
}

```

## 4. 分析规则（ANALYST.md，必须遵守）

# AIP 项目宪法（开发侧 AI 分析规则）

> **Version:** 1.0 · **Ratified:** 2026-09-24 · **Governance:** 修改任一 NON-NEGOTIABLE 条款需项目所有者批准，并在本文件头部更新版本号。
>
> 你（开发侧 AI）收到一条待分析的 Intent。本宪法是流程契约：**每一条都必须遵守，标 NON-NEGOTIABLE 的条款无任何例外。**

## 第一章 线索与假设（NON-NEGOTIABLE）

- **A1.** 客户线索（`customer_analysis`）与客户假设（`assumptions`）是**待验证假设，不是答案**。必须逐条对照真实代码独立核实，把结论写进 Proposal 的 `clue_verification`（对上了 / 偏了 / 不相关 + 一句说明）。
- **A2.** 线索错了照常给正确方案。客户 AI 的猜测只影响你从哪开始读代码，不影响结论正确性。
- **A3.** 若 Intent 带 `needs_clarification` 且问题足以阻塞分析，流转 `blocked` 并在 note 写清缺什么。

## 第二章 证据（NON-NEGOTIABLE）

- **B1.** 方案里只写有证据的结论。`files_to_change` 与 `tasks[].files` 必须是你在代码库里**真实看到并读过**的路径，禁止推测补全。
- **B2.** 拿不准的判断全部写进 `risks`，不要硬给结论。宁可多写一条风险，不可少一份证据。

## 第三章 上下文

- **C1.** 分析前必读三样：Intent 全文（含 `raw_quote` 客户原话）、项目地图（`project-map/`）、同主题历史记忆（任务包已自动附上）。
- **C2.** 历史经验（`memory/`）是参考不是圣旨——上次的方案这次未必适用，但上次的坑必须避开。

## 第四章 任务拆解

- **D1.** **one task = one context window = one PR。** 每个任务（`tasks[]`）必须小到能在一个上下文窗口内完成、能独立成一个 PR。
- **D2.** 每个任务写明涉及的真实文件路径；可并行的任务标 `parallel: true`。
- **D3.** 涉及多阶段（如先动库再改代码）时，按依赖排序编号（T001、T002…），后续任务不得引用未完成前置任务的产物。

## 第五章 轻量（NON-NEGOTIABLE）

- **E1.** 方案保持瘦：summary 一句话，risks 只列真实风险。**不要把方案写成文档**——评审者是忙着看代码的人类，不是审计散文的律师。
- **E2.** 需求变化 = 新 Intent（走 rejected → submitted 重新提交），不要偷改已提交的需求；只有 >50% 重叠的同类需求才允许在原方案上更新。

## 第六章 流程

1. 读任务包（`.aip/tasks/<id>-task.md`，已含 Intent + 记忆 + 地图）——或手动 `aip analyze <id>`
2. 按第一~五章核实与拆解，产出 Proposal JSON（结构见任务包文末的输出要求）
3. `aip propose <id> --file <proposal.json>` 落盘并流转 `proposal_ready`
4. 向开发者口头汇报：方案摘要 + 风险 + 建议（做 / 不做 / 换方案）
5. 获批（`aip approve <id>`）后实现；完成后 `aip done <id> --effort 实际工时 --lessons 踩坑经验`
   —— outcome 自动沉淀进项目记忆；**若实际改动影响了模块/接口/表结构，必须同步更新 `project-map/` 三件套**（这是 done 的隐性验收标准）

## 附：Proposal 字段速查

```json
{
  "intent_id": "<id>",
  "summary": "一句话方案",
  "files_to_change": ["真实存在的文件路径"],
  "tasks": [{"id": "T001", "title": "任务标题", "files": ["真实路径"], "parallel": false}],
  "new_apis": ["需要新增或修改的 API"],
  "db_changes": "none | minor | major",
  "db_changes_detail": "若动数据库，写明改哪些表/字段",
  "estimated_effort": "预估工时，如 4h",
  "risks": ["风险点，包括不确定的部分"],
  "clue_verification": "对客户线索/假设的核实结论"
}
```


## 5. 输出要求

- 只输出一个 JSON，不要输出其他内容
- files_to_change 必须是你在项目里核实过的真实路径；地图没写的就以读到的代码为准
- 拿不准的判断全部放进 risks，宁可多写

```json
{
  "intent_id": "<Intent 编号>",
  "summary": "一句话方案",
  "files_to_change": ["必须是你确认存在的真实路径"],
  "tasks": [
    {"id": "T001", "title": "任务标题", "files": ["真实路径"], "parallel": false}
  ],
  "new_apis": ["需要新增/修改的 API"],
  "db_changes": "none | minor | major",
  "db_changes_detail": "动库说明，none 时留空",
  "estimated_effort": "如 4h",
  "risks": ["拿不准的全部写这里"],
  "clue_verification": "对客户线索/假设的核实结论：对上了/偏了/不相关 + 说明"
}
```
