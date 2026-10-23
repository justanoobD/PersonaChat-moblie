[README.md](https://github.com/user-attachments/files/33874250/README.md)

# 人格对话（Persona Chat）

一个自用、不上架的 Android APK：在手机上导入人格 skill，填入 DeepSeek API Key 后直接对话。

**没有内置任何人格。** 首次启动是空白的，需要你自己导入 `.md` / `.txt` / `.json` / `.zip` 人格包。

## 功能

- 手机系统文件选择器导入人格 skill，不需要电脑
- 支持 `SKILL.md`、`.md`、`.txt`、`.json` 和 `.zip`
- ZIP 包自动读取 `SKILL.md`、`agents/openai.yaml` 显示名、`references/*.md` 参考资料
- 多个人格本地保存、切换、删除
- DeepSeek 流式对话，可停止生成
- 基础 Markdown 渲染（粗体、代码块、列表、链接）
- 对话历史本地保存，退出后可恢复
- API Key 使用 Android Keystore AES/GCM 加密保存
- 无服务器、无账号、无云同步，直连 `api.deepseek.com`

## 不包含

- 执行 skill 里的 Python/JS 脚本或工具调用
- 图片、语音、文件上传
- 多会话管理
- iOS、应用商店上架

## 安装

1. 把 `PersonaChat-1.0.apk` 传到 Android 手机。
2. 在文件管理器里点开 APK。
3. 如果系统提示，允许“安装未知应用”。
4. 安装后打开「人格对话」。

要求 Android 8.0（API 26）及以上。

## 首次使用

1. 点右上角「设置」。
2. 填入 DeepSeek API Key（在 DeepSeek 开放平台创建）。
3. 模型默认 `deepseek-flash`；也可以改成 `deepseek-v4-pro`。
4. 点「导入人格 skill」，选择手机里的 `.md` / `.zip` 文件。
5. 导入成功后回到对话页，直接输入消息。

> API Key 只保存在本机，不会写进 APK，也不会经过第三方服务器。请求直接发往 DeepSeek。

## 导入格式

### 单文件

- `.md` / `.txt`：可带 YAML frontmatter，正文作为人格提示词。
- `.json`：支持 `name`、`description`、`prompt` / `system_prompt` / `instructions` 字段。

### ZIP 包

推荐结构：

```text
my-persona/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    └── canon.md
```

- 优先读取根目录或任意子目录的 `SKILL.md`。
- `name`、`description` 从 YAML frontmatter 读取。
- `agents/openai.yaml` 的 `display_name` 用作显示名。
- `references/*.md`、`references/*.txt` 会合并进系统提示词；其他脚本文件会保留在包里，但不会执行。

## 构建

环境：JDK 17、Android SDK（compileSdk 35、build-tools 35.0.0）。项目自带 Gradle 8.9 wrapper。

```powershell
$env:JAVA_HOME = "C:\Program Files (x86)\Android\openjdk\jdk-17.0.14"
$env:ANDROID_HOME = "C:\Program Files (x86)\Android\android-sdk"
.\gradlew.bat assembleRelease
```

输出：

```text
app/build/outputs/apk/release/app-release.apk
```

签名文件：项目根目录的 `keystore.jks`，alias `personachat`，store/key password 均为 `persona123`。这是自用签名，不需要上架；请妥善保存，后续升级 APK 需要同一个签名。源码分发包未放入签名密钥，如需覆盖安装请保留原工程的 `keystore.jks`。

## 实现说明

- 纯 Java + Android 平台控件，没有引入 AndroidX/Compose 依赖。
- `SkillParser` 不依赖 Android，可单独做解析烟雾测试。
- DeepSeek 请求默认关闭思考模式（`"thinking": {"type": "disabled"}`），让角色对话回复更快。
- 上下文最多带最近约 24,000 字符，避免历史无限增长。

