# 🌧️ AstrBot 伤感故事插件

在 QQ 群聊中以**合并转发消息**的形式展示一段伤感故事，模拟真实群聊中有人连续讲故事、围观网友穿插评论的效果。

## ✨ 功能特点

- 📖 通过 LLM 生成原创伤感故事，每次都不一样
- 💬 合并转发形式呈现，像真实的群聊记录一样
- 🎨 Prompt 风格可自定义：口语化、文学风格等，后台直接编辑，支持多风格随机
- 🎭 支持从指定群读取真实用户作为故事角色（昵称+头像）
- 👥 也支持手动配置虚拟角色
- 📝 支持导入故事模板作为 LLM 参考素材，生成风格更贴近真实
- 🤖 支持自定义 LLM 模型
- 🔒 仅管理员可触发，支持冷却时间

## 📦 安装

在 AstrBot WebUI 中搜索 `astrbot_plugin_sadstory` 安装，或手动克隆：

```bash
cd AstrBot/data/plugins
git clone https://github.com/Towqs/astrbot_plugin_sadstory.git
```

## 🚀 使用方法

| 命令 | 说明 | 权限 |
|------|------|------|
| `/sadstory` | 随机生成一个伤感故事 | 管理员 |
| `/sadstory 主题` | 指定主题生成故事（如 `/sadstory 校园暗恋`） | 管理员 |
| `/sadstory @某人` | 指定某人作为故事主讲人 | 管理员 |
| `/sadstory_reload` | 重新加载素材群用户列表 | 管理员 |
| `/sadstory_style` | 查看当前生成参数和 Prompt 风格列表 | 管理员 |
| `/sadstory_addstyle` | 添加新的 Prompt 风格（换行后跟内容） | 管理员 |
| `/sadstory_usestyle 序号` | 切换 Prompt 风格的启用/禁用 | 管理员 |
| `/sadstory_listtpl` | 查看所有故事模板 | 管理员 |
| `/sadstory_addtpl` | 添加故事模板（换行后跟内容） | 管理员 |
| `/sadstory_usetpl 序号` | 切换故事模板的启用/禁用 | 管理员 |
| `/sadstory_deltpl 模板名` | 删除指定故事模板 | 管理员 |

## ⚙️ 配置说明

安装后在 WebUI → 插件配置中设置：

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| 素材群群号 | 机器人所在的群，用于读取成员作为角色 | 空 |
| 素材群用户白名单 | 只用这些 QQ 号，逗号分隔 | 空（全部） |
| 素材群用户黑名单 | 排除这些 QQ 号（如机器人自己） | 空 |
| 优先使用群名片 | 角色昵称优先用群名片还是 QQ 昵称 | 开启 |
| 自定义虚拟用户 | 格式：`昵称1:QQ号1,昵称2:QQ号2` | 空 |
| 群白名单 | 允许触发的群号，逗号分隔，留空则所有群 | 空 |
| 冷却时间（秒） | 同群两次触发的最短间隔 | 60 |
| 故事最少/最多消息条数 | 控制故事长度 | 30 / 80 |
| 围观网友数量 | 穿插评论的网友数 | 3 |
| LLM 模型 | 指定 LLM 提供商 ID，留空用默认 | 空 |

## 📝 故事模板

插件支持导入故事模板作为 LLM 的参考素材。将 `.txt` 文件放入插件目录下的 `templates/` 文件夹即可：

```
astrbot_plugin_sadstory/
├── templates/
│   ├── 故事模板-01.txt
│   ├── 故事模板-02.txt
│   └── ...
├── main.py
├── metadata.yaml
└── _conf_schema.json
```

每次生成故事时，插件会随机选取一个模板注入到 prompt 中，让 LLM 模仿其叙事风格（碎片化、口语化、真实群聊感），但创作全新的故事内容。

## 🎨 Prompt 风格

插件内置口语化和文学两种 LLM prompt 风格，可在 WebUI 后台「Prompt 风格模板列表」中直接查看和编辑。

- 支持添加多个自定义风格，生成时从已启用的风格中随机选取
- prompt 中可使用变量：`{protagonist}`（主角名）、`{bystanders}`（网友名）、`{min_msg}`/`{max_msg}`（消息条数）、`{theme_line}`（主题）、`{reference_section}`（参考模板）、`{emoji_instruction}`（表情说明）
- 也可在群内用 `/sadstory_style` 查看、`/sadstory_addstyle` 添加、`/sadstory_usestyle` 切换

## 🎭 角色来源

角色（昵称 + 头像）有两种来源，可以共存：

1. **素材群读取**：把机器人拉到一个群里，配置群号后插件自动读取成员信息。头像通过 QQ 号自动关联。
2. **手动配置**：在 WebUI 中填写 `昵称:QQ号` 格式的自定义用户。填真实 QQ 号会显示对应头像，填假号显示默认灰色头像。

## 📋 依赖

- AstrBot 4.x+
- QQ 平台适配器（aiocqhttp）
- NapCat 或其他支持 `send_group_forward_msg` 的 OneBot v11 实现
- 已配置 LLM 服务提供商

## 📝 更新日志

### v0.3.4
- 新增 Prompt 风格模板管理系统，后台可直接编辑口语化/文学等 prompt 风格
- 新增 `/sadstory_style`、`/sadstory_addstyle`、`/sadstory_usestyle` 指令
- 支持多风格随机选取，无启用风格时自动回退内置默认

### v0.2.3
- 修复头像和昵称显示问题：未配置素材群时自动从当前群拉取真实成员
- 修复合并转发 content 格式为消息段数组，符合 NapCat 规范

### v0.2.2
- 彻底修复群消息识别问题：改用 `AiocqhttpMessageEvent` 类型替代通用 `AstrMessageEvent`
- 使用 `event.get_group_id()` 原生方法获取群号，不再手动解析
- 参考 astrbot_plugin_qqadmin 插件的正确实现方式

### v0.2.1
- 尝试修复群号提取（未生效，因为 AstrMessageEvent 不支持 group_id）

### v0.2.0
- 新增故事模板系统，支持从 `templates/` 目录加载参考素材
- 新增自定义 LLM 模型配置
- WebUI 可视化配置（`_conf_schema.json`）

### v0.1.0
- 初始版本，基础伤感故事生成与合并转发发送

## 📄 许可

MIT License
