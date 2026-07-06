---
marp: true
theme: cenbrain
paginate: true
size: 16:9
title: 面向实验协议的 rteeg 语音客户端采集基础设施
---

# 面向实验协议的 rteeg 语音客户端采集基础设施

**短期研究进展汇报**  
Peibin / rteeg speech project  
2026 年 7 月

---

# 汇报概要

- 近期工作聚焦于 **rteeg-speech-client**，而不是新的神经解码算法。
- 主要贡献是构建更 **面向实验协议的采集前端**，服务于神经语音解码实验。
- 主要进展包括：
  - YAML 任务控制
  - 句子预览与字符级执行
  - 实时/最终解码显示分离
  - 标记信息队列化发送
  - 辅助音频与截图采集
- 证据边界：目前仅支持实现层面的结论；暂不声称准确率、临床效果或正式延迟指标。

---

# 研究动机

神经语音解码系统不仅依赖模型推理。

- 视觉刺激需要可重复。
- 实验事件需要被明确标记。
- 在线输出需要在采集过程中可见。
- 辅助行为轨迹应能与神经数据对齐。

**本阶段定位：**准备一个可控、可观察、可审计的采集界面，为后续语音解码数据采集打基础。

---

# 系统背景

![width:920px](figures/architecture.svg)

---
<!-- _class: two-cols -->

# 客户端在流程中的角色

<div class="columns">
<div class="col-left">

语音客户端是面向受试者和实验操作者的交互层。

| 职责 | 目的 |
|---|---|
| 加载任务文件 | 定义可重复的提示语和时间参数 |
| 呈现实验刺激 | 控制受试者看到的内容 |
| 发送事件标记 | 标记 session、句子、词或字符事件 |
| 接收解码消息 | 展示在线反馈 |
| 采集音频/截图 | 提供辅助审计数据流 |

</div>
<div class="col-right">

<div class="figure-placeholder">
<div class="figure-title">采集闭环流程</div>
<div class="mini-flow">
<span>任务 YAML</span><span class="arrow">→</span><span>预览界面</span><span class="arrow">→</span><span>标记队列</span><span class="arrow">→</span><span>服务端记录</span><span class="arrow">→</span><span>解码反馈</span>
</div>
<div class="figure-subtitle">建议文件：<code>figures/acquisition-flow.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# 执行层结构

<div class="columns">
<div class="col-left">

| 层级 | 主要职责 | 研究意义 |
|---|---|---|
| Vue renderer | 任务状态、刺激呈现、预览覆盖层、解码显示 | 受试者交互与可见反馈 |
| Electron main process | session 发现、ZeroMQ 通信、音频流、截图、IPC 路由 | 与记录和处理管线通信 |

</div>
<div class="col-right">

<div class="figure-placeholder blue">
<div class="figure-title">Renderer / Main Process</div>
<div class="mini-flow">
<span>Vue Renderer</span><span class="arrow">⇄</span><span>IPC Bridge</span><span class="arrow">⇄</span><span>Electron Main</span><span class="arrow">⇄</span><span>ZeroMQ</span>
</div>
<div class="figure-subtitle">建议文件：<code>figures/client-layer-separation.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# 进展 1：YAML 任务配置

<div class="columns">
<div class="col-left">

已实现或文档化的任务字段包括：

- `preview_time`
- `show_preview_text`
- `record_by_character`

这些字段将任务时间控制和采集粒度显式写入配置。

**研究价值：**为语音实验提供可重复的刺激与时间定义。

</div>
<div class="col-right">

<div class="figure-placeholder">
<div class="figure-title">任务配置 → 运行时对象</div>
<div class="mock-panel"><strong>YAML</strong><br><code>preview_time</code><br><code>show_preview_text</code><br><code>record_by_character</code></div>
<div style="color:#FB9231; font-size:28px; margin:8px;">↓</div>
<div class="mock-panel"><strong>运行时对象</strong><br>预览状态 · 执行单元 · 标记计划</div>
<div class="figure-subtitle">建议文件：<code>figures/yaml-task-schema-flow.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# 进展 2：句子预览与字符级执行

<div class="columns">
<div class="col-left">

实验核心现在可以：

- 展示带倒计时的句子预览；
- 在保留时间控制的同时隐藏预览文字；
- 将短语拆分为字符级单元；
- 在执行过程中发出 session、句子和词级标记。

**研究价值：**更可控的刺激呈现与更细粒度的事件标签。

</div>
<div class="col-right">

<div class="figure-placeholder">
<div class="figure-title">预览与执行时间线</div>
<div class="timeline-labels"><span>预览</span><span>句子</span><span>单元</span><span>完成</span></div>
<div class="timeline-bar"></div>
<div class="timeline-labels"><span>显示/隐藏文字</span><span>起始 marker</span><span>词/字符标签</span><span>结束 marker</span></div>
<div class="figure-subtitle">建议文件：<code>figures/preview-character-timeline.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# 进展 3：在线解码反馈显示

<div class="columns">
<div class="col-left">

客户端将以下内容分离显示：

- 当前 trial 中的 **实时 partial decoding**；
- 句子完成后的 **最终句子历史记录**。

main process 的 topic routing 区分 partial、word-level、sentence-word 与 final sentence decoding 消息。

**研究价值：**提高在线反馈的可观察性，但不声称显示结果一定准确。

</div>
<div class="col-right">

<div class="figure-placeholder blue">
<div class="figure-title">解码显示界面示意</div>
<div class="mock-panel"><strong>实时 partial decoding</strong><br><code>... real-time words ...</code></div>
<div class="mock-panel"><strong>最终句子历史</strong><br><code>sentence_result[]</code></div>
<div class="figure-subtitle">建议文件：<code>figures/decoding-display-mockup.png</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# 进展 4：标记可靠性

<div class="columns">
<div class="col-left">

control client 通过 promise queue 串行化发送 marker。

main process 也会缓存 control client 就绪前产生的 marker，并在初始化后刷新发送。

**解决的问题：**过快或过早的 marker 调用可能与 ZeroMQ request/reply 顺序冲突。

**可声明：**客户端降低了控制消息顺序风险。  
**暂不声明：**已达到毫秒级同步精度。

</div>
<div class="col-right">

<div class="figure-placeholder blue">
<div class="figure-title">Marker 队列机制</div>
<div class="mini-flow">
<span>UI marker</span><span class="arrow">→</span><span>待发送缓存</span><span class="arrow">→</span><span>Promise queue</span><span class="arrow">→</span><span>ZMQ control</span><span class="arrow">→</span><span>服务端日志</span>
</div>
<div class="figure-subtitle">建议文件：<code>figures/marker-queue-sequence.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# 进展 5：辅助采集流

<div class="columns">
<div class="col-left">

客户端可以采集辅助行为证据：

| 数据流 | 当前作用 | 后续验证需求 |
|---|---|---|
| 音频 | 单声道麦克风包发送到服务端音频通道 | 声学延迟和数据包时间 |
| 截图 | 带时间戳的帧和回调信息 | 与服务端记录对齐的帧时间 |

</div>
<div class="col-right">

<div class="figure-placeholder">
<div class="figure-title">多数据流对齐</div>
<div class="mock-panel">神经数据轨道</div>
<div class="mock-panel">Marker 轨道 · 音频包 · 截图帧</div>
<div class="mock-panel">在线解码消息</div>
<div class="figure-subtitle">建议文件：<code>figures/multistream-alignment-placeholder.svg</code></div>
</div>

</div>
</div>

---

# 主要进展汇总

| 进展方向 | 本地证据 | 研究价值 |
|---|---|---|
| 任务配置 | README、schema、`configUtils.ts` | 可重复的刺激/时间定义 |
| 预览与字符模式 | `expcore.ts`、`ExperimentView.vue` | 可控呈现与细粒度标签 |
| 在线反馈 | `index.ts`、`DecodingDisplay.vue` | 可观察的实时/最终解码流 |
| Marker 队列 | `netutils.ts`、`index.ts` | 降低控制消息顺序风险 |
| 辅助流 | `audio.ts`、`screenshot.ts` | 为后续审计提供行为证据 |

---

# 研究意义

后续 decoder 的结果只有在采集记录足够清楚时才容易解释。采集记录应说明：

1. 哪个刺激被展示；
2. 句子、词或字符单元何时开始；
3. 当时可见的在线输出是什么；
4. 哪些辅助行为流被记录；
5. marker 与 session metadata 如何关联到神经数据。

本阶段贡献的是解码性能评估之前的 **基础设施准备度**。

---

# 声明边界

| 可以声明 | 应避免声明 |
|---|---|
| 客户端实现了面向实验协议的任务和 marker 基础设施。 | 客户端提高了神经解码准确率。 |
| 在线显示区分实时和最终句子输出。 | 系统已经是完整临床语音神经假体。 |
| 队列化 marker 发送降低了控制路径中的顺序风险。 | marker 时间已验证到毫秒级精度。 |
| 音频和截图流可支持后续审计。 | 辅助流已与神经数据完成精确同步。 |

---

# 当前限制

现有材料尚不包含：

- 神经解码准确率；
- 患者或受试者结果；
- 正式 marker 延迟测量；
- 临床验证；
- 已验证的音频或截图时间精度；
- 与服务端记录系统的完整端到端集成测试。

因此，当前结论应保持在实现层面。

---
<!-- _class: two-cols -->

# 验证计划

<div class="columns">
<div class="col-left">

下一阶段应将已实现机制转化为可测量证据。

1. 端到端 marker 一致性检查
2. marker / 音频 / 截图延迟测量
3. 预览与字符级任务的 UI 彩排
4. YAML 工作流文档整理
5. 与服务端记录管线的集成测试
6. 失败模式文档化

</div>
<div class="col-right">

<div class="figure-placeholder blue">
<div class="figure-title">验证路线图</div>
<div class="mini-flow">
<span>实现证据</span><span class="arrow">→</span><span>Session 彩排</span><span class="arrow">→</span><span>时间基准测试</span><span class="arrow">→</span><span>服务端集成</span><span class="arrow">→</span><span>采集质量证据</span>
</div>
<div class="figure-subtitle">建议文件：<code>figures/validation-roadmap.svg</code></div>
</div>

</div>
</div>

---

# 总结

短期贡献是构建了一个更 **面向实验协议的 rteeg 语音客户端**，用于神经语音解码实验。

它集成了：

- 任务可配置性；
- 刺激预览；
- 在线反馈显示；
- marker 队列化；
- 辅助音频与截图采集。

其价值是支持 **更可控的未来数据采集**。其边界是：解码性能和同步精度仍需要实证验证。

---

# 背景参考

- Metzger et al., *Nature*, 2023 — 高性能语音神经假体背景
- Willett et al., *Nature*, 2023 — 实时通信与记录管线背景
- Card et al., *NEJM*, 2024 — 临床语音 BCI 背景
- Moses et al., *NEJM*, 2021 — 语音神经假体相关背景
- Makin et al., *Nature Neuroscience*, 2020 — 神经解码相关背景

<!-- 最终版本可替换为项目专用 bibliography slide 或二维码/链接。 -->
