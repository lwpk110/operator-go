# Extensions & Config Generation

## 1) Extension 使用策略

在以下场景优先使用 extension，而不是侵入 role handler：

- 跨 Role 的共性行为（健康检查、标签/注解策略）
- 生命周期 Hook（Pre/Post Reconcile）
- 产品能力插件化（如目录、连接器、Sidecar 管理）

## 2) 注册模式

在 `cmd/main.go` 统一注册：

- `RegisterClusterExtension(...)`
- `RegisterRoleExtension(...)`

要求：

- 注册点集中管理
- 扩展初始化失败应阻断启动

## 3) 配置生成边界

配置模块（`internal/config`）负责：

- 从 Spec 与 RoleGroup 上下文生成配置文本
- 输出多文件配置（properties/yaml/env 等）
- 支持模板化与可测试构建

handler 负责：

- 将配置写入 ConfigMap 并挂载
- 连接配置与工作负载资源

## 4) 可维护性建议

- 每种配置格式独立模块
- 复杂配置生成必须有单测
- 插件新增应尽量不改动已有 role handler 主路径
