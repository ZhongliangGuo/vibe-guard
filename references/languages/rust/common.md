# Rust — Common Security Patterns

Rust's ownership model prevents many memory safety issues at compile time. However, security risks still exist:

## `unsafe` Blocks (Top Rust Risk)

Any `unsafe` block disables Rust's safety guarantees. Flag all `unsafe` usage and scrutinize carefully:

| Pattern | Risk | Severity |
|---------|------|----------|
| `unsafe { *raw_pointer }` | Memory corruption if pointer is invalid | 🟠 High |
| `unsafe { std::mem::transmute() }` | Type confusion, UB | 🟠 High |
| `unsafe impl Send/Sync` | Data races if incorrect | 🟠 High |
| `unsafe` FFI calls | All C safety issues reintroduced | 🟠 High |
| Excessive `unsafe` blocks | General code smell | 🟡 Medium |

## Logic & Runtime Risks

| Pattern | Risk | Severity |
|---------|------|----------|
| `.unwrap()` on user input | Panic → DoS | 🟡 Medium |
| `Command::new("sh").arg("-c").arg(user_input)` | Shell injection | 🔴 Critical |
| `std::process::Command` with unsanitized args | Command injection | 🟠 High |
| Regex without timeout (use `regex` crate, not hand-rolled) | ReDoS | 🟡 Medium |
| `std::fs::read_to_string(user_path)` | Path traversal | 🟠 High |
| Deadlocks from `Mutex` ordering issues | DoS | 🟡 Medium |

## Dependency Risks

| Pattern | Risk | Severity |
|---------|------|----------|
| `build.rs` executing arbitrary commands | Supply chain attack vector | 🟠 High |
| `proc-macro` crates from untrusted sources | Compile-time code execution | 🟠 High |
| Crates with `unsafe` that aren't audited | Hidden memory safety issues | 🟡 Medium |

## Web (Actix, Axum, Rocket)

| Pattern | Risk | Severity |
|---------|------|----------|
| Missing input validation on extractors | Injection | 🟡 Medium |
| `Html(user_content)` without escaping | XSS | 🟠 High |
| `CorsLayer::very_permissive()` | Overly permissive CORS | 🟡 Medium |
| `sqlx::query!` with format strings instead of binds | SQL injection | 🟠 High |

## Cryptography

| Pattern | Risk | Severity |
|---------|------|----------|
| `rand::thread_rng()` is fine for general use but flag if used for key generation without `OsRng` | Potentially predictable | 🟡 Medium |
| Custom crypto implementations | Rolling your own crypto | 🟠 High |
| `ring` or `rustls` with outdated versions | Known vulnerabilities | 🟡 Medium |
