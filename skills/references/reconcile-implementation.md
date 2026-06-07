# Reconcile Implementation Guide

## 1) CRD 设计与 `ClusterInterface`

实现要点：

- Spec 使用产品可读字段（例如 `coordinators`、`workers`）
- `GetSpec()` 在运行时映射到 `GenericClusterSpec.Roles`
- `GetStatus()` / `SetStatus()` 对接 `GenericClusterStatus`
- `DeepCopyCluster()`、`GetRuntimeObject()`、`GetObjectMeta()` 按接口补齐

关键目标：

- 对用户保持产品语义
- 对框架保持通用语义

## 2) `GenericReconcilerConfig` 必备项

必须设置：

- `Client`
- `Scheme`
- `Recorder`
- `RoleGroupHandler`
- `Prototype`

可按需设置：

- `HealthCheckInterval`
- `HealthCheckTimeout`

## 3) RoleGroupHandler 组织方式

推荐模式：

1. Controller 层做 role 路由（按 `buildCtx.RoleName`）
2. 每个 role 由独立 handler 负责资源构建
3. handler 返回 `RoleGroupResources`，不负责主流程编排

这样可以：

- 降低 role 间耦合
- 便于新增角色时最小变更
- 让测试可以按 role 维度隔离

## 4) 资源构建顺序建议

在 RoleGroup 内保持稳定输出：

1. ConfigMap
2. Headless Service
3. Service（如需要）
4. StatefulSet
5. PodDisruptionBudget

注意：

- 只在 handler 中处理资源定义与关联关系
- 不在 handler 中更新全局状态条件

## 5) 错误与重试策略

- API Server 限流/暂时错误：使用 `RequeueAfter`
- 不可恢复配置错误：让 webhook 前置拦截
- 业务错误需可观测并触发 `Degraded` 语义

## 6) 新增 Role 的标准动作

1. 扩展 Spec 字段
2. 在 `GetSpec()` 增加 Roles 映射
3. 新建 Role handler
4. controller 路由接入
5. webhook 默认值/校验同步更新
6. 补充 role 对应单测与集成测试
