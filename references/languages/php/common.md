# PHP — Common Security Patterns

## Code Execution (Top PHP Risk)

| Pattern | Risk | Severity |
|---------|------|----------|
| `eval($userInput)` | Arbitrary code execution | 🔴 Critical |
| `system($cmd)`, `exec($cmd)`, `shell_exec($cmd)`, `passthru($cmd)` | Shell injection | 🔴 Critical |
| `preg_replace('/pattern/e', $replacement)` | Code execution via `e` modifier (removed in PHP 7) | 🔴 Critical |
| `assert($userInput)` (PHP < 8) | Code execution (assert evaluates strings in PHP < 8) | 🔴 Critical |
| `create_function()` | Dynamic function creation (deprecated in 7.2, removed in 8.0) | 🟠 High |
| `include($userInput)` / `require($userInput)` | Local/Remote file inclusion (LFI/RFI) | 🔴 Critical |
| `unserialize($untrustedData)` | Object injection → RCE via magic methods | 🔴 Critical |

## SQL Injection

```php
// Vulnerable
$query = "SELECT * FROM users WHERE id = " . $_GET['id'];
mysql_query($query);  // Also: mysql_* functions are removed in PHP 7

// Safe: PDO with prepared statements
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);
```

## XSS

| Pattern | Risk | Severity |
|---------|------|----------|
| `echo $_GET['input']` | Reflected XSS | 🟠 High |
| Missing `htmlspecialchars()` on output | XSS | 🟠 High |
| `htmlspecialchars()` without `ENT_QUOTES` flag | Attribute-based XSS | 🟡 Medium |

## File Upload / Path Traversal

| Pattern | Risk | Severity |
|---------|------|----------|
| `move_uploaded_file()` without extension validation | Arbitrary file upload (webshell) | 🔴 Critical |
| `file_get_contents($userUrl)` | SSRF | 🟠 High |
| `readfile($userPath)` | Path traversal / arbitrary file read | 🟠 High |
| `$_FILES['file']['name']` used directly in path | Path traversal | 🟠 High |

## Session & Auth

| Pattern | Risk | Severity |
|---------|------|----------|
| `session.use_strict_mode = 0` | Session fixation | 🟠 High |
| `session.cookie_httponly = 0` | Session cookie theft via XSS | 🟡 Medium |
| `session.cookie_secure = 0` (HTTPS sites) | Cookie sent over HTTP | 🟡 Medium |
| `md5($password)` or `sha1($password)` | Weak password hashing (use `password_hash()`) | 🟠 High |
| `==` for password comparison (use `hash_equals()`) | Timing attack | 🟡 Medium |

## PHP Configuration Risks

| Setting | Risk | Severity |
|---------|------|----------|
| `display_errors = On` in production | Information disclosure | 🟡 Medium |
| `allow_url_include = On` | Remote file inclusion | 🔴 Critical |
| `allow_url_fopen = On` + user-controlled URLs | SSRF | 🟠 High |
| `register_globals = On` (PHP < 5.4) | Variable injection | 🔴 Critical |
| `magic_quotes_gpc = Off` without manual escaping (PHP < 5.4) | SQL injection | 🟠 High |
| `open_basedir` not set | Arbitrary file access | 🟡 Medium |
| `disable_functions` not configured | Unrestricted function access | 🟡 Medium |

## Laravel / Symfony Specific

| Pattern | Risk | Severity |
|---------|------|----------|
| `DB::raw($userInput)` | SQL injection | 🔴 Critical |
| `{!! $userInput !!}` (Blade unescaped) | XSS | 🟠 High |
| `.env` file exposed in webroot | Full credential leak | 🔴 Critical |
| `APP_DEBUG=true` in production | Stack trace disclosure + RCE (Ignition) | 🔴 Critical |
| Missing CSRF middleware | CSRF | 🟠 High |
