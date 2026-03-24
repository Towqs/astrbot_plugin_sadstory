# 更新日志

## v0.2.2
- 彻底修复群消息识别问题：改用 `AiocqhttpMessageEvent` 替代通用 `AstrMessageEvent`
- 使用 `event.get_group_id()` 原生方法获取群号
- 参考 astrbot_plugin_qqadmin 插件的正确实现方式

## v0.2.1
- 尝试修复群号提取（未生效）

## v0.2.0
- 新增故事模板系统，支持从 `templates/` 目录加载参考素材
- 新增自定义 LLM 模型配置
- WebUI 可视化配置（`_conf_schema.json`）

## v0.1.0
- 初始版本，基础伤感故事生成与合并转发发送
