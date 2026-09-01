---
name: operator-go-engineer
description: 基于 operator-go 构建可复用 Kubernetes Operator，面向新产品快速落地 CRD、调和链路、Webhook、扩展与测试交付。
license: MIT
metadata:
  author: https://github.com/lwpk110
  version: "1.1.0"
  domain: kubernetes
  role: specialist
  scope: implementation
  output-format: code
  triggers: operator-go, kubebuilder, kubernetes operator, generic reconciler, rolegrouphandler
---

# operator-go Engineer

## Core Workflow

1. **需求建模** — 明确产品角色、状态语义、配置边界、依赖与伸缩策略
2. **框架选型** — 固定 `operator-go + Kubebuilder`，避免自行实现平行调和主流程
3. **分层落地** — 按 CRD、controller 装配、handlers、webhook、extensions 切分职责
4. **实现调和** — 使用 `GenericReconcilerConfig` 与 `RoleGroupHandler` 构建统一调和链路
5. **质量闭环** — 完成默认值、校验、状态回写、测试与发布前验证

## Reference Guide

| Topic | Reference | Load When |
| --- | --- | --- |
| 架构与目录蓝图 | `references/architecture.md` | 新建产品 Operator，需要先做整体结构设计 |
| CRD 与调和实现 | `references/reconcile-implementation.md` | 需要定义 CRD、实现 `ClusterInterface`、接入 `GenericReconciler` |
| Webhook 与校验 | `references/webhook-validation.md` | 需要默认值、字段约束、不可变字段校验 |
| Extension 与配置生成 | `references/extensions-config.md` | 需要扩展生命周期 Hook、动态配置文件、Sidecar 组合 |
| 测试与交付 | `references/testing-delivery.md` | 需要补齐单测/集成测试、执行发布前检查 |
| Trino 对照模板 | `references/trino-mapping.md` | 需要参考 `examples/trino-operator` 迁移到其他产品 |

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

- controller 仅做装配：Manager、Reconciler、Webhook 注册，不承载业务资源构建
- CRD 实现 `ClusterInterface`，通过 `GetSpec/GetStatus/SetStatus/DeepCopyCluster` 对接框架
- 角色处理采用“路由 Handler + 每个 Role 独立 Handler”模式
- 状态更新遵循统一语义：失败标记 `Degraded`，成功写入 `ReconcileComplete` 与 `ObservedGeneration`
- 在 webhook 完成默认值与输入校验，避免问题下沉到运行期
- 对限流/暂时性错误使用 `RequeueAfter` 延迟重试

### MUST NOT DO

- 不要手写独立于 `GenericReconciler` 的主调和流程
- 不要把产品业务逻辑堆在 `cmd/main.go`
- 不要在 `BuildResources()` 中混入流程编排与状态管理
- 不要跳过 webhook 默认值与校验直接依赖运行期容错
- 不要只覆盖 happy path 测试

## Scenario Routing

- **新产品从 0 到 1**：先读 `references/architecture.md` + `references/reconcile-implementation.md`
- **新增一个 Role**：读 `references/reconcile-implementation.md` + `references/webhook-validation.md`
- **新增扩展或配置系统**：读 `references/extensions-config.md`
- **准备提测或发布**：读 `references/testing-delivery.md`
- **需要可运行模板对照**：读 `references/trino-mapping.md`
