# Open Story Studio

An open-source, local-first AI filmmaking workspace: turn generated shots into a coherent story, not a folder of disconnected clips.

一个面向 AI 短片制作的开源工作台：管理剧本、人物、场景、镜头、提示词和候选素材，在时间线上反复调整，并连接不同的生成模型。

**Status: research and design only. There is no runnable application yet.**

当前为调研与技术设计阶段；下列功能均为规划，尚未实现。项目名为暂定名。与 LIBTV、LTX Studio 等产品无隶属关系。

## Documentation

- [竞品与开源调研](docs/research.zh-CN.md)
- [技术选型与架构](docs/architecture.zh-CN.md)
- [MVP 范围、实施顺序与验收](docs/mvp.zh-CN.md)
- [模型插件契约草案](docs/provider-contract.zh-CN.md)

## Proposed workflow

```text
Story → Scenes → Shots → Reference frames → Generated takes → Timeline → Film
                     ↑ characters, locations, continuity state ↑
```

Planned foundations:

- Versioned character/location assets and human-approved storyboards.
- Provider adapters for language, image, video, and eventually audio models.
- Multiple takes per shot, explicit selection, reproducible generation recipes.
- A lightweight timeline with trim, reorder, replacement, audio, and MP4 export.
- Persistent jobs, local media storage, portable projects, no mandatory cloud account.

## Scope and limitations

Web-first means a browser UI backed by a locally/self-hosted API, not a cloud-only website. Desktop packaging is a later option. A provider adapter cannot add capabilities the underlying model does not have. Character consistency is a production workflow and review problem, not a guaranteed property of every generation.

The repository intentionally contains no deployment addresses, credentials, generated media, or model weights. Actual model/provider licenses and usage terms are separate from the software license.

## License

[MIT](LICENSE). Third-party code, model weights, and media retain their own licenses. No third-party application source has been copied into this initial repository.
