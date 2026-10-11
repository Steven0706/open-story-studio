# Provider 插件契约草案

状态：设计草案，不是已发布 SDK。目标是让项目数据不绑定某个模型，同时诚实表达模型差异。

## 1. 三个概念

Provider 表示服务接入，例如本地 HTTP 或某云 API；Model 表示具体模型/工作流版本；Operation 表示一次能力调用，例如 `llm.chat`、`image.generate`、`image.edit`、`video.t2v`、`video.i2v`、`audio.tts`。

“可插件化”表示未来可以编写适配器，不是任何服务仅填 URL 就能立即兼容。

## 2. 能力清单

每个 model/operation 返回经过服务端校验的清单：

| 字段 | 含义 |
| --- | --- |
| `manifest_version` | 插件契约版本 |
| `provider_id`, `model_id`, `model_revision` | 服务与模型版本标识；不知道精确模型版本时显式标 unknown |
| `operation` | 操作类型 |
| `input_schema` | JSON Schema：必填/可选项、未知字段拒绝、合法参数范围 |
| `conditioning` | first_frame、last_frame、identity_references、style_references 等独立能力及数量上限 |
| `duration`, `resolution`, `aspect_ratios` | 可选枚举/范围以及输出约束 |
| `execution` | sync/async、poll/webhook、是否支持取消、幂等键、进度 |
| `resource_group` | 共享 GPU/队列身份，不等于 provider_id |
| `terms` | 模型许可/服务条款链接、商用信息（无法确认则 unknown） |

UI 根据当前操作生成控制项。首帧条件不等于身份参考；只支持首帧时，身份参考按钮必须禁用并说明原因。对于不支持的尾帧、参考数量、分辨率或时长，拒绝或要求用户明确选择降级，不静默丢参数。

## 3. 适配器边界

建议异步接口：

```text
discover_models() -> model capability manifests
validate(request) -> validated request / unsupported capability
submit(request, request_key) -> result OR remote job receipt
poll(remote_job_id) -> normalized status / result
cancel(remote_job_id) -> acknowledged / unsupported / unknown
download(result) -> validated local media assets
```

平台传入资产 ID；适配器负责解析为允许访问的本地文件、上传或供应商需要的引用。密钥通过服务器配置传入，不属于项目或生成配方。错误类型区分参数不合法、未配置、鉴权失败、限流、远端失败、网络不确定与媒体校验失败。

工作流包是另一种适配器输入：ComfyUI API graph + 参数映射 + 输出映射。不同引擎的节点 ID 不进入项目的核心数据模型。

## 4. 任务状态与恢复

```text
queued → submitting → submitted → running → downloading → completed
            │             │          │            │
            └ unknown     └ failed   └ canceled   └ failed
```

这是平台规范化状态，不是要求所有供应商都提供这些状态。progress 无法测量时为 null，不编造百分比。

提交前持久化 recipe snapshot 和本地 request key，收到远端 ID 后立即持久化。超时/断网可能发生在远端已接受任务之后：无可靠幂等键时进入 unknown，由查询/人工核实解决，不自动重发 POST。重试轮询和下载通常安全；重试创建生成任务不一定安全。

平台的 at-least-once worker 不能对外声称实现了 exactly-once 生成。支持幂等的 provider 使用相同 key 恢复；不支持的 provider 必须有明确的不确定状态。

取消本地等待不等于远端生成已停止；分别记录用户取消请求和远端确认。远端任务完成但下载失败，不重新生成，只重试下载。实际输出参数与请求参数分开保存。

## 5. 初始适配器顺序

1. 模拟 provider：无 GPU、无费用，用于测试超时、失败、恢复、参数拒绝。
2. OpenAI-compatible LLM：剧本拆分和提示词草稿。
3. 已有 HTTP T2V/I2V 服务：共享资源组，first-frame-only I2V。
4. 图像生成/编辑服务：候选方案确定后接入；此前支持手动首帧。
5. 通用 ComfyUI workflow pack 与至少一个不同协议 provider：验证抽象真正跨引擎，而不是只把一家 API 包一层。

音频接口保留设计位置，但不阻塞 MVP；先允许导入旁白和音乐。
