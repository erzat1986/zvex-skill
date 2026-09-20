# ZVEX Skill

声桥（zvex）多语种视频译制配音的 **支付宝 AI 付智能体技能包**（Agent Skills 格式）。

安装本技能后，支持支付宝 AI 付的智能体（OpenClaw 等）即可**零注册、零预充值**调用两个按量付费服务：

| 能力 | 端点 | 计价 |
|---|---|---|
| 多语种文本翻译 | `POST https://zvex.cn/a2m/v1/translate` | ¥0.1 / 次 |
| AI 视频配音 | `POST https://zvex.cn/a2m/v1/dubbing` | ¥1/分钟（最低 ¥2），按实测时长动态计费 |

## 安装

把本仓库（或 `SKILL.md`）交给你的智能体：

- **OpenClaw**：按技能目录规范放入 skills 目录，或直接引用本仓库地址
- **通用 Agent Skills 客户端**：复制 `SKILL.md` 到对应技能目录

## 工作方式

本技能是 A2M（402 协议）的调用说明书：

1. 智能体按 `SKILL.md` 中的端点发起调用
2. 收到 `402 + Payment-Needed` 签名账单后，用支付宝 AI 付钱包自动完成支付
3. 携带 `Payment-Proof` 重试，拿到资源；`Payment-Validation` 可二次校验

配音为异步任务：受理后凭 `out_trade_no` 轮询状态接口拿成片与字幕。

## 相关仓库

- MCP Server（API Key 模式，适配 Claude/Cursor 等客户端）：<https://github.com/erzat1986/zvex>
- 服务官网与协议文档：<https://zvex.cn> · [MCP/AI 付接入文档](https://zvex.cn/docs/mcp)

## 许可

MIT
