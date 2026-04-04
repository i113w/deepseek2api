# DeepSeek2API (Enhanced Fork)

[![](https://img.shields.io/github/license/iidamie/deepseek2api.svg)](LICENSE)
![](https://img.shields.io/github/stars/iidamie/deepseek2api.svg)
![](https://img.shields.io/github/forks/iidamie/deepseek2api.svg)

> 本 Fork 专注于增强消息预处理机制，实现特殊 Token 注入以优化长对话上下文处理。

## ✨ 新增功能

### 特殊 Token 注入机制

本 Fork 在原版基础上实现了 **消息预处理增强**，通过注入特定控制 Token 优化模型对长对话历史的理解，并避免部分情况下对模型输出的截断处理。

#### 核心改动

| 改动项 | 说明 | 适用场景 |
|--------|------|----------|
| `system` 消息转换 | 将 `system` 角色消息转换为 `user` 角色，并包裹于 `<system_instructions>` 标签中 | 确保系统指令在多轮对话中稳定传递，避免因角色切换导致指令丢失 |
| Reasoner 模型专属 Token | 当 `model` 参数为 `deepseek-reasoner` / `deepseek-r1` 系列时，在历史 `assistant` 消息中自动注入 `<｜end▁of▁thinking｜>` Token | 告知后端历史轮次的推理过程已结束 |

#### 技术原理简述

```python
# system 消息处理示例
# 输入: {"role": "system", "content": "你是一个助手"}
# 输出: <system_instructions>你是一个助手</system_instructions>

# reasoner 模型 assistant 消息处理示例  
# 输入: <｜Assistant｜>推理内容<｜end▁of▁sentence｜>
# 输出: <｜Assistant｜><｜end▁of▁thinking｜>推理内容<｜end▁of▁sentence｜>
```

#### 使用方式

无需额外配置，功能自动生效：

```bash
# 普通对话（自动应用 system 消息转换）
curl -X POST http://localhost:5001/v1/chat/completions \
  -H "Authorization: Bearer your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "messages": [
      {"role": "system", "content": "你是一个专业助手"},
      {"role": "user", "content": "你好"}
    ]
  }'

# Reasoner 模型对话（自动注入 thinking 结束标记）
curl -X POST http://localhost:5001/v1/chat/completions \
  -H "Authorization: Bearer your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-reasoner",
    "messages": [
      {"role": "user", "content": "请逐步推理：1+1=?"}
    ]
  }'
```

#### 兼容性说明

- ✅ 完全兼容原版接口规范与请求格式
- ✅ `model` 参数未传入时自动降级为默认行为
- ✅ 不影响非 Reasoner 模型的正常对话流程
- ⚠️ 特殊 Token 为后端内部协议，请勿在用户输入中手动构造

---

## 🔗 灵感来源

本功能的实现灵感来源于 **SoliUmbra** 开发的 `ShallowSeek` JavaScript 脚本，感谢其在 DeepSeek 协议逆向与消息构造方面的开创性工作。

> 本 Fork 仅参考其核心思想，代码实现完全独立，未直接复用任何源代码。

---

## ⚠️ 免责声明

**逆向API是不稳定的，建议前往DeepSeek官方 https://platform.deepseek.com/ 付费使用API，避免封禁的风险。**

**本组织和个人不接受任何资金捐助和交易，此项目是纯粹研究交流学习性质！**

**仅限自用，禁止对外提供服务或商用，避免对官方造成服务压力，否则风险自担！**

**仅限自用，禁止对外提供服务或商用，避免对官方造成服务压力，否则风险自担！**

**仅限自用，禁止对外提供服务或商用，避免对官方造成服务压力，否则风险自担！**

---

## 📦 快速部署（精简指引）

### Docker 部署

```shell
# 1. 准备 config.json（参考原项目文档）
# 2. 启动容器
docker run -d -p 5001:5001 \
  -v "$(pwd)/config.json:/app/config.json" \
  --name deepseek2api \
  ghcr.io/iidamie/deepseek2api:latest

# 3. 查看日志
docker logs -f deepseek2api
```

### 原生部署

```shell
git clone https://github.com/iidamie/deepseek2api.git
cd deepseek2api
pip install -r requirements.txt
# 配置 config.json 后启动
python app.py
```

> 完整部署文档、接口说明及多账号配置请参考 [原项目 README](https://github.com/iidamie/deepseek2api)。

---

## 📄 许可证

本项目遵循原项目许可证（见 `LICENSE` 文件）。本 Fork 的新增代码同样遵循相同许可证。