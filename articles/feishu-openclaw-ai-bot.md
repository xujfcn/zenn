---
title: "飛書(Feishu/Lark)にAIチャットボットを一発デプロイ — CrazyRouter + OpenClaw"
emoji: "🤖"
type: "tech"
topics: ["ai", "chatbot", "feishu", "openclaw"]
published: true
---

# 手把手教你用 CrazyRouter + OpenClaw 一键部署飞书 AI 机器人

> 一条命令部署，10 分钟配置飞书，你的企业就拥有了能调用 300+ AI 模型的智能助手。

## 前言

很多企业已经在用飞书作为日常办公工具，但想在飞书里接入 AI 能力（ChatGPT、Claude、Gemini 等），往往面临这些痛点：

- 🔑 每个模型都要单独注册、单独付费
- 🌐 很多 AI 服务需要科学上网
- 🔧 自建 AI 机器人需要写大量代码
- 💰 ChatGPT Plus $20/月，Claude Pro $20/月，加起来太贵

**现在有了更好的方案：** CrazyRouter + OpenClaw，一条命令部署，飞书里直接对话 300+ AI 模型，按量付费，大多数团队每月 $5-15 就够了。

## 整体方案

```
飞书群聊/私聊 → 飞书机器人 → OpenClaw → CrazyRouter API → 300+ AI 模型
                                                            ├── GPT-4o / GPT-4.5
                                                            ├── Claude Opus 4 / Sonnet 4
                                                            ├── Gemini 2.5 Pro
                                                            ├── DeepSeek R1
                                                            └── 更多...
```

你需要准备：

| 准备项 | 说明 | 获取方式 |
|--------|------|---------|
| 一台服务器 | Linux，1GB 内存即可 | 推荐 [Vultr](https://vultr.com) $5/月 VPS |
| CrazyRouter API Key | 统一访问 300+ AI 模型 | [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=feishu_bot) 注册获取 |
| 飞书企业管理员权限 | 创建和审批应用 | 飞书管理后台 |

## 第一步：一条命令部署 OpenClaw

SSH 登录你的服务器，执行：

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/install.sh | bash
```

脚本会自动完成：
- ✅ 安装 Node.js 22+
- ✅ 安装 OpenClaw
- ✅ 配置 CrazyRouter API Key
- ✅ 设置系统服务（开机自启）
- ✅ 启动 OpenClaw 网关

全程 30 秒，中英双语提示。

安装完成后，脚本会自动生成访问地址，按提示打开即可进入管理界面。

## 第二步：创建飞书应用

### 2.1 进入开发者后台

前往 [飞书开放平台](https://open.feishu.cn/)，点击右上角 **开发者后台**

![进入开发者后台](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/01_open_platform.jpg)

### 2.2 创建企业自建应用

在「企业自建应用」页面，点击 **创建企业自建应用**

![创建企业自建应用](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/02_create_app.jpg)

填写应用信息：
1. **应用名称**：自定义（如 OpenClaw、AI助手）
2. **应用描述**：如"OpenClaw AI 聊天机器人"
3. 点击 **创建**

![填写应用信息](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/03_app_info.jpg)

### 2.3 添加机器人能力

在应用设置左侧，点击 **添加应用能力**，找到 **机器人**，点击 **+ 添加**

![添加机器人能力](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/04_add_bot.jpg)

### 2.4 获取凭据

点击左侧 **凭证与基础信息**，复制以下信息（后面配置要用）：
- **App ID**（以 `cli_` 开头）
- **App Secret**

![获取凭据](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/05_credentials.jpg)

## 第三步：配置应用权限

### 3.1 开通即时通讯权限

点击左侧 **开发配置** → **权限管理**，点击 **开通权限**

![权限管理](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/06_permissions.jpg)

在弹出窗口中：
1. 搜索 `im:`
2. 点击 **全部** 勾选所有即时通讯权限（55 个）
3. 点击 **确认开通权限**

![开通即时通讯权限](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/07_im_permissions.jpg)

### 3.2 开通其他必要权限

同样方式搜索并开通：
- **通讯录权限**：获取通讯录基本信息
- **多维表格权限**：多维表格读写（可选，支持读写飞书多维表格）
- **云空间权限**：云空间文件读写（可选，支持读写飞书文档）

## 第四步：发布应用

### 4.1 创建版本

点击顶部提示栏中的 **创建版本**

![创建版本](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/08_create_version.jpg)

填写版本信息：
1. **应用版本号**：`1.0.0`
2. 移动端/桌面端默认能力：选择 **机器人**
3. **更新说明**：如"创建机器人，开通权限等"

![版本详情](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/09_version_detail.jpg)

### 4.2 保存并发布

设置可用范围，点击 **保存**

![保存版本](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/10_save.jpg)

点击 **申请线上发布**

![申请线上发布](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/11_publish.jpg)

### 4.3 审批

登录飞书客户端，在消息中找到审批通知，点击 **同意**

![飞书客户端审批](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/12_approve.jpg)

审批通过，应用发布成功 ✅

![应用已成功发布](https://raw.githubusercontent.com/xujfcn/images/main/feishu_bot/13_published.jpg)

## 第五步：配置事件回调

回到飞书开放平台后台，进行最关键的一步——让机器人能接收消息。

### 5.1 设置订阅方式

点击左侧 **事件与回调**，选择 **使用长连接接收事件**

> 💡 长连接模式不需要公网回调地址，适合大多数部署场景。

### 5.2 添加事件

点击 **添加事件**，添加 **接收消息 v2.0**（`im.message.receive_v1`）

这样当有人 @机器人 或私聊机器人时，OpenClaw 就能收到消息。

### 5.3 重新发布

添加事件后需要重新创建版本并发布（重复第四步），审批通过后生效。

## 第六步：连接 OpenClaw

### 6.1 在 OpenClaw 中配置飞书

在 OpenClaw 管理面板或配置文件中，填入飞书应用的凭据：

```yaml
# OpenClaw 飞书配置
feishu:
  app_id: cli_xxxxxxxxxx      # 第二步获取的 App ID
  app_secret: xxxxxxxxxx       # 第二步获取的 App Secret
```

### 6.2 启动飞书连接

```bash
openclaw gateway restart
```

看到类似日志说明连接成功：

```
[feishu] Connected via WebSocket
[feishu] Bot ready: OpenClaw
```

## 第七步：开始对话！

在飞书中找到你的机器人，直接发消息：

- **私聊机器人**：直接发消息即可
- **群聊中使用**：@机器人 + 你的问题

机器人通过 CrazyRouter 调用 AI 模型回复，支持：
- 💬 多轮对话（自动维护上下文）
- 🔄 切换模型（GPT-4o、Claude、Gemini 等）
- 📎 发送图片让 AI 分析（Vision 能力）
- 📄 读写飞书文档和多维表格
- 🔍 联网搜索

## 为什么选择 CrazyRouter？

| 对比项 | 直接用 OpenAI | CrazyRouter |
|--------|-------------|-------------|
| 模型数量 | 仅 OpenAI 系列 | 300+ 模型 |
| 价格 | 官方原价 | 低至官方 55% |
| 切换模型 | 换 API Key | 同一个 Key |
| 网络要求 | 需科学上网 | 国内直连 |
| 付费方式 | 按量预付 | 按量付费，充值即用 |

👉 注册 CrazyRouter：[crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=feishu_bot)

## 常见问题

### Q: 机器人不回复消息？

检查以下几点：
1. 应用是否已发布（状态为"已启用"）
2. 权限是否开通（特别是 `im:` 开头的权限）
3. 事件回调是否配置（接收消息事件）
4. OpenClaw 服务是否运行：`openclaw gateway status`

### Q: 支持哪些飞书功能？

- ✅ 私聊对话
- ✅ 群聊 @机器人
- ✅ 发送/接收图片
- ✅ 读写飞书文档
- ✅ 读写多维表格
- ✅ 云空间文件操作

### Q: 费用怎么算？

- 服务器：$5/月（最低配置即可）
- AI 模型调用：按量付费，通过 CrazyRouter 比官方便宜 45%
- 飞书：企业版免费功能即可，无额外费用

### Q: 可以在多个群里同时使用吗？

可以。一个机器人可以被添加到多个群聊，每个群独立维护对话上下文。

## 总结

整个部署流程：

1. **30 秒**：一条命令部署 OpenClaw
2. **5 分钟**：创建飞书应用、配置权限
3. **2 分钟**：发布应用、审批
4. **1 分钟**：连接 OpenClaw
5. **开始用**：飞书里直接和 AI 对话

从此你的飞书就是一个全能 AI 助手，背后有 300+ 模型随时切换，成本只有 ChatGPT Plus 的零头。

---

📌 **相关链接**：
- CrazyRouter 官网：[crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=feishu_bot)
- OpenClaw 文档：[docs.openclaw.ai](https://docs.openclaw.ai)
- OpenClaw GitHub：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- 一键部署脚本：[github.com/xujfcn/crazyrouter-openclaw](https://github.com/xujfcn/crazyrouter-openclaw)
