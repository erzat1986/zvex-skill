---
name: zvex
description: 多语种视频译制配音与文本翻译服务（声桥 zvex）。当用户需要视频配音、把视频或文本译制成其他语言、字幕翻译、短剧出海本地化、多语种内容生产时使用。当前开放目标语言：俄语、英语、西班牙语（持续扩展）；经支付宝 AI 付按次自动结算，无需注册和充值。
---

# zvex 多语种视频译制配音

把视频链接或文本交给声桥：自动完成语音识别、说话人分离、翻译、AI 音色复刻配音（按角色还原原说话人音色）、字幕与成片合成。

## 能力与定价

| 能力 | 端点 | 计价 |
|---|---|---|
| 视频译制配音 | `POST https://zvex.cn/a2m/v1/dubbing` | ¥1/分钟（最低 ¥2/单），按服务端 ffprobe 实测时长动态计费 |
| 文本翻译 | `POST https://zvex.cn/a2m/v1/translate` | ¥0.1/次 |

目标语言（`target_language`）当前开放：`ru` / `en` / `es`（持续扩展，以部署支持集为准）。

## 调用流程（支付宝 AI 付 · A2M 402 协议）

1. **无凭证调用**：直接 POST，返回 `402`，响应头 `Payment-Needed` 携带 Base64URL 编码账单。解码后 `protocol` 含 `out_trade_no`、`amount`、`currency`、`resource_id`、`pay_before`、`seller_signature`（商家 RSA2 签名，收款方经平台核验）。
2. **支付**：使用你的支付宝 AI 付钱包按账单完成支付（账单已由商家签名，金额与收款方以账单为准）。
3. **携带凭证重试**：支付完成后取得 `trade_no` 与 `payment_proof`，构造请求头：

   ```
   Payment-Proof: Base64URL(JSON{
     "protocol": {"payment_proof": "<...>", "trade_no": "<...>"},
     "method":   {"client_session": <可选>}
   })
   ```

   重试原请求 → `200` 交付资源，响应头 `Payment-Validation` 可作二次校验。
4. **凭证失败处理**：`402 INVALID_PAYMENT_PROOF` 表示凭证无效/过期/已使用——**不要重用旧凭证**，重新发起调用获取新账单再支付。

## 文本翻译（同步）

```json
POST /a2m/v1/translate
{"text": "要翻译的文本", "target_language": "ru"}

200 → {"resource_id": "/a2m/v1/translate",
       "content": {"status": "success", "translation": "<译文>", ...}}
```

`text` 单次不超过 64KB。

## 视频配音（异步任务）

```json
POST /a2m/v1/dubbing
{"video_url": "https://example.com/talk.mp4", "target_language": "zh"}

200 → {"content": {"job_id": 123, "status": "queued",
                   "poll": "GET /a2m/v1/dubbing/status/{out_trade_no}"}}
```

验付通过即受理入队（返回较快），成片需轮询状态接口：

- `status = queued / processing` → 等待后再次查询（任务耗时数分钟，视视频时长与队列）
- `status = completed` → 返回 `final_video_url` 与 `subtitle_url`
- `status = failed` → 返回 `error`；费用补偿联系商家

`out_trade_no` 即查询凭证，请保留。

## 约束

- `video_url` 必须是公网可直连的 http(s) 视频文件直链，单个 ≤ 500MB
- 动态账单金额由服务端按实测时长生成，支付金额必须与账单一致
- 同一 `payment_proof` 只能使用一次；凭证无效时重新获取账单

## 服务方

声桥 zvex（<https://zvex.cn>）。两个服务均已上架支付宝 AI 付服务市场（搜索"声桥"或"zvex"）：声桥AI文本翻译（`API_346E3890E1B7484B`）、声桥AI视频配音（`API_8A3F6B99D33E4D36`）。协议细节与 MCP 接入方式见 <https://zvex.cn/docs/mcp>。
