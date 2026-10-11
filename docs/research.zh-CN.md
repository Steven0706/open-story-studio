# 调研：从 AI 片段生成到故事制作

调研日期：2026-10-11。以官方产品页面、帮助文档、仓库 README 与架构文档为依据；未登录各商业产品完成端到端实测。以下“官方介绍”不等于我们已经验证效果。公开信息不足以判断 LIBTV 的市场份额或国内排名。

## 1. 产品定位

目标不是再造一个生成按钮，也不是第一版就复制完整剪辑软件，而是建立“故事制作层”：剧本和分镜组织创作，角色与场景资产约束生成，镜头候选版本支持试错，时间线组成成片。

ComfyUI 仍然可以是底层生成引擎。它能够构建复杂工作流；缺少的不是理论生成能力，而是面向创作者的跨镜头项目组织、资产语义和编辑体验。

## 2. 商业产品参照

| 产品 | 官方信息可确认的方向 | 对我们的启发 |
| --- | --- | --- |
| [LIBTV](https://www.liblib.tv/) | 首页包含项目、资产、插件、画布与 Agent，以及图像、视频、音频、剧本和剪辑入口 | 生成应置于项目和资产工作区内；画布可作为创意入口，但不替代时间线 |
| [LTX Studio](https://ltx.io/studio) | 将角色、物体、地点组织为 Elements，并提供分镜、时间线和声音设计 | 最接近“故事→资产→镜头→成片”的产品参照 |
| [Katalist](https://www.katalist.ai/storytelling) | 强调角色一致性、分镜和多视频引擎 | 模型应可替换，创作项目不应被某一家模型锁死 |
| [Google Flow](https://labs.google/fx/tools/flow) | 围绕生成素材和场景组织视频创作；帮助文档区分参考素材、首尾帧等能力 | 引用人物/地点的交互值得借鉴；不同模型支持不同条件输入，必须明确展示 |
| [Higgsfield Cinema Studio](https://higgsfield.ai/cinematic-video-generator) | 强调镜头、光学控制和可复用人物/场景元素 | 参考其镜头控制；该页面不能作为已具备传统时间线编辑器的证据 |
| [Adobe Firefly Video Editor](https://helpx.adobe.com/firefly/web/firefly-video-editor/about-firefly-video-editor/about-firefly-video-editor.html) | 浏览器视频编辑器，支持多轨视频、音频、文本和剪辑 | 将生成直接嵌入编辑环境，而不是生成后再手工搬运文件 |

Flow 的参考素材使用方式见[官方帮助](https://support.google.com/flow/answer/16353334?hl=en)。Firefly 的不同视频模型有不同设置约束，例如 [Runway 接入说明](https://helpx.adobe.com/firefly/web/firefly-video-editor/generate-videos/generate-videos-using-runway-video-models.html)。我们的界面不能用一套“万能参数”掩盖这些差异。

LIBTV 当前公开首页不足以确认其完整时间线操作、项目格式或插件 SDK。本轮不据此推断其内部技术栈，也不复刻其品牌或私有代码。

## 3. 开源与可见源码项目

| 项目 | 相关性 | 许可/成熟度边界 | 建议 |
| --- | --- | --- | --- |
| [StoryBored](https://github.com/storybored-app/storybored) | 项目/场景/镜头、候选 take、角色、LLM 拆剧本、ComfyUI 引擎包和 animatic 输出 | MIT；README 展示功能，不等于已通过我们的部署测试 | 第一优先级复用评估对象 |
| [AI Movie Studio v2](https://github.com/Heroesjouney/AIMovieStudiov2) | 3D 分镜、本地与云端模型、多参考连续性、时间线导出 | 仓库标明 AGPL-3.0；不能将其代码直接混入 MIT 项目并忽略原许可 | 参考产品设计；代码复用需单独评估许可义务 |
| [OpenCut](https://github.com/OpenCut-app/OpenCut) | 视频编辑基础、未来插件与编辑 API | MIT；当前 README 明确处于重写中 | 跟踪，不把规划中的 API 当成现成依赖 |
| [OpenCut Classic](https://github.com/opencut-app/opencut-classic) | 可检查已有时间线、预览和媒体架构 | MIT；仓库已归档、不再维护 | 可做局部复用实验，不直接承担整个平台底座 |
| [ArtCraft](https://github.com/storytold/artcraft) | 面向艺术家的图像/视频创作工作台 | 仓库标明 MIT / Apache-2.0 双许可 | 桌面交互与创作工具参考；不因此假设能直接作为剪辑 SDK |
| [Storyboard](https://github.com/crashtestoz/storyboard) | 本地视频制作流水线 | CC BY-NC-SA 4.0，存在非商业限制；不应混称为不受限开源底座 | 仅作功能研究，默认不复用代码 |

StoryBored 的[架构文档](https://github.com/storybored-app/storybored/blob/main/ARCHITECTURE.md)显示其采用 React SPA、FastAPI、SQLite、数据库任务队列和生成适配器，与本项目的轻量自托管方向相符。它围绕 ComfyUI 引擎包构建；我们还需要评估通用远端 HTTP provider、可编辑时间线和完整版本依赖能否自然扩展。

许可表只是本轮工程筛选依据，不是完整法律意见；实际复用必须逐文件核查代码与依赖许可。AGPL 并不等于禁止商业使用，“源码可见”也不自动等于满足开源定义。

## 4. 复用还是全新开发

建议先做限时技术验证，再决定是否 fork StoryBored，而不是仅凭 README 做决定：

1. 用模拟服务运行其现有测试和演示，核实项目/场景/镜头/take/任务模型。
2. 做一个独立 HTTP 视频适配器实验，验证无需修改具体 ComfyUI 工作流即可接入。
3. 验证“替换一个 take 而保留剪辑入出点、旧版可恢复”的数据结构。
4. 验证项目导出可包含必要媒体与版本，而不是临时远端 URL。

如果核心耦合可控，保留上游许可和来源，采用 fork/模块复用；如果需要大幅改写数据模型，建立自己的小核心。无论哪种方式，新的品牌和 GitHub 项目均不应掩盖上游来源。本次仓库只包含原创调研与设计文档，未复制上述应用代码。

## 5. 差异化与主要风险

差异化应是“可追溯、可替换、可持续修改的故事项目”：角色版本与服装状态分离，场景有连续性状态，提示词和参考素材被固定到生成记录，时间线锁定具体候选素材。单纯模型数量多或界面像画布，不足以形成优势。

风险最高的不是按钮界面，而是身份漂移、跨镜头空间关系、动作接续、音频连贯性、生成失败与重复收费，以及媒体编辑的时间精度。需要人工审批关卡，不能承诺任意模型自动实现人物或故事一致性。
