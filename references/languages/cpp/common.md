# C/C++ — Common Security Patterns

## Memory Safety (Top C/C++ Threat)

| Pattern | Risk | Severity |
|---------|------|----------|
| `strcpy()`, `strcat()`, `sprintf()` | Buffer overflow | 🔴 Critical |
| `gets()` | Unbounded read — always exploitable | 🔴 Critical |
| `malloc()` without size validation | Integer overflow → heap overflow | 🟠 High |
| `free()` then use (use-after-free) | RCE via heap manipulation | 🔴 Critical |
| Double `free()` | Heap corruption → RCE | 🔴 Critical |
| Missing null check after `malloc()` | Null pointer dereference | 🟡 Medium |
| Array access without bounds checking | Out-of-bounds read/write | 🔴 Critical |
| Pointer arithmetic without validation | Arbitrary memory access | 🟠 High |
| `memcpy()` with overlapping regions (use `memmove()`) | Undefined behavior | 🟡 Medium |

Safe alternatives:
- `strcpy` → `strncpy` or `strlcpy`
- `sprintf` → `snprintf`
- `gets` → `fgets`
- Raw pointers → smart pointers (`std::unique_ptr`, `std::shared_ptr`) in C++

## Format String Vulnerabilities

```c
// Vulnerable: user-controlled format string
printf(user_input);        // 🔴 Can read/write arbitrary memory
fprintf(fp, user_input);   // 🔴

// Safe: always specify format
printf("%s", user_input);
```

## Integer Issues

| Pattern | Risk | Severity |
|---------|------|----------|
| Signed/unsigned comparison | Logic bypass | 🟠 High |
| Integer overflow in size calculations | Heap overflow | 🔴 Critical |
| Truncation (e.g., `int` to `short`) | Logic errors | 🟡 Medium |

## Command & Code Execution

| Pattern | Risk | Severity |
|---------|------|----------|
| `system(user_input)` | Shell injection | 🔴 Critical |
| `popen(user_input)` | Shell injection | 🔴 Critical |
| `dlopen(user_path)` | Arbitrary library loading | 🔴 Critical |
| `execve()` with unsanitized args | Command injection | 🟠 High |

## C++ Specific

| Pattern | Risk | Severity |
|---------|------|----------|
| `reinterpret_cast` on untrusted data | Type confusion | 🟠 High |
| Virtual destructor missing in base class | Memory leak / UB | 🟡 Medium |
| `std::string::c_str()` dangling pointer | Use-after-free | 🟠 High |
| Exception in destructor | Undefined behavior / crash | 🟡 Medium |
| `std::thread` without proper synchronization | Race conditions | 🟡 Medium |
