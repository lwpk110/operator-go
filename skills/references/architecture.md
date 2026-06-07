# Architecture Blueprint for Product Operators

## 1) 目标

把产品 Operator 设计为“框架稳定 + 业务可插拔”：

- 框架层复用 `operator-go` 标准调和生命周期
- 产品差异集中在 CRD 字段、Role 处理器、扩展和配置生成

## 2) 分层职责

### API 层（`api/v1alpha1`）

- 定义产品 CRD 的 Spec/Status
- 将产品角色字段（如 coordinators/workers）映射到框架 Roles
- 对外提供类型安全 API；对内兼容通用调和框架

### 装配层（`cmd/main.go`）

- 初始化 manager 与 scheme
- 注册 extension
- 创建 `GenericReconcilerConfig` 并调用 `NewGenericReconciler`
- 注册 webhook 和健康检查

### 调和实现层（`internal/controller` + `internal/handlers`）

- controller 实现角色路由
- 各 Role handler 负责构建 RoleGroup 资源（ConfigMap、Service、StatefulSet、PDB）

### 校验层（`internal/webhook`）

- 默认值：补齐镜像、端口、最小可运行配置
- 校验：字段合法性、枚举、格式、不可变字段

### 扩展层（`internal/extensions`）

- 通过 Cluster/Role/RoleGroup Hook 增加通用行为
- 典型场景：健康状态、目录/数据源动态扩展、审计逻辑

## 3) 目录基线

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

## 4) 设计检查清单

- 是否已明确每个 Role 的职责边界与资源形态
- Spec 是否避免重复表达（产品字段与通用 roles 不双写）
- Status 是否可表达可观测状态与调和结果
- controller 是否保持“装配薄层”
- 是否预留扩展点而非硬编码在 handler
