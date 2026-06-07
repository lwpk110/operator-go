---
name: operator-go-engineer
description: 使用 operator-go 与 Kubebuilder 构建标准化 Kubernetes Operator，覆盖 CRD 设计、GenericReconciler 装配、RoleGroupHandler 实现、Webhook 校验、测试与工程化交付。
license: MIT
metadata:
  author: https://github.com/lwpk110
  version: "1.0.0"
  domain: kubernetes
  role: specialist
  scope: implementation
  output-format: code
  triggers: operator-go, kubebuilder, kubernetes operator, generic reconciler, rolegrouphandler
---

# operator-go Engineer

## Core Workflow

1. **澄清需求** — 确认产品角色模型、配置边界、状态语义、扩缩容策略
2. **设计结构** — 规划 CRD、controller 装配层、handlers、webhook、extension 目录与职责
3. **实现调和链路** — 用 `GenericReconciler` + `RoleGroupHandler` 构建标准化调和流程
4. **完善校验与默认值** — 在 webhook 层处理输入约束，避免运行期失败
5. **补齐测试** — 单测覆盖映射与路由，集成测试覆盖 reconcile 生命周期与 status 更新
6. **执行验证** — 运行 `make generate fmt vet test lint`，确认可持续集成可通过

## Reference Guide

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Reconcile 框架 | `pkg/reconciler` | 需要统一调和生命周期与状态推进 |
| 资源构建 | `pkg/builder` | 需要构建 StatefulSet / Service / ConfigMap / PDB |
| 扩展机制 | `pkg/common` | 需要注册扩展点或复用扩展模式 |
| Webhook | `pkg/webhook` | 需要默认值填充、输入校验、准入控制 |
| 测试基建 | `pkg/testutil` | 需要 envtest、mock、matcher 进行单测和集成测试 |

## Quick Start — 推荐最小结构

```text
<product>-operator/
├── api/v1alpha1/
├── cmd/main.go
├── config/
├── internal/
│   ├── controller/
│   ├── handlers/
│   ├── extensions/
│   ├── webhook/
│   └── config/
└── test/
```

## Constraints

### MUST DO

| Rule | Correct Pattern |
|------|-----------------|
| 控制器职责 | controller 仅做装配，不承载业务调和细节 |
| 调和入口 | 使用 `GenericReconcilerConfig`，并设置 `Client`、`Scheme`、`Recorder`、`RoleGroupHandler`、`Prototype` |
| CRD 抽象 | CRD 必须实现 `ClusterInterface`（`GetSpec`/`GetStatus`/`SetStatus`/`DeepCopyCluster` 等） |
| Role 解耦 | 使用“路由 Handler + 每个 Role 独立 Handler” |
| 状态一致性 | 失败标记 `Degraded`；成功写入 `ReconcileComplete` 与 `ObservedGeneration` |
| 限流处理 | 遇到 Kubernetes API Server 限流（HTTP 429）时通过 `RequeueAfter` 退避 |
| 测试覆盖 | 单测覆盖映射与路由；集成测试覆盖 reconcile 生命周期 |

### MUST NOT DO

- 手写一套平行于 `GenericReconciler` 的主调和流程
- 将产品业务逻辑堆入 `cmd/main.go`
- 将输入校验延迟到运行期调和再处理
- 无序覆盖或跳步更新 Status 条件
- 在 `BuildResources()` 中混入调和流程编排逻辑

## Scenarios

### 场景 A：创建新 Operator

1. 使用 Kubebuilder 初始化并创建 API
2. 定义 CRD 的 Spec/Status，并实现 `ClusterInterface`
3. 实现 Role 分层的 `RoleGroupHandler`
4. 在 `cmd/main.go` 装配并注册 `GenericReconciler`
5. 增加 webhook 默认值与校验逻辑
6. 接入 extension（如有需要）
7. 补齐 sample、单测与集成测试
8. 执行 `make generate fmt vet test lint`

### 场景 B：新增 Role

1. 扩展 Spec 字段
2. 在 `GetSpec()` 中映射到 `GenericClusterSpec.Roles`
3. 新增对应 Role handler
4. 在路由 handler 中接入新 Role
5. 更新 webhook 的默认值与校验规则
6. 补充单测与集成测试

## Do / Don’t

### Do

- 使用 `GenericReconciler` 固化生命周期
- 将产品逻辑聚焦在 `RoleGroupHandler` / handlers
- 使用 webhook 完成默认值和输入校验
- 使用 `testutil` 统一测试基建

### Don’t

- 不要绕过 operator-go 核心抽象自行拼装主流程
- 不要在 controller 层直接写资源构建细节
- 不要忽略 `GenericClusterSpec.ClusterOperation`（类型为 `ClusterOperationSpec`，如 `ReconciliationPaused`、`Stopped`）语义
- 不要只做 happy path，缺少错误与状态回归验证
