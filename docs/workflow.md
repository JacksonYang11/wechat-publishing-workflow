# 从热点选题到公众号发布

## 1. 每天用 Trae 收集热点、写初稿

1. 在自己的飞书中新建云文档《已写历史话题记录》。可以先把过去写过的历史话题补进去，每条写上日期、历史话题、叙事角度和文章标题，方便任务查重。
2. 复制文档链接，替换 [历史选题和写文提示词](../prompts/trae-history-drafts.md) 中的 `{{FEISHU_HISTORY_TOPICS_DOC_URL}}`。确认 Trae 定时任务使用的飞书连接可以读取和编辑这份文档。
3. 在 Trae 创建每日自动化任务，粘贴填好链接的提示词，选一个方便自己看稿的运行时间。任务选题前会读取记录，写完后把本次实际完成的初稿逐篇追加到文档末尾。

### Trae 的模型和用量

- **用 Trae 会员。** 我开了 Trae 会员来获取 AI 用量。日常跑选题和初稿时，DeepSeek 是我用下来性价比最高的选择。
- **用自己的模型 API。** 手头已有 DeepSeek 等模型厂商的 API 账号，也可以在支持自定义模型的 Trae 版本里填入厂商的 API 基础地址（Base URL）和自己创建的 API Key，再选择对应模型。通过这组配置发出的模型请求消耗厂商账号里的用量。给定时任务使用前，先确认该任务可以选到这个自定义模型。

## 2. 人工选稿、审阅和定稿

定时任务交来的是初稿。我通常在这一步花最多时间，因为哪篇值得写下去、哪些细节该留、文章读起来有没有自己的味道，都得靠我自己判断。

我会在当天的初稿和以前写过的稿子里挑一篇，先从头读一遍。审稿时，我会核对关键史实和引文，看看热点能不能自然带出历史故事，再检查标题、开头和叙事顺序。遇到空泛的总结、套话或生硬的段落，我会圈出具体哪里有 AI 味，自己动手改，也会请合适的写作或润色 Skill 帮我试几种写法。改完再通读一遍，由我决定标题和正文是否可以定稿。

确定标题和正文后，为文章在 `articles/` 下建一个独立子目录，正文保存为 Markdown。下面是完成配图和草稿准备后的示例目录。

```text
articles/
└── example-article/
    ├── article.md
    ├── AI-generated-images/
    │   ├── cover.png
    │   └── opening.png
    └── wechat-draft-package/
        ├── article.json
        └── article.md
```

文章目录名和图片文件名以实际输出为准。导入用的 Markdown 会标好每张插图的位置，`article.json` 保存标题、导语和封面路径。交接包通常直接引用已经生成的图片。

## 3. 生成封面和正文插图

### 配置生图 API

我用 APIMart 调用 GPT Image 2。先在 APIMart 注册账号，再创建 API Key，并从平台获取 API 基础地址（Base URL）。 [APIMart 的 GPT Image 2 页面](https://apimart.ai/zh/model/gpt-image-2)

在 `universal-apimart-images-skill` 根目录，将 `.secrets.example.json` 复制为本机的 `.secrets.json`。把基础地址和 Key 分别填入 `profiles.primary` 下的 `api_base` 和 `api_key`，确认 `active_profile` 为 `primary`。已有配置时直接编辑 `.secrets.json`。 [配图 Skill 的安装和配置](https://github.com/JacksonYang11/universal-apimart-images-skill#使用前准备)

换平台时，在同一处更新 `api_base` 和 `api_key` 即可。当前版本的 Skill 要求服务支持 `gpt-image-2`，并兼容脚本使用的生图提交接口 `POST /v1/images/generations` 和任务查询接口 `GET /v1/tasks/{task_id}`。

### 用 Agent 生成配图

让 Codex、DeepSeek Harness 或 Trae Agent 读取本篇定稿和配图仓库的 `SKILL.md`。

> 请用 universal-apimart-images-skill 给这篇定稿拟一段导语，规划封面、开篇图和需要的正文插图。先给我看导语和生图提示词，确认后再生成。图片保存到文章目录下。

Agent 根据文章内容写提示词，确认后调用生图模型，再把生成的图片下载到 `AI-generated-images/`。生成另一组时，目录可能是 `AI-generated-images1/` 等。用实际输出目录里的预览挑图，确定封面和每张插图的位置。

## 4. 导入微信公众号草稿箱

第一次导入前，在对应公众号的微信开发者平台完成以下配置。 [微信开发者平台](https://developers.weixin.qq.com/platform/)

- 准备公众号的 **AppID** 和 **AppSecret**，按导入 Skill 的说明把凭证保存在本机。
- 确认账号具备获取接口调用凭证、上传正文图片和封面素材、创建或更新及读取草稿所需的接口权限。
- 把运行导入工具时微信看到的**公网出口 IP**加入 **API IP 白名单**。家庭宽带和手机热点的公网出口可能变化。遇到白名单报错时，按工具返回的 IP 更新后重试。

配置完成后，让 Agent 用导入 Skill 整理定稿和选中的图片。工具可先生成本地预览，再调用微信官方接口上传图片、写入草稿并回读核验。正文插图按 Markdown 中的位置插入，封面和导语各自进入对应字段，作者可读取本机默认设置或使用这篇文章单独指定的值。 [导入 Skill 的安装、配置和用法](https://github.com/JacksonYang11/WeChat-Draft-Importer-skill)

> 请用 wechat-draft-importer 把这篇定稿和选好的配图导入我的公众号草稿箱。沿用已确定的标题，按指定位置插入正文图片，把封面和导语填到对应字段。先检查本地预览，导入后返回草稿标题和 `media_id`。

## 5. 在公众号后台检查并发布

在后台打开这篇草稿，看一遍标题、排版、封面、插图和导语。按账号功能和单篇需要，设置原创声明、赞赏、合集以及创作来源。确认无误后，在后台选择立即发布或定时发布。
