# 人格对话（Persona Chat）

一个自用、不上架的 Android APK：在手机上导入人格 skill，填入 DeepSeek API Key 后直接对话。

**没有内置任何人格。** 首次启动是空白的，需要你自己导入 `.md` / `.txt` / `.json` / `.zip` 人格包。

## 功能

- 手机系统文件选择器导入人格 skill，不需要电脑
- 支持 `SKILL.md`、`.md`、`.txt`、`.json` 和 `.zip`
- ZIP 包自动读取 `SKILL.md`、`agents/openai.yaml` 显示名、`references/*.md` 参考资料
- 多个人格本地保存、切换、删除
- **每个人格独立保存对话**；切换人格不会清空记录
- **每个角色可单独设置头像**：点头像或设置里的「更换当前人格头像」
- **每个角色可单独设置聊天背景图**：设置里的「更换当前人格背景」
- DeepSeek 流式对话，可停止生成
- **深度思考开关**：输入框左侧「深」按钮，开启后使用 thinking + high effort
- 基础 Markdown 渲染（粗体、代码块、列表、链接）
- 对话历史本地保存，退出后可恢复
- API Key 使用 Android Keystore AES/GCM 加密保存
- 无服务器、无账号、无云同步，直连 `api.deepseek.com`

## 不包含

- 执行 skill 里的 Python/JS 脚本或工具调用
- 聊天图片、语音、文件上传
- 同一角色的多会话管理
- iOS、应用商店上架

## 安装

1. 把 `PersonaChat-1.3.apk` 传到 Android 手机。
2. 在文件管理器里点开 APK。
3. 如果系统提示，允许“安装未知应用”。
4. 安装后打开「人格对话」。

要求 Android 8.0（API 26）及以上。

### 从 1.0 / 1.1 / 1.2 升级

保持同一个包名和签名，直接覆盖安装即可。1.0 的单条历史记录会自动迁移到升级时当前选中的人格；1.1 的独立对话、头像和 1.2 的背景都会保留。

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

## 更新日志

见 [CHANGELOG.md](CHANGELOG.md)。

## 实现说明

- 纯 Java + Android 平台控件，没有引入 AndroidX/Compose 依赖。
- `SkillParser` 不依赖 Android，可单独做解析烟雾测试。
- 深度思考默认关闭；打开后请求 `thinking: enabled` + `reasoning_effort: high`，只展示最终回复，不把思维链写进对话历史。
- 每个人格使用独立的 SharedPreferences 历史键；1.0 的旧 `history` 键会在首次加载时迁移。
- 上下文最多带最近约 24,000 字符，避免历史无限增长。
- 头像和背景优先打开系统相册/照片选择器；设备没有可用相册时自动退回文件选择器。
- 头像压缩为 512px JPEG 后保存在 App 私有目录。
- 背景图同样从相册或文件选择器导入，压缩为最长边 1600px JPEG；聊天区叠加半透明遮罩保证文字可读。
