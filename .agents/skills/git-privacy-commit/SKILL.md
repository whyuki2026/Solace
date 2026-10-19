---
name: git-privacy-commit
description: 检查待提交的 Git 变更是否存在隐私泄漏、密钥或敏感数据，并生成一条英文 Conventional Commits 提交信息。适用于提交前审查暂存或未暂存改动，以及需要撰写提交信息的场景。
---

# Git 变更隐私审查与提交信息

审查待提交的变更，汇报隐私风险，并给出一条英文提交信息。这个 skill 只做审查：除非用户明确要求，否则不要执行 `git add`、`git commit`、`git push`，也不要修改文件。

用中文汇报结果；建议的提交信息必须使用英文。

## 1. 确定审查范围

同时检查暂存区、工作区和未跟踪文件：

```bash
git status --short
git diff --stat
git diff
git diff --cached
git ls-files --others --exclude-standard
```

- 对每个即将提交的未跟踪文件，都要读取实际内容；`git diff` 不会显示它们。
- 如果工作区干净，直接说明并停止，不要编造变更或提交信息。
- 明确说明审查覆盖的是暂存改动、未暂存改动，还是两者都有。
- 查看 `AGENTS.md`、`CONTRIBUTING.md` 和 `git log --oneline -10` 了解项目约定，但最终提交信息仍使用英文。

## 2. 检查实际内容

审查新增和修改的行，不要只看文件名。发现匹配项时，打开上下文确认。对于二进制或大文件，检查类型并判断是否应该入库。

不要只依赖关键词扫描，它既可能误报，也可能漏报。如果本机已经安装 `gitleaks` 或 `trufflehog`，可以辅助使用；不要为此安装工具，也不要把 diff 上传到外部服务。

## 3. 隐私检查清单

### 密钥与凭据

- API Key、访问令牌、密码、私钥、证书、签名配置、keystore（`*.jks`、`*.keystore`）、`local.properties`、`.env*`、凭据 JSON。
- 高敏感字段名：`apiKey`、`secret`、`token`、`password`、`passwd`、`private_key`、`client_secret`、`authorization`、`Bearer`。
- 区分真实值和明显的占位符，例如 `<YOUR_KEY>`、`example`、`changeme`。

### 个人与机器数据

- 姓名、邮箱、电话、住址、身份证件号、账号或用户 ID。
- 本机绝对路径（`/Users/<name>/`、`C:\Users\<name>\`）、设备序列号、主机名、内网 IP、私有 URL。
- 真实照片、视频、数据库、备份、日志、截图，或包含个人媒体的测试数据。

### 应用行为与权限

- `AndroidManifest.xml` 中新增的权限，尤其是 `INTERNET`、定位、通讯录、相机、麦克风，以及范围过大的媒体或存储权限。
- 分析、崩溃上报、广告等联网 SDK；对外网络请求；埋点；可能暴露媒体路径或 URI 的剪贴板或日志输出。
- 可能把私密媒体复制到设备外的备份或数据提取配置。

### 仓库卫生

- 构建产物、IDE 元数据、缓存、本机 SDK 路径，以及本应被忽略的机器相关文件。
- 误加入的大型二进制媒体或压缩包。

## 4. 核实与汇报

- 对每个匹配项打开上下文，判断是否真的是问题。
- 不要输出完整的密钥、令牌或隐私值。用 `<redacted>` 脱敏，最多显示很短的前缀或后缀。
- 不要在摘要、修复建议或提交信息中重复敏感值。
- 如果某个密钥可能已经进入 Git 历史，要提醒用户：只在新提交中删除还不够，必须轮换密钥；必要时清理历史。

使用以下结构，保持简洁：

```text
## 隐私审查
范围：暂存 | 未暂存 | 两者（另有 N 个未跟踪文件）
结论：通过 | 需注意 | 阻止

发现：
- [高|中|低] 路径:行号 — 类别。风险原因。修复建议。
```

如果没有问题，写：`未在本次审查的变更中发现隐私或密钥风险。`

结论说明：

- **阻止**：存在有效密钥或凭据、私钥、真实个人数据，或明确的隐私退化。提醒用户暂时不要提交。
- **需注意**：可疑值、本机路径、不必要的权限或 SDK，或本应被忽略的文件。
- **通过**：未发现问题。

## 5. 生成提交信息

按 Conventional Commits 格式，给出且只给出一条英文提交信息：

```text
<type>(<scope>): <imperative summary>

<optional body: what changed and why>
```

- 类型：`feat`、`fix`、`refactor`、`perf`、`test`、`docs`、`build`、`ci`、`chore`、`style`。
- 标题：使用祈使语气；冒号后小写；结尾不加句号；不超过 72 个字符。
- Scope：可选，简短小写（例如 `ui`、`gradle`、`media`）。
- 正文：每行不超过 72 个字符；说明动机和重要行为，不要逐文件罗列。
- 根据实际变更生成信息。如果混入了无关改动，应说明建议拆分，并分别给出提交信息。
- 提交信息中绝不能出现密钥、个人数据或隐私审查内容。
- 如果结论是「阻止」，仍然给出信息，但追加一句：`Do not commit until the blocking findings are resolved.`

仅作格式示例：

```text
feat(ui): add Hello Solace home screen
```
