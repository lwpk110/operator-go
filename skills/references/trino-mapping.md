# Trino Example Mapping (for Other Products)

将 `examples/trino-operator` 作为可迁移模板时，按下面映射替换：

## 1) API 层映射

- `api/v1alpha1/trinocluster_types.go`
    - 替换为 `<product>cluster_types.go`
    - 保留 `ClusterInterface` 实现套路
    - 将 `coordinators/workers` 替换为产品角色模型

## 2) 装配层映射

- `cmd/main.go`
    - 保留 manager 初始化、scheme 注册、healthz/readyz
    - 替换产品 API、handler、webhook、extension 注册
    - 保留 `GenericReconcilerConfig` 装配模式

## 3) 调和层映射

- `internal/controller/trino_handler.go`
    - 保留 role 路由骨架
    - 新增或替换为产品角色 handler

- `internal/handlers/*_handler.go`
    - 将 Trino 资源构建改为产品资源构建
    - 保留 RoleGroup 粒度构建思路

## 4) Webhook 映射

- `internal/webhook/v1alpha1/trinocluster_webhook.go`
    - 迁移默认值、校验、不可变字段处理模式
    - 替换为产品字段约束

## 5) 扩展与配置映射

- `internal/extensions/*.go`：保留 Hook 机制，替换业务逻辑
- `internal/config/*.go`：保留配置生成层分离，替换产品配置格式

## 6) 测试映射

- 复制 Trino 的 API/controller/handlers/webhook/extensions/config 测试结构
- 先保持结构一致，再替换断言内容为产品语义

## 7) 迁移完成判定

- 核心角色都能走通 reconcile
- webhook 能拦截非法输入
- status 条件与观测字段可反映真实状态
- 样例 CR 可部署并稳定运行
