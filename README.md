# 🛡️ Vibe Guard

English | [中文](README_CN.md)

**Community-driven Claude skill for detecting security backdoors in AI-generated code.**

Vibe coding — using AI agents to generate code via natural language — is fast, fun, and productive. But it introduces a unique threat surface: the generated code *looks* correct, passes cursory review, and yet may contain subtle backdoors, data exfiltration logic, or insecure patterns that a human would miss during rapid iteration.

Vibe Guard is an open, community-maintained security skill for Claude that catches these threats. The project ships with a starter set of detection rules and vulnerability references, but **its real strength comes from the community**: every language pattern, CVE entry, and AI-specific anti-pattern contributed by the community makes everyone's vibe coding safer.

> **This is a community-first project.** We provide the framework and pipeline — you bring the expertise. Whether you're a security researcher, a penetration tester, or a developer who just found a sketchy pattern in AI-generated code, your contributions are what make Vibe Guard work. See [Contributing](#contributing) to get started.

## Features

- **12 Universal Threat Categories**: Data exfiltration, RCE, credential exposure, supply chain attacks, backdoor auth, obfuscated code, file system abuse, network threats, persistence & stealth, cryptographic weaknesses, injection attacks, and AI-specific risks
- **Language-Aware Detection**: Version-specific vulnerability patterns for Python, JavaScript/TypeScript, Java, C/C++, Go, Rust, and PHP — [add yours](#1-language-references-high-impact)
- **Dependency Scanning**: Known vulnerability advisories for PyPI, npm, and Maven ecosystems with version-range matching — [add yours](#2-dependency-advisories-high-impact)
- **Dual Output**: Inline annotated code + structured security report with severity levels (🔴 Critical / 🟠 High / 🟡 Medium / 🔵 Low)
- **Auto-Scan Mode**: Optional always-on scanning after every code generation, togglable to save tokens
- **Modular References**: Language and dependency advisories are stored as independent reference files — easy to update, extend, and contribute to

## Contributing

Vibe Guard's power comes from its reference files — the more comprehensive they are, the more threats it catches. The current release covers the most common patterns, but the security landscape is vast and constantly evolving. **We need your help.**

### How to Contribute

#### 1. Language References (High Impact)

Add version-specific vulnerability files for existing or new languages.

**Extending existing languages** — For example, Python currently has `common.md`, `python2.md`, `python3.0-3.7.md`, and `python3.8+.md`. You could add:
- `python3.12+.md` covering new security changes in the latest releases

**Adding new languages** — We don't yet cover:
- Swift / Objective-C (iOS security patterns)
- Kotlin (Android-specific risks, coroutine safety)
- C# / .NET (deserialization, Razor injection, Entity Framework)
- Ruby (ERB injection, mass assignment, Marshal deserialization)
- Scala (Akka, Play Framework patterns)
- Shell/Bash (injection, quoting, globbing attacks)
- Solidity (reentrancy, overflow, front-running)

To add a new language, create `references/languages/<language>/common.md` following the format of existing reference files.

#### 2. Dependency Advisories (High Impact)

Add or update vulnerability entries in the YAML advisory files.

**Extending existing ecosystems** — Add newly disclosed CVEs to `pypi/common.yaml`, `npm/common.yaml`, or `maven/common.yaml`.

**Adding category-specific files** — Split large advisory files by domain:
- `pypi/ml-stack.yaml` — ML/AI specific packages (torch, transformers, jax, etc.)
- `pypi/web-frameworks.yaml` — Web framework packages
- `npm/frontend-frameworks.yaml` — React, Vue, Angular, Svelte advisories

**Adding new ecosystems** — We don't yet cover:
- `cargo/` — Rust crates (crates.io advisories)
- `nuget/` — .NET packages
- `gems/` — Ruby gems
- `packagist/` — PHP Composer packages
- `go-modules/` — Go module vulnerabilities
- `cocoapods/` — iOS dependencies
- `pub/` — Dart/Flutter packages

#### 3. Detection Rules (Medium Impact)

Improve the universal detection patterns in `SKILL.md`:
- Add new threat subcategories under existing categories (3.1–3.12)
- Refine false positive guidance
- Add framework-specific patterns (e.g., Next.js middleware, Remix loaders)

#### 4. AI/Vibe-Coding-Specific Patterns (High Impact)

This is where Vibe Guard differs from traditional security scanners. We especially need:
- Real-world examples of AI-generated code containing subtle security flaws
- Patterns that are common in LLM training data but insecure
- Prompt injection techniques hidden in code comments or docstrings
- "Looks right but isn't" anti-patterns that AI agents frequently produce

### Contribution Format

#### For Language References

Create a Markdown file following this structure:

```markdown
# [Language] [Version Range] — Version-Specific Vulnerabilities

## [Severity Icon] [Severity]: [Brief Title]

[Explanation of the vulnerability and why it matters]

\```[language]
// Vulnerable code
...

// Safe alternative
...
\```

## Table Format for Multiple Related Patterns

| Pattern | Risk | Severity |
|---------|------|----------|
| `dangerous_function()` | What can go wrong | 🔴/🟠/🟡/🔵 |
```

#### For Dependency Advisories

Add entries to the relevant YAML file following this structure:

```yaml
- package: package-name
  affected: ">=1.0,<2.3.1"          # Version range in ecosystem-native format
  severity: 🔴 critical              # One of: 🔴 critical, 🟠 high, 🟡 medium, 🔵 low
  cve: CVE-YYYY-NNNNN               # Or "N/A" if no CVE assigned
  description: "Brief description of the vulnerability"
  fix: "Upgrade to package-name>=2.3.1"
```

### Contribution Guidelines

1. **Be specific** — Include affected version ranges, CVE numbers where available, and concrete code examples showing both vulnerable and safe patterns.
2. **Prioritize actionability** — Every entry should help the user fix the problem. "Don't do this" is not enough — show what to do instead.
3. **Minimize false positives** — If a pattern has legitimate uses (e.g., `eval()` in a REPL), note the context in which it's actually dangerous vs. acceptable.
4. **Keep entries self-contained** — Each vulnerability entry should be understandable without reading other files. Include enough context that Claude can explain the issue to a developer who didn't write the code.
5. **Test your additions** — Before submitting, try using the updated skill to scan code containing the vulnerability you documented. Make sure it's detected correctly and the output is helpful.
6. **Stay current** — When adding CVEs, verify the information against the original advisory source (NVD, GitHub Advisories, ecosystem-specific databases).

### Where to Find Vulnerability Data

- [National Vulnerability Database (NVD)](https://nvd.nist.gov/)
- [GitHub Advisory Database](https://github.com/advisories)
- [Snyk Vulnerability Database](https://security.snyk.io/)
- [PyPI Advisory Database](https://github.com/pypa/advisory-database)
- [npm Audit](https://docs.npmjs.com/auditing-package-dependencies-for-security-vulnerabilities)
- [RustSec Advisory Database](https://rustsec.org/advisories/)
- [OSV (Open Source Vulnerabilities)](https://osv.dev/)
- Language-specific security mailing lists and blogs

## Quick Start

### Installation

Download `vibe-guard.skill` and install it in Claude Code or Claude.ai.

### Usage

**Manual scan** (default):
```
Review this code for security issues
```
```
Is this code safe? [paste code]
```
```
Run a vibe-guard check on my project
```

**Enable auto-scan mode** (checks every code generation):
```
Enable vibe-guard auto mode
```

**Disable auto-scan mode**:
```
Disable vibe-guard auto mode
```

### Example Output

When Vibe Guard detects issues, you get two things:

**1. Inline annotations** directly in your code:
```python
# 🔴 CRITICAL: Hardcoded admin backdoor — remove this entirely
if username == "superadmin" and password == "master123":
    return grant_full_access()

# 🟠 HIGH: SQL injection — use parameterized queries instead
query = f"SELECT * FROM users WHERE id = {user_input}"
```

**2. A structured security report** with findings, severity, location, and concrete remediation steps.

## How It Works

Vibe Guard follows a 5-step detection pipeline:

1. **Identify Environment** — Detect language, version, and dependency manifest files
2. **Load References** — Pull in the relevant language and dependency reference files
3. **Universal Pattern Scan** — Check against all 12 threat categories
4. **Cross-Reference** — Match findings against version-specific known vulnerabilities
5. **Classify & Report** — Assign severity levels and generate dual-format output

## Project Structure

```
vibe-guard/
├── SKILL.md                          # Core detection logic & pipeline
├── README.md                         # English docs
├── README_CN.md                      # 中文文档
└── references/                       # Community-maintained vulnerability database
    ├── languages/                    # Language-specific patterns (add yours!)
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
    └── dependencies/                 # Known vulnerable versions (add yours!)
        ├── pypi/common.yaml
        ├── npm/common.yaml
        └── maven/common.yaml
```

## Roadmap

- [ ] Add more languages (C#, Kotlin, Swift, Ruby, Shell, Solidity)
- [ ] Add more dependency ecosystems (Cargo, NuGet, Gems, Go Modules)
- [ ] Fine-grained version-specific files for JavaScript (ES5 vs ES2015+ vs Node.js versions)
- [ ] Java version-specific files (Java 8 vs 11 vs 17 vs 21)
- [ ] Framework-specific deep references (Next.js, Django, Spring Boot, Rails)
- [ ] Integration test suite with known-vulnerable code samples
- [ ] Automated CVE feed ingestion to keep dependency advisories current
- [ ] Severity scoring calibration based on real-world exploit data
