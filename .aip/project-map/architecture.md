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
