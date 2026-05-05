# Python 2.x — Version-Specific Vulnerabilities

Python 2 reached end-of-life on January 1, 2020. Any project still using Python 2 is inherently at risk due to unpatched vulnerabilities.

## 🔴 Critical: `input()` executes arbitrary code

In Python 2, `input()` calls `eval()` on user input. This is the single most dangerous Python 2 pattern.

```python
# Python 2 — THIS EXECUTES ARBITRARY CODE
name = input("Enter your name: ")
# User types: __import__('os').system('rm -rf /')

# Safe alternative in Python 2:
name = raw_input("Enter your name: ")
```

## 🟠 High: String formatting with `%` in SQL

Python 2 code often uses `%` formatting for SQL queries, which is vulnerable to injection:

```python
# Vulnerable
cursor.execute("SELECT * FROM users WHERE name = '%s'" % name)

# Safe
cursor.execute("SELECT * FROM users WHERE name = %s", (name,))
```

## 🟡 Medium: Integer overflow

Python 2 has separate `int` and `long` types. On 32-bit systems, `int` can overflow:

```python
# Python 2: may overflow on 32-bit
x = 2**31  # Could wrap around
```

## 🟡 Medium: `exec` statement vs function

In Python 2, `exec` is a statement that can modify local scope unexpectedly:

```python
# Python 2: exec as statement can modify locals
exec "x = 1"  # Modifies local scope
```

## 🟠 High: Unicode handling issues

Python 2's inconsistent unicode handling can lead to encoding-based attacks:

```python
# Path traversal via unicode normalization
filename = user_input.encode('utf-8')
# Certain unicode characters normalize to '../'
```

## General: End-of-Life Risk

Flag any `python_requires` or version indicator showing Python 2 usage:

| Indicator | Action |
|-----------|--------|
| `#!/usr/bin/python2` | 🟠 Flag: EOL runtime |
| `print "hello"` (print statement) | 🟡 Likely Python 2 code |
| `raw_input()` usage | Informational: confirms Python 2 |
| `from __future__ import` | Mixed 2/3 code — check carefully |
