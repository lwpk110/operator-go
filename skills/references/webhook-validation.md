# Webhook & Validation Guide

## 1) Defaulter 责任

- 保证 CR 进入调和前具备最小可运行默认值
- 默认值应覆盖镜像、端口、必要结构初始化（如 role 对象）
- 默认逻辑幂等：重复执行不改变语义

## 2) Validator 责任

- Create/Update 都执行结构与语义校验
- 校验范围包括：
  - 镜像格式
  - 端口范围
  - 名称格式
  - 枚举合法性
- Update 必须处理不可变字段（immutable）

## 3) 错误表达

- 返回聚合错误，指出精确字段路径（如 `spec.catalogs[i].type`）
- 错误信息可用于用户直接修正输入
- 不泄露实现细节，只暴露约束语义

## 4) 设计原则

- 约束前置：能在 webhook 失败就不要等到 reconcile 失败
- 默认值明确：默认配置必须可解释、可预期
- 与 CRD schema 协同：schema 负责静态约束，webhook 负责动态/跨字段约束
