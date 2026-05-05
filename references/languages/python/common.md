# Python — Common Security Patterns (All Versions)

These patterns apply across all Python versions unless noted otherwise.

## Dangerous Built-ins & Functions

| Pattern | Risk | Severity |
|---------|------|----------|
| `eval(user_input)` | Arbitrary code execution | 🔴 Critical |
| `exec(user_input)` | Arbitrary code execution | 🔴 Critical |
| `__import__(user_input)` | Dynamic module loading from untrusted input | 🔴 Critical |
| `compile()` + `exec()` | Obfuscated code execution | 🔴 Critical |
| `os.system(cmd)` | Shell injection if cmd contains user input | 🟠 High |
| `subprocess.call(cmd, shell=True)` | Shell injection | 🟠 High |
| `os.popen(cmd)` | Shell injection | 🟠 High |
| `pickle.loads(data)` | Arbitrary code execution via deserialization | 🔴 Critical |
| `yaml.load(data)` (without `Loader=SafeLoader`) | Code execution via YAML deserialization | 🔴 Critical |
| `marshal.loads()` | Code object deserialization | 🟠 High |
| `shelve.open()` with untrusted data | Uses pickle internally | 🟠 High |

## Web Framework Patterns

### Flask
| Pattern | Risk | Severity |
|---------|------|----------|
| `app.run(debug=True)` in production | Debugger RCE (Werkzeug debugger allows code execution) | 🔴 Critical |
| `render_template_string(user_input)` | Server-side template injection (SSTI) | 🔴 Critical |
| `app.secret_key = "hardcoded"` | Session forgery | 🟠 High |
| `@app.route()` without method restriction | Unintended HTTP method exposure | 🟡 Medium |

### Django
| Pattern | Risk | Severity |
|---------|------|----------|
| `DEBUG = True` in production settings | Information disclosure | 🟠 High |
| `ALLOWED_HOSTS = ['*']` | Host header attacks | 🟠 High |
| `mark_safe(user_input)` | XSS | 🟠 High |
| `extra()` or `raw()` with string formatting | SQL injection | 🔴 Critical |
| `CSRF_COOKIE_SECURE = False` | CSRF token interception | 🟡 Medium |

### FastAPI
| Pattern | Risk | Severity |
|---------|------|----------|
| Missing dependency injection validation | Authorization bypass | 🟠 High |
| `Response(content=user_input, media_type="text/html")` | XSS | 🟠 High |

## File & Path Handling
| Pattern | Risk | Severity |
|---------|------|----------|
| `open(user_input)` without path validation | Path traversal, arbitrary file read | 🟠 High |
| `os.path.join(base, user_input)` without checking result starts with base | Path traversal via `../` | 🟠 High |
| `tempfile.mktemp()` | Race condition (use `mkstemp()` instead) | 🟡 Medium |
| `chmod(path, 0o777)` | Overly permissive file permissions | 🟡 Medium |

## Cryptography
| Pattern | Risk | Severity |
|---------|------|----------|
| `hashlib.md5()` / `hashlib.sha1()` for security | Weak hash algorithms | 🟡 Medium |
| `random.random()` for tokens/keys | Predictable PRNG (use `secrets` module) | 🟠 High |
| `ssl._create_unverified_context()` | Disabled SSL verification | 🟠 High |
| `requests.get(url, verify=False)` | Disabled SSL verification | 🟠 High |
| `jwt.decode(token, options={"verify_signature": False})` | JWT signature bypass | 🔴 Critical |

## Network
| Pattern | Risk | Severity |
|---------|------|----------|
| `socket.socket()` + `connect()` + `subprocess` piping | Reverse shell | 🔴 Critical |
| `http.server.SimpleHTTPServer` in production | Directory listing, no auth | 🟠 High |
| Requests to `169.254.169.254` | Cloud metadata SSRF | 🔴 Critical |
| `urllib.request.urlopen(user_input)` | SSRF | 🟠 High |
