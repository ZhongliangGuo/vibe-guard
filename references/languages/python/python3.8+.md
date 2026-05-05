# Python 3.8+ — Version-Specific Vulnerabilities

## 🟡 Medium: Walrus operator misuse (Python 3.8+)

The walrus operator (`:=`) can obscure security-critical logic:

```python
# Hard to spot: assignment inside condition can hide bypass logic
if (admin := check_override()) or validate_user(token):
    grant_access()  # admin override silently bypasses validation
```

## 🟠 High: `importlib.resources` path traversal (Python 3.9–3.11)

Some versions had issues with path traversal in package resource access. Always validate resource names.

## 🟡 Medium: `TypedDict` and `dataclass` — missing validation

Python 3.8+ `TypedDict` and `dataclass` provide type hints but NO runtime validation. AI-generated code often assumes they validate input:

```python
# DOES NOT validate at runtime — any dict passes through
class UserInput(TypedDict):
    username: str
    role: str  # Nothing prevents role="admin"

# Need explicit validation
def create_user(data: UserInput):
    if data["role"] not in ALLOWED_ROLES:  # Must check manually
        raise ValueError("Invalid role")
```

## 🟠 High: `tomllib` (Python 3.11+) — no safety issues, but watch for fallbacks

Projects supporting pre-3.11 may use `toml` or `tomli` packages. These are generally safe, but check that they're not using custom TOML parsers with `eval()`.

## 🟡 Medium: `match` statement (Python 3.10+)

Structural pattern matching can obscure control flow in security logic:

```python
# Complex match can hide auth bypass
match request.auth:
    case {"role": "admin"} | {"override": True}:  # Too permissive
        grant_admin()
    case _:
        deny()
```

## 🟠 High: `gh-91133` — `tarfile` path traversal (all Python 3, fixed in 3.12)

The `tarfile` module's `extractall()` was vulnerable to path traversal attacks (CVE-2007-4559 — yes, it went unfixed for 15 years):

```python
# Vulnerable in Python < 3.12
import tarfile
tar = tarfile.open(untrusted_file)
tar.extractall()  # Can write files outside target directory

# Safe in Python 3.12+: data_filter is the default
tar.extractall(filter='data')

# Safe in Python < 3.12: manual path checking
import os
for member in tar.getmembers():
    if os.path.isabs(member.name) or '..' in member.name:
        raise SecurityError("Path traversal detected")
```

## 🔵 Low: `sys.addaudithook()` (Python 3.8+)

Audit hooks can be used defensively, but malicious code might install hooks that suppress security events:

```python
# Suspicious: audit hook that silences events
sys.addaudithook(lambda event, args: None)
```
