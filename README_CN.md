# 🛡️ Vibe Guard

[English](README.md) | 中文

**社区驱动的 Claude 技能，用于检测 AI 生成代码中的安全后门。**

Vibe coding —— 通过自然语言驱动 AI 代理生成代码 —— 高效、有趣且生产力极高。但它引入了一种独特的威胁面：生成的代码*看起来*正确，通过粗略审查也没问题，但可能包含微妙的后门、数据窃取逻辑或不安全的模式，在快速迭代中极易被人类忽略。

Vibe Guard 是一个开放的、由社区维护的 Claude 安全技能，用于捕获这些威胁。项目提供了一套初始的检测规则和漏洞参考文件，但**它真正的力量来自社区**：每一条由社区贡献的语言模式、CVE 条目和 AI 特有的反模式，都让所有人的 vibe coding 更加安全。

> **这是一个社区优先的项目。** 我们提供框架和流水线 —— 你带来专业知识。无论你是安全研究员、渗透测试工程师，还是一个刚在 AI 生成代码中发现可疑模式的开发者，你的贡献就是 Vibe Guard 的核心驱动力。查看[参与贡献](#参与贡献)了解如何开始。

## 功能特性

- **12 大通用威胁类别**：数据外泄、远程代码执行（RCE）、凭证泄露、供应链攻击、后门认证、代码混淆、文件系统滥用、网络威胁、持久化与隐蔽、密码学弱点、注入攻击、以及 AI/Vibe-Coding 特有风险
- **语言感知检测**：针对 Python、JavaScript/TypeScript、Java、C/C++、Go、Rust 和 PHP 的版本特定漏洞模式 —— [添加你的语言](#1-语言参考文件高影响力)
- **依赖扫描**：覆盖 PyPI、npm 和 Maven 生态系统的已知漏洞通告，支持版本范围匹配 —— [添加你的通告](#2-依赖通告高影响力)
- **双重输出**：行内代码标注 + 结构化安全报告，配合四级严重程度（🔴 严重 / 🟠 高危 / 🟡 中危 / 🔵 低危）
- **自动扫描模式**：可选的代码生成后自动检查功能，可随时开关以节省 token
- **模块化参考文件**：语言和依赖通告以独立参考文件存储 —— 易于更新、扩展和贡献

## 参与贡献

Vibe Guard 的核心能力来自其参考文件 —— 参考文件越全面，能捕获的威胁就越多。当前版本覆盖了最常见的模式，但安全领域广阔且持续演进。**我们需要你的帮助。**

### 如何贡献

#### 1. 语言参考文件（高影响力）

为现有或新增语言添加版本特定的漏洞文件。

**扩展现有语言** —— 例如，Python 目前有 `common.md`、`python2.md`、`python3.0-3.7.md` 和 `python3.8+.md`。你可以添加：
- `python3.12+.md` 覆盖最新版本的安全变更

**添加新语言** —— 我们尚未覆盖：
- Swift / Objective-C（iOS 安全模式）
- Kotlin（Android 特定风险、协程安全）
- C# / .NET（反序列化、Razor 注入、Entity Framework）
- Ruby（ERB 注入、批量赋值、Marshal 反序列化）
- Scala（Akka、Play Framework 模式）
- Shell/Bash（注入、引号处理、通配符攻击）
- Solidity（重入攻击、溢出、抢跑交易）

添加新语言时，请按照现有参考文件的格式创建 `references/languages/<language>/common.md`。

#### 2. 依赖通告（高影响力）

在 YAML 通告文件中添加或更新漏洞条目。

**扩展现有生态系统** —— 向 `pypi/common.yaml`、`npm/common.yaml` 或 `maven/common.yaml` 中添加新披露的 CVE。

**添加分类文件** —— 将大型通告文件按领域拆分：
- `pypi/ml-stack.yaml` —— ML/AI 专用包（torch、transformers、jax 等）
- `pypi/web-frameworks.yaml` —— Web 框架包
- `npm/frontend-frameworks.yaml` —— React、Vue、Angular、Svelte 通告

**添加新生态系统** —— 我们尚未覆盖：
- `cargo/` —— Rust crates（crates.io 通告）
- `nuget/` —— .NET 包
- `gems/` —— Ruby gems
- `packagist/` —— PHP Composer 包
- `go-modules/` —— Go 模块漏洞
- `cocoapods/` —— iOS 依赖
- `pub/` —— Dart/Flutter 包

#### 3. 检测规则（中等影响力）

改进 `SKILL.md` 中的通用检测模式：
- 在现有类别（3.1–3.12）下添加新的威胁子类
- 优化误报处理指引
- 添加框架特定模式（如 Next.js 中间件、Remix loader）

#### 4. AI/Vibe-Coding 特有模式（高影响力）

这是 Vibe Guard 与传统安全扫描器的最大区别。我们特别需要：
- AI 生成代码中包含微妙安全缺陷的真实案例
- LLM 训练数据中常见但不安全的代码模式
- 隐藏在代码注释或文档字符串中的 Prompt 注入技术
- AI 代理频繁产出的"看似正确实则有害"的反模式

### 贡献格式

#### 语言参考文件格式

创建如下结构的 Markdown 文件：

```markdown
# [语言] [版本范围] — 版本特定漏洞

## [严重程度图标] [严重程度]: [简要标题]

[漏洞说明及其危害解释]

\```[language]
// 有漏洞的代码
...

// 安全的替代方案
...
\```

## 多个相关模式可使用表格格式

| 模式 | 风险 | 严重程度 |
|------|------|----------|
| `dangerous_function()` | 可能出现的问题 | 🔴/🟠/🟡/🔵 |
```

#### 依赖通告格式

按以下结构向相关 YAML 文件添加条目：

```yaml
- package: 包名
  affected: ">=1.0,<2.3.1"          # 使用生态系统原生格式的版本范围
  severity: 🔴 critical              # 可选值：🔴 critical、🟠 high、🟡 medium、🔵 low
  cve: CVE-YYYY-NNNNN               # 如未分配 CVE 则填 "N/A"
  description: "漏洞的简要描述"
  fix: "升级到 package-name>=2.3.1"
```

### 贡献准则

1. **具体明确** —— 包含受影响的版本范围、CVE 编号（如有），以及展示漏洞代码和安全替代方案的具体代码示例。
2. **注重可操作性** —— 每个条目都应帮助用户修复问题。仅说"不要这样做"是不够的 —— 需要展示应该怎么做。
3. **减少误报** —— 如果某个模式有合理用途（例如 REPL 中的 `eval()`），请注明它在什么上下文中是真正危险的，什么情况下是可接受的。
4. **保持条目独立** —— 每个漏洞条目应在不阅读其他文件的情况下即可理解。提供足够的上下文，使 Claude 能向不了解代码的开发者解释问题。
5. **测试你的添加** —— 提交之前，请尝试使用更新后的技能扫描包含你所记录漏洞的代码，确保它能被正确检测且输出有帮助。
6. **保持时效性** —— 添加 CVE 时，请对照原始通告来源（NVD、GitHub Advisories、各生态系统特定数据库）核实信息。

### 漏洞数据来源

- [国家漏洞数据库 NVD](https://nvd.nist.gov/)
- [GitHub 安全通告数据库](https://github.com/advisories)
- [Snyk 漏洞数据库](https://security.snyk.io/)
- [PyPI 安全通告数据库](https://github.com/pypa/advisory-database)
- [npm Audit](https://docs.npmjs.com/auditing-package-dependencies-for-security-vulnerabilities)
- [RustSec 安全通告数据库](https://rustsec.org/advisories/)
- [OSV（开源漏洞数据库）](https://osv.dev/)
- 各语言的安全邮件列表和博客

## 快速开始

### 安装

下载 `vibe-guard.skill` 并安装到 Claude Code 或 Claude.ai 中。

### 使用方法

**手动扫描**（默认模式）：
```
检查这段代码的安全问题
```
```
这段代码安全吗？[粘贴代码]
```
```
对我的项目运行 vibe-guard 检查
```

**启用自动扫描模式**（每次代码生成后自动检查）：
```
Enable vibe-guard auto mode
```

**关闭自动扫描模式**：
```
Disable vibe-guard auto mode
```

### 输出示例

当 Vibe Guard 检测到问题时，你会获得两种输出：

**1. 行内标注** —— 直接在代码中标注问题：
```python
# 🔴 严重：硬编码管理员后门 —— 请完全移除
if username == "superadmin" and password == "master123":
    return grant_full_access()

# 🟠 高危：SQL 注入 —— 请使用参数化查询
query = f"SELECT * FROM users WHERE id = {user_input}"
```

**2. 结构化安全报告** —— 包含发现的问题、严重程度、位置和具体修复建议。

## 工作原理

Vibe Guard 遵循 5 步检测流水线：

1. **识别环境** —— 检测编程语言、版本和依赖清单文件
2. **加载参考文件** —— 拉取相关的语言和依赖参考文件
3. **通用模式扫描** —— 对照全部 12 个威胁类别进行检查
4. **交叉比对** —— 将发现结果与版本特定的已知漏洞进行匹配
5. **分级与报告** —— 分配严重程度等级并生成双格式输出

## 项目结构

```
vibe-guard/
├── SKILL.md                          # 核心检测逻辑与流水线
├── README.md                         # 英文文档
├── README_CN.md                      # 中文文档（本文件）
└── references/                       # 社区维护的漏洞数据库
    ├── languages/                    # 语言特定模式（欢迎添加！）
    │   ├── python/
    │   │   ├── common.md
    │   │   ├── python2.md
    │   │   ├── python3.0-3.7.md
    │   │   └── python3.8+.md
    │   ├── javascript/common.md
    │   ├── java/common.md
    │   ├── cpp/common.md
    │   ├── go/common.md
    │   ├── rust/common.md
    │   └── php/common.md
    └── dependencies/                 # 已知漏洞版本（欢迎添加！）
        ├── pypi/common.yaml
        ├── npm/common.yaml
        └── maven/common.yaml
```

## 路线图

- [ ] 添加更多语言（C#、Kotlin、Swift、Ruby、Shell、Solidity）
- [ ] 添加更多依赖生态系统（Cargo、NuGet、Gems、Go Modules）
- [ ] JavaScript 细粒度版本文件（ES5 vs ES2015+ vs 各 Node.js 版本）
- [ ] Java 版本特定文件（Java 8 vs 11 vs 17 vs 21）
- [ ] 框架特定深度参考（Next.js、Django、Spring Boot、Rails）
- [ ] 已知漏洞代码样本的集成测试套件
- [ ] 自动化 CVE 数据源接入，保持依赖通告实时更新
- [ ] 基于真实漏洞利用数据的严重程度评分校准
