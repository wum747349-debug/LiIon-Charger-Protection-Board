# Stage 3 — Module Design Entry

Status: Entry baseline only — module design and schematic capture not started

本目录按绑定 Framework 的 Stage 3 structure contract 启用。后续模块文档将在实际开展 charger / Power Path、USB-C input control、3.3V Buck-Boost、output protection、external NTC 和接口设计时分别记录连接、参数计算、依据、风险与布局要求。

当前输入边界：

- Stage 2 Critical Component Selection closeout PASS，Stage 3 已获用户授权。
- TYPE-C-31-M-12 是 Rev.A USB-C receptacle Primary；1.6 mm FR-4 仅为 preliminary PCB thickness baseline。
- USB 输入策略为 5V Sink-only、no PD、no data；legacy USB-A 不实现 BC1.2 / proprietary detection。
- TUSB320 的 initial-only advertisement limitation 已接受；unknown / invalid / unqualified state 必须采用 conservative input-current behavior。
- BQ25185、TPS63802、TPS259474、NTC、VBUS TVS 及 1A / 100ms waveform definition 的详细工作均由后续 Stage 3 模块设计完成。
- Battery connector series / polarity / wire gauge 保持 Physical Verification Pending；battery product-spec 与 pack internal protection behavior 保持 Not Verified 边界。

本文件不包含已完成的电路、外围定值、原理图、Final BOM、EDA、ERC、DRC、simulation、thermal/procurement/manufacturing verification、bring-up 或 test 证据。
