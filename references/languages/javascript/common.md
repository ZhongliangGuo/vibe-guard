# JavaScript / TypeScript — Common Security Patterns

## Dangerous Functions & Patterns

| Pattern | Risk | Severity |
|---------|------|----------|
| `eval(userInput)` | Arbitrary code execution | 🔴 Critical |
| `new Function(userInput)` | Dynamic code execution | 🔴 Critical |
| `setTimeout(stringArg, ...)` | Implicit eval | 🟠 High |
| `setInterval(stringArg, ...)` | Implicit eval | 🟠 High |
| `document.write(userInput)` | DOM-based XSS | 🟠 High |
| `element.innerHTML = userInput` | XSS | 🟠 High |
| `element.outerHTML = userInput` | XSS | 🟠 High |
| `location.href = userInput` | Open redirect | 🟠 High |
| `child_process.exec(cmd)` | Shell injection (Node.js) | 🔴 Critical |
| `child_process.execSync(cmd)` | Shell injection (Node.js) | 🔴 Critical |
| `require(userInput)` | Arbitrary module loading | 🔴 Critical |
| `vm.runInNewContext(code)` | Sandbox escape possible | 🟠 High |
| `JSON.parse()` without try-catch | DoS via malformed input | 🔵 Low |

## Prototype Pollution

Prototype pollution is a JavaScript-specific class of vulnerability where an attacker modifies `Object.prototype`:

```javascript
// Vulnerable: recursive merge without prototype check
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === 'object') {
      target[key] = merge(target[key] || {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
}

// Attack payload:
merge({}, JSON.parse('{"__proto__": {"isAdmin": true}}'));
// Now ALL objects have isAdmin === true

// Safe: check for dangerous keys
if (key === '__proto__' || key === 'constructor' || key === 'prototype') continue;
```

## Node.js Specific

| Pattern | Risk | Severity |
|---------|------|----------|
| `process.env` logged or exposed | Secret leakage | 🟠 High |
| `fs.readFile(userInput)` | Path traversal | 🟠 High |
| `path.join(base, userInput)` without validation | Path traversal via `../` | 🟠 High |
| `path.resolve(userInput)` | May resolve outside intended directory | 🟠 High |
| `express.static('/')` | Serving entire filesystem | 🔴 Critical |
| `app.use(cors())` with no options | Wildcard CORS | 🟡 Medium |
| `helmet` not used in Express | Missing security headers | 🟡 Medium |
| `cookie` without `httpOnly`, `secure`, `sameSite` | Session hijacking | 🟡 Medium |
| `npm install --ignore-scripts` then re-enabling | Supply chain bypass | 🟡 Medium |
| Disabling `__proto__` checks in body parsers | Prototype pollution | 🟠 High |

## Frontend / Browser Specific

| Pattern | Risk | Severity |
|---------|------|----------|
| `postMessage('*')` | Cross-origin data leak | 🟠 High |
| Missing `event.origin` check in `onmessage` | Cross-origin message injection | 🟠 High |
| `window.open(userInput)` | Phishing / open redirect | 🟡 Medium |
| `<a href={userInput}>` without sanitization | `javascript:` protocol XSS | 🟠 High |
| `dangerouslySetInnerHTML` (React) | XSS bypass of React's escaping | 🟠 High |
| `v-html` (Vue) | XSS | 🟠 High |
| `[innerHTML]` (Angular) | XSS (though Angular sanitizes by default) | 🟡 Medium |
| Service Worker registering external script | Persistent compromise | 🔴 Critical |
| `localStorage` for sensitive tokens | XSS can steal them (use httpOnly cookies) | 🟡 Medium |

## TypeScript-Specific

TypeScript type assertions (`as`, `!`) can override safety:

```typescript
// Dangerous: type assertion bypasses validation
const input = req.body as AdminRequest;  // No runtime check!
input.role  // Could be anything

// Safe: runtime validation with zod, io-ts, etc.
const input = AdminRequestSchema.parse(req.body);
```

`any` type usage disables all type checking — flag excessive `any` usage in security-critical code paths.

## Regex DoS (ReDoS)

JavaScript's regex engine is vulnerable to catastrophic backtracking:

```javascript
// Vulnerable: exponential backtracking
const emailRegex = /^([a-zA-Z0-9]+)*@/;
"aaaaaaaaaaaaaaaaaaaaaaaaaaaa!".match(emailRegex);  // Hangs

// Safe: use atomic groups or RE2
```
