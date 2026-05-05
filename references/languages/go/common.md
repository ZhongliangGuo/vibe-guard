# Go — Common Security Patterns

## Top Go Risks

| Pattern | Risk | Severity |
|---------|------|----------|
| `os/exec.Command("sh", "-c", userInput)` | Shell injection | 🔴 Critical |
| `text/template` with user input (use `html/template` for web) | XSS / template injection | 🟠 High |
| `net/http` default client (no timeout) | DoS via slowloris / resource exhaustion | 🟡 Medium |
| `json.Unmarshal` into `interface{}` | Type confusion, unexpected behavior | 🟡 Medium |
| `unsafe.Pointer` usage | Memory corruption | 🟠 High |
| `reflect` to bypass unexported fields | Access control bypass | 🟠 High |
| Goroutine leak (missing context cancellation) | Resource exhaustion DoS | 🟡 Medium |
| `defer file.Close()` after error without check | Resource leak | 🔵 Low |

## HTTP / Web

| Pattern | Risk | Severity |
|---------|------|----------|
| Missing `http.Server.ReadTimeout` / `WriteTimeout` | Slowloris DoS | 🟡 Medium |
| `http.ListenAndServe` (no TLS) in production | Cleartext traffic | 🟡 Medium |
| `r.URL.Path` without cleaning (use `path.Clean`) | Path traversal | 🟠 High |
| Custom `http.Dir` serving root filesystem | Directory listing / file exposure | 🔴 Critical |
| CORS `Access-Control-Allow-Origin: *` | Overly permissive | 🟡 Medium |
| Missing CSRF protection in forms | CSRF | 🟠 High |

## Cryptography

| Pattern | Risk | Severity |
|---------|------|----------|
| `crypto/md5` or `crypto/sha1` for security | Weak hash | 🟡 Medium |
| `math/rand` for security (use `crypto/rand`) | Predictable PRNG | 🟠 High |
| `tls.Config{InsecureSkipVerify: true}` | SSL verification disabled | 🟠 High |
| Hardcoded keys/secrets in Go source | Secret exposure | 🟠 High |

## SQL

```go
// Vulnerable: string concatenation
db.Query("SELECT * FROM users WHERE id = " + userID)

// Safe: parameterized
db.Query("SELECT * FROM users WHERE id = $1", userID)
```

## Go-Specific Gotchas

- **Error swallowing**: `result, _ := dangerousOperation()` — the blank identifier `_` silently discards errors. In security-critical code, this can mask failures.
- **Slice header sharing**: Slicing a byte slice shares underlying memory — mutations in one affect the other. Can lead to unintended data exposure.
- **`cgo` usage**: Calling C code from Go reintroduces all C memory safety issues. Flag any `import "C"` in security-critical paths.
