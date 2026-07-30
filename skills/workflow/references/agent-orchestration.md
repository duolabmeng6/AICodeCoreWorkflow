# 代理编排参考

## 目录

- [决策矩阵](#决策矩阵)
- [契约与屏障](#契约与屏障)
- [回退与闭环](#回退与闭环)
- [账本字段](#账本字段)

## 决策矩阵

| 等级 | 判定 | 必需步骤 |
| --- | --- | --- |
| simple | 清晰、局部机械、无不确定性、有客观验证 | 根代理记忆→授权→计划/账本→实施→验证→验收；可不委派 |
| standard | 交互/调试/陌生路径/多文件耦合/回归风险 | 只读 explorer→根最终计划→一个写 worker→独立 reviewer→集成验证 |
| complex/high-risk | 架构、跨系统、公共接口、迁移、安全、权限、并发、不可逆、多工作流或验证缺口 | 独立问题 explorer（可并行）→计划 reviewer→workers→按风险轴 reviewer→集成验证 |

根代理记录分类理由。升级补齐屏障，降级不删除证据。explorer 完成屏障仅适用于 standard/complex；simple 跳过委派且不得因缺少 explorer 阻塞。

## 契约与屏障

负载字段：objective/question；相关路径；约束/非目标；所有权与只读边界；验收/验证；预期精炼输出。禁止子代理委派。输出只含结论、证据位置、风险/未决项及变更/测试/发现，不含原始日志、完整搜索、思维链、代理身份、模型、令牌或运行轨迹。

standard/complex explorer 结果必须完成后才能最终计划和产品写入；simple 豁免。复杂计划 reviewer 必须是独立只读能力/代理，不能是根计划作者或任何执行 worker，并且必须先于 workers 完成；所有 worker 完成后才 review；reviewer 与被评 worker 不并发；实质修复需聚焦复评。执行中根代理只读并维护编排计划/CoreFlow/memory，不改 worker 所有文件。默认一名写 worker；并行写入须声明不重叠路径/符号、输入输出依赖、验证方法，并由根代理做交集检查，遇重叠或动态依赖串行化或重派。

## 回退与闭环

发现系统能力并确保匹配；一次有界重试后，simple/standard 可由根代理回退（须等价证据与验证），complex 只有建立等价证据才可回退，否则 blocked。记录失败/回退，不能伪造证据；缺少代理不阻塞 simple。每个评审发现记录 severity、evidence、owner、disposition、verification；worker 不评审自身。实质发现执行 patch→聚焦复评；两轮仍存在则重计划、缩小范围、请求输入/权限或报告阻塞。根代理检查最终文件/diff 并运行集成验证。

## 账本字段

`implement.md` 保存分类理由、编排计划、所有权/依赖/验证及修订；`research/` 保存精炼事实、证据、未决项；`review.md` 保存发现、处置和复评。`task.json` 仅生命周期与工件索引。禁止记录代理 ID/名字/模型、令牌、完整 prompt、transcript、原始日志、实时状态、敏感信息。CoreFlow 全生命周期不运行 Git。
