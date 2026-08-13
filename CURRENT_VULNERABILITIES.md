# 🔴 Vulnerabilidades Atuais (2025-2026)

## Trending Security Issues

### 1. **AI/LLM Injection**
- Prompt injection em ChatGPT, Claude, etc.
- Jailbreak techniques
- Training data extraction
- **Status:** 🟡 Emergente

### 2. **API Security**
- GraphQL injection
- API rate limiting bypass
- Exposed API endpoints
- **Status:** 🔴 Crítico

### 3. **Vertical Privilege Escalation**
- IDOR + account enumeration
- Parameter pollution
- **Status:** 🔴 Comum

### 4. **Supply Chain Attacks**
- Dependency vulnerabilities (npm, pip, etc.)
- Malicious packages
- SolarWinds-like attacks
- **Status:** 🔴 Alto risco

### 5. **Cloud Misconfigurations**
```
AWS:
- S3 bucket public access
- EC2 security groups open
- IAM policies overly permissive
- Exposed metadata endpoints (169.254.169.254)

Azure:
- Storage account exposure
- Key Vault misconfiguration
- RBAC bypass

GCP:
- Firestore/Datastore exposure
- Service account key leaks
```

### 6. **Serverless Security**
- Lambda function exploitation
- Cold start vulnerabilities
- Environment variable leaks
- **Status:** 🟡 Crescente

### 7. **WebAssembly (WASM) Vulnerabilities**
- WASM decompilation
- Buffer overflows in WASM
- Key extraction from WASM
- **Status:** 🟡 Emergente

### 8. **Cryptography Flaws**
- Weak encryption (MD5, SHA1)
- Poor random number generation
- Hardcoded keys
- **Status:** 🔴 Crítico

### 9. **Kubernetes Security**
```
- Insecure container registries
- RBAC misconfiguration
- Network policy bypass
- Kubelet API exposure (port 10250)
- etcd exposure
```

### 10. **Modern JavaScript Vulnerabilities**
- npm typosquatting
- Prototype pollution
- Code injection via eval()
- **Status:** 🔴 Comum

---

## Vulnerabilidades Obsoletas (Raramente vistas)

- ❌ **Nullbyte injection** - Fixado em PHP 5.3+
- ❌ **Adobe Flash exploits** - Flash descontinuado
- ❌ **Gopher protocol RCE** - Removido de navegadores
- ❌ **HTTP Response Splitting** - Fixado em servidores modernos

---

## Top CVEs 2025

| CVE | Produto | CVSS | Status |
|---|---|---|---|
| CVE-2024-xxxxx | OpenSSL | 9.8 | Crítico |
| CVE-2024-xxxxx | Apache Struts | 9.0 | Crítico |
| CVE-2024-xxxxx | WordPress Plugin | 8.8 | Alto |
| CVE-2024-xxxxx | Kubernetes | 9.1 | Crítico |

---

## Técnicas de Bypass Modernas

### WAF Bypass
```
# Encoding
- Unicode encoding (%u)
- UTF-7 encoding
- Hex encoding
- Base64 encoding

# Obfuscation
- Case variation
- Comment insertion (/**/)
- Whitespace manipulation
- Polyglot payloads
```

### SSTI (Server-Side Template Injection)
```
Jinja2: {{ 7*7 }} → 49
Mako: ${ 7*7 } → 49
Erb: <%= 7*7 %> → 49

RCE via SSTI:
{{ ''.__class__.__mro__[1].__subclasses__()[396]('id',shell=True) }}
```

### Insecure Deserialization
```python
# Pickle RCE
import pickle
payload = b'cos\nsystem\n(S"id"\ntR.'
pickle.loads(payload)  # Executa "id"

# YAML RCE
yaml.load('!!python/object/apply:os.system ["id"]')
```

### Log Injection
```
1. Log message via input
2. Read logs via LFI
3. Inject code in logs
4. Execute via include
```

---

## Zero-Days & Exploits Não-Publicados

⚠️ **Sempre considere:**
- Comportamento anômalo é sinal de 0-day
- WAF bypass pode ser 0-day
- Reportar responsavelmente

---

## Defesas Modernas

### SIEM (Security Information & Event Management)
- Splunk, ELK Stack, Datadog
- Detecta padrões de ataque

### EDR (Endpoint Detection & Response)
- CrowdStrike, Microsoft Defender
- Bloqueia execução suspeita

### DLP (Data Loss Prevention)
- Previne exfiltração de dados

### Zero Trust Architecture
- Never trust, always verify
- Micro-segmentation
- Device trust

### SBOM (Software Bill of Materials)
- Rastreia todas as dependências
- Identifica componentes vulneráveis

---

## Recursos Atualizados 2026

| Recurso | Link |
|---|---|
| HackTheBox | https://www.hackthebox.com |
| TryHackMe | https://tryhackme.com |
| PortSwigger 2024 | https://portswigger.net/web-security |
| OWASP 2024 | https://owasp.org/www-project-top-ten/ |
| Burp 2024 | https://portswigger.net/burp |

---

**Última atualização:** Agosto 2026

⚠️ **Nota:** Sempre mantenha conhecimento atualizado. Segurança é campo em constante evolução.
