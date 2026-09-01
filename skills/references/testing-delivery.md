# Testing & Delivery Checklist

## 1) 测试分层

### 单元测试（优先）

- CRD 字段到 Roles 映射逻辑
- Role 路由逻辑
- 配置生成逻辑
- webhook 默认值与校验逻辑
- extension 行为

### 集成测试

- reconcile 生命周期完整性
- 资源落地与状态更新
- 错误路径下条件回写

## 2) 最小验收要求

- 新增/变更 Role 有对应单测
- webhook 新增约束有正反案例
- 关键资源构建行为可断言
- 状态条件语义可验证

## 3) 发布前命令

仓库标准命令：

- `make generate`
- `make fmt`
- `make vet`
- `make test`
- `make lint`

说明：如果仓库已有基线失败（与本次文档/技能变更无关），需在交付说明里显式注明。
