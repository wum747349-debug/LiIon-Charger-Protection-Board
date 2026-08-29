# 阶段文档

本目录保存按阶段启用或按项目需要创建的文档。Bootstrap 不预建这些内容。

| 路径                            | 职责                                      | 启用时机                                     |
| ----------------------------- | --------------------------------------- | ---------------------------------------- |
| `component_selection_plan.md` | 关键器件候选与决策                               | Stage 2 开始                               |
| `project_design_overview.md`   | 面向工程师、项目展示和面试复习的系统架构、模块协同与关键设计决策说明 | 系统设计形成后按硬件版本维护                           |
| `module_design/*.md`          | 模块连接、计算、依据、风险和布局要求                      | 模块进入 Stage 3 详细设计                        |
| `schematic_review.md`         | 实际原理图审查依据、问题、状态和结论                      | Stage 4 开始                               |
| `pcb_design_rules.md`         | 项目规则值、Scope、Priority、配置状态和豁免定义          | Stage 5 Layout Preflight 前               |
| `pcb_review.md`               | Layout Preflight、布线问题、用户 DRC 摘要、豁免和制造放行 | 记录 Layout Preflight 时启用，在 Stage 6–7 继续维护 |
| `bringup_log.md`              | 装配、首次上电、测量和调试历史                         | Stage 8 开始装配或上电调试                        |
| `test_report.md`              | 测试条件、结果、限值和验收结论                         | Stage 8 开始正式测试                           |
| `revision_history.md`         | 硬件版本、变更原因、影响和必要的重新验证                    | 确立首个硬件版本或发生重要变更                          |
| `user/`                       | 面向用户的配置、接线、接口和安全说明                      | 项目确需用户文档                                 |

当前项目阶段只在仓库根 `README.md` 中维护。文件存在不表示相应阶段已经通过。
