# 硬件源文件与证据

本目录保存项目的硬件实现源文件与可直接查看的导出结果。

- `altium_project/`：最新维护的 `.PrjPcb`、`.SchDoc`、`.PcbDoc` 以及当前项目需要保留的工程库文件；
- `outputs/`：方便直接查看的原理图 PDF、BOM 等人类可读输出；
- 项目照片、PCB 3D、Bring-up 和示波器截图可在公开整理时放入仓库的展示图片目录，并由根 `README.md` 引用。

## Source / Hardware Revision Boundary

物理硬件版本为 `Rev.A`，已经完成制造、装配与 primary-function basic Bring-up。

仓库中的 Altium 文件是当前最新维护版本。`.SchDoc` 与 Rev.A 设计基线一致；`.PcbDoc` 在实际 Rev.A 制造后经历过少量 layout / copper / via / solder-mask refinement，因此不声明它与当时下单制造数据 byte-equivalent。

仓库内所有实测电压、充电电流、ripple / noise 和 Debug 结果均指向实际 Rev.A 实板，而不是后续维护过的 PCB 文件。

## Evidence Rule

`.SchDoc` 和 `.PcbDoc` 是当前 EDA 实现源，但目录、PDF、图片或文字设计意图本身不能证明 ERC、DRC、规则命中、铺铜状态或制造放行。

实际 EDA 操作、制造与测试证据仍以对应工程记录为准。AI / Codex 可以辅助分析、Review、测试规划与文档整理，但不能把未执行的检查或未测量的结果写成 PASS。

本公开项目不以制造 release archive 为目标，因此不要求保留 Gerber、Drill、Pick & Place 或 Assembly package。需要新的 manufacturing outputs 时，可以由当前维护的 Altium 工程重新生成，并重新执行必要的 release checks；这些新输出只对应当前维护设计版本，不用于声明可精确复现历史 Rev.A manufacturing package。
