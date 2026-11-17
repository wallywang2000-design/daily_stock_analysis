# new action 1029 运行维护

本页记录 wallywang2000-design/daily_stock_analysis 的定时分析配置。

- 任务文件：`.github/workflows/new-action-1029.yml`。
- 定时：`17 12 * * 1-5`，北京时间周一至周五 20:17；GitHub 调度可能延迟。
- 原“每日股票分析”任务保持手动停用，避免同一天重复分析。
- Repository variable `ANALYSIS_DELAY=120`：5 只股票之间的 4 次等待累计 8 分钟；该等待发生在结果收集阶段，不是 API 并发控制。
- 首选原密钥的 Gemini 3.6 Flash，依次回退到 Wally 密钥的 Gemini 3.6 Flash、原密钥的 Gemini 3.8 Flash、Wally 密钥的 Gemini 3.5 Flash-Lite。
- 后两个型号在 2026-11-17 的 Google 官方价格页提供免费层，并已在 GitHub runner 验证实际 API 和 LiteLLM 调用。免费层仍受配额及临时服务繁忙影响。
- 密钥继续从现有 GitHub Secrets 读取；此配置不启用付费服务。

## 验收与故障定位

`Gemini key check 1029` 仅手动运行且不推送 Telegram。默认模式验证生产路由与真实回退；`probe_alternatives=true` 用少量请求验证两个免费候选型号的直接 API 和 LiteLLM 可用性。

检查完整任务时，必须同时核对日志中的成功/失败股票数量、Telegram 发送结果和报告上传结果。应用在所有个股分析失败时仍可能以 0 退出，因此 Actions 绿色状态本身不能证明 AI 分析成功。

GitHub 的派生项目暂停需要维护者重新启用；API 503 代表模型服务暂时繁忙，429 代表额度限制，均不能仅据此认定密钥被删除。

若需回滚跨型号回退，删除两个新增的 model_list 条目，并将 LITELLM_FALLBACK_MODELS 恢复为 gemini-wally-backup。等待间隔可单独在 Repository variables 中调整；无需更换密钥。

参考：[Google 模型价格](https://aistudio.google.com/docs/pricing)、[Google 配额说明](https://ai.google.dev/gemini-api/docs/rate-limits)。本文为该派生项目的中文运维记录，不涉及上游产品双语文档同步。
