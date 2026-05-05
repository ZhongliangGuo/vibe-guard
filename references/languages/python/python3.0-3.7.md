# Python 3.0–3.7 — Version-Specific Vulnerabilities

## 🟠 High: `xml.etree.ElementTree` — XXE (Python < 3.7.1)

Before Python 3.7.1, the XML parser did not disable external entity processing by default:

```python
# Vulnerable in Python < 3.7.1
import xml.etree.ElementTree as ET
tree = ET.parse(untrusted_xml)  # XXE possible

# Mitigation: use defusedxml
from defusedxml.ElementTree import parse
tree = parse(untrusted_xml)
```

## 🟡 Medium: `ssl.PROTOCOL_TLSv1` / `ssl.PROTOCOL_TLSv1_1`

These protocols are deprecated and insecure. Python 3.6+ supports them but they should not be used:

```python
# Vulnerable
context = ssl.SSLContext(ssl.PROTOCOL_TLSv1)

# Safe: use TLS 1.2+
context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
```

## 🟡 Medium: `os.makedirs()` race condition (Python < 3.2)

Before `exist_ok` parameter was added in 3.2, `os.makedirs()` had TOCTOU race conditions:

```python
# Race condition in Python < 3.2
if not os.path.exists(path):
    os.makedirs(path)  # Race between check and creation

# Safe in Python 3.2+
os.makedirs(path, exist_ok=True)
```

## 🟡 Medium: `secrets` module unavailable (Python < 3.6)

The `secrets` module for cryptographically secure random numbers was added in Python 3.6. Code targeting 3.0–3.5 might use `random` for security-sensitive operations:

```python
# Python < 3.6: no secrets module, may use insecure random
import random
token = ''.join(random.choices(string.ascii_letters, k=32))  # INSECURE

# Python 3.6+: use secrets
import secrets
token = secrets.token_urlsafe(32)
```

## 🟠 High: `asyncio` subprocess shell injection (Python 3.4+)

`asyncio.create_subprocess_shell()` is equivalent to `shell=True`:

```python
# Vulnerable
proc = await asyncio.create_subprocess_shell(f"echo {user_input}")

# Safe
proc = await asyncio.create_subprocess_exec("echo", user_input)
```

## 🔵 Low: f-string debugging (Python 3.6+)

f-strings can inadvertently leak sensitive data in logs:

```python
# May leak sensitive data
logger.debug(f"Authenticating user with token={token}")
```
