# AstrBot 伪装聊天插件 — 开发备忘

## 插件概念
- 群聊中触发命令后，以 QQ 合并转发消息的形式展示一段伪装聊天
- 故事以多人聊天记录的形式呈现：主角讲述 + 围观网友穿插评论/表情
- 效果类似 QQ 转发别人的聊天记录，点开是一个完整的"对话场景"

## 核心 API

### 合并转发消息 — send_group_forward_msg
- OneBot v11 扩展接口，NapCat 支持
- 通过 aiocqhttp 调用：`await event.bot.send_group_forward_msg(group_id=gid, messages=[...])`
- 每条消息是一个 node 消息段：

```python
# OneBot v11 标准格式（推荐使用这个）
{
    "type": "node",
    "data": {
        "user_id": "10001000",  # QQ号，决定头像显示（真实QQ号=真实头像，假号=默认灰色头像）
        "nickname": "某人",      # 显示昵称，完全自定义，随便填
        "content": "消息内容"    # 支持字符串或消息段数组（文本、图片、表情等）
    }
}

# go-cqhttp 扩展格式（NapCat 也兼容）
{
    "type": "node",
    "data": {
        "name": "显示昵称",   # 同 nickname
        "uin": "10001",       # 同 user_id
        "content": "消息内容"
    }
}
```

> **头像机制**：头像不能通过传图片 URL 自定义，而是通过 `user_id`/`uin` 关联 QQ 头像系统。
> 填真实 QQ 号 → 显示该号的头像；填不存在的号 → 默认灰色头像。

- content 支持 CQ 码和消息段数组两种格式
- 支持嵌套图片：`[{"type": "image", "data": {"file": "url"}}]`
- 支持表情：`[{"type": "face", "data": {"id": "176"}}]`
- 注意：不支持转发套娃（合并转发里再嵌合并转发）

### AstrBot 框架层长消息处理（参考，本插件不依赖）
- `t2i`: 文转图，超过 `t2i_word_threshold`（默认150字）自动渲染为图片
- `forward_threshold`: 超过1500字自动折叠为合并转发（仅QQ平台）
- 本插件直接用 send_group_forward_msg，不走框架的自动转发

## AstrBot 插件开发框架

### 基础结构
- 主类继承 `Star`（from `astrbot.api.star`）
- 注册插件：`@register("插件名", "作者", "描述", "版本", "仓库URL")`
- 命令注册：`@filter.command("命令名")` 
- 消息监听：`@filter.on_message()` 可按平台过滤 `platform=["qq"]`
- 所有 handler 都是 `async def`，用 `yield event.plain_result()` 返回响应
- 多条消息用 `await event.send(MessageChain([...]))` 发送

### 插件目录结构
```
addons/
└── astrbot_plugin_sadstory/
    └── main.py
```

### LLM 调用方式
- 通过 `self.context.tool_loop_agent()` 调用 LLM（支持 function calling）
- 也可以自定义 `FunctionTool` 注册为 LLM 工具
- 示例：
```python
llm_resp = await self.context.tool_loop_agent(
    event=event,
    chat_provider_id=await self.context.get_current_chat_provider_id(event.unified_msg_origin),
    prompt="你的 prompt",
    tools=ToolSet([]),
    max_steps=5,
    tool_call_timeout=30
)
result = llm_resp.completion_text
```

### Bot API（通过 event.bot 访问，OneBot v11）
- `send_group_msg(group_id, message)` — 发群消息
- `send_private_msg(user_id, message)` — 发私聊
- `send_group_forward_msg(group_id, messages)` — 发合并转发（本插件核心）
- `get_stranger_info(user_id)` — 获取用户信息
- `get_group_member_info(group_id, user_id)` — 获取群成员信息
- `get_group_member_list(group_id)` — 获取群成员列表（用于读取素材群用户）

## 参考文档

### AstrBot
- 官方文档首页: https://docs.astrbot.app/
- 插件开发: https://docs.astrbot.app/dev/
- HTTP API: https://docs.astrbot.app/dev/openapi.html
- 配置文件详解: https://docs.astrbot.app/en/dev/astrbot-config.html
- API 交互式文档: https://docs.astrbot.app/scalar.html

### OneBot v11
- 协议规范: https://github.com/botuniverse/onebot-11
- 合并转发相关讨论: https://github.com/Mrs4s/go-cqhttp/discussions/926

### NapCat
- 官方文档: https://napneko.github.io/
- 插件 API 类型定义: https://napneko.github.io/develop/plugin/api/type

## 设计要点

### 角色数据来源（两种模式，可共存）

#### 模式 A：从素材群读取真实用户
- 把机器人拉到一个专门的"素材群"
- 插件通过 `get_group_member_list(group_id)` 获取群成员列表
- 每个成员返回 `user_id`（QQ号）、`nickname`（昵称）、`card`（群名片）
- 头像自动通过 `user_id` 关联，不需要额外获取
- 后台配置：
  - `source_group_id`: 素材群群号
  - `include_users`: 白名单（只用这些人），为空则用全部成员
  - `exclude_users`: 黑名单（排除这些人，比如机器人自己）
  - `use_card_as_name`: 是否优先使用群名片作为显示昵称（默认 true）

#### 模式 B：手动配置虚拟用户
- 在配置文件中手动定义一批角色
- 示例：
```json
{
    "custom_users": [
        {"nickname": "路过的小明", "user_id": "123456789"},
        {"nickname": "吃瓜群众", "user_id": "987654321"},
        {"nickname": "深夜emo的人", "user_id": "111222333"}
    ]
}
```

### 角色分工
- 主角（讲故事的人）— 从用户池中随机选一个，贯穿整个故事
- 围观网友 — 从用户池中随机选 3-5 个，穿插评论、表情、吐槽
- 每次生成故事时随机分配角色

### 故事来源 — LLM 生成
- 使用 AstrBot 框架的 `self.context.tool_loop_agent()` 调用 LLM
- Prompt 要求 LLM 输出结构化的对话格式（JSON 数组）
- 每条消息控制在 1-3 句话，短小精悍，适合合并转发阅读体验
- 总消息条数控制在 10-30 条
- Prompt 示例思路：
  - 给定主角名和围观网友名
  - 要求输出 JSON 数组，每条包含 `speaker`（角色标识）和 `content`（台词）
  - 主角讲述故事主线，网友穿插反应（共情、吐槽、表情）
  - 故事风格：伤感、治愈、青春回忆等

### 数据存储 — JSON 文件（轻量方案）
- 不需要真正的数据库
- `data/sadstory_config.json` — 插件配置（素材群号、用户白名单/黑名单、自定义用户等）
- `data/sadstory_cache.json` — 可选，缓存已生成的故事，避免重复调用 LLM
- 用户池数据可以在插件启动时加载，运行时缓存在内存中

### 群白名单
- 配置项 `allowed_groups`: 允许触发伪装聊天的群号列表
- 为空则所有群都可以触发（不推荐）
- 只有在白名单内的群发送命令才会响应，其他群静默忽略

### 可能的命令
- `/伪装聊天` 或 `/sadstory` — 随机一个故事
- `/伪装聊天 主题` — 指定主题生成
- `/sadstory_reload` — 重新加载素材群用户列表（管理员命令）
- 可以考虑加冷却时间防刷屏

### 注意事项
- user_id 填真实QQ号显示真实头像，填假号显示默认灰色头像
- 合并转发消息在手机端和PC端展示效果略有不同
- NapCat 对 send_group_forward_msg 的支持可能有版本差异，需实测
- 消息条数不宜过多，建议 10-30 条为宜
- LLM 生成的内容需要做格式校验，防止 JSON 解析失败时插件崩溃
- 素材群用户列表建议启动时缓存，不要每次触发都请求 API

## 待确认 / TODO
- [ ] 确认 AstrBot 中 `tool_loop_agent` 的具体参数和返回值（需实测或查源码）
- [ ] 设计 LLM prompt 模板，确保输出格式稳定可解析
- [ ] 测试 NapCat 对 `get_group_member_list` 的返回字段
- [ ] 确定配置文件的存放路径和加载方式（AstrBot 插件的 data 目录）
