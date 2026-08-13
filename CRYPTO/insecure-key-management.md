# 🔑 Insecure Key Management

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 8.8 (Alto) 🟠 |
| **OWASP Top 10** | #2 - Cryptographic Failures |
| **CWE** | CWE-321, CWE-798 |
| **Status** | 🔴 Crítico e ativo |
| **Impacto** | Perda total de confidencialidade |

---

## 🔍 O que é?

Armazenamento, geração ou gerenciamento inadequado de chaves criptográficas, permitindo acesso não autorizado aos dados criptografados.

---

## ❌ Vulnerabilidades Comuns

### **1. Chaves Hardcoded no Código**

```python
# ❌ VULNERÁVEL - Chave em plaintext
SECRET_KEY = "super_secret_key_12345"
DB_PASSWORD = "admin123"
API_KEY = "sk-1234567890abcdef"

from Crypto.Cipher import AES
cipher = AES.new(SECRET_KEY, AES.MODE_CBC)

# Qualquer pessoa com acesso ao código-fonte tem a chave!
```

**Riscos:**
- Exposição em repositories públicos
- Visível em binários compilados
- Acessível via decompilação
- Mesmo risco em strings de recursos

### **2. Chaves em Arquivos de Configuração**

```ini
# ❌ VULNERÁVEL - config.ini
[database]
host=db.example.com
password=db_password_123
api_key=sk-7890abcdef

# ❌ VULNERÁVEL - .env (se commitado)
SECRET_KEY=my_secret_key
DATABASE_PASSWORD=postgres123
STRIPE_KEY=sk_live_abc123
```

**Por que é ruim:**
- Commits acidentais no git
- Expontos em backups
- Acessível a devs que não precisam
- Sem rotação/auditoria

### **3. Chaves com Entropia Baixa**

```python
# ❌ VULNERÁVEL - Gerador fraco
import random
key = ''.join(random.choice('abcdefghijklmnopqrstuvwxyz') for _ in range(32))
# Entropia: ~5 bits/caractere = 160 bits efetivos
# Bruteforce viável

# ❌ VULNERÁVEL - Seed previsível
random.seed(12345)
key = random.randbytes(32)
# Mesmo seed = mesma chave = previsível
```

### **4. Reutilização de Chaves**

```python
# ❌ VULNERÁVEL - Mesma chave para múltiplos propósitos
master_key = b'0123456789abcdef0123456789abcdef'

# Criptografar dados de usuário
cipher_user = AES.new(master_key, AES.MODE_GCM)
user_encrypted = cipher_user.encrypt(user_data)

# Criptografar tokens
cipher_token = AES.new(master_key, AES.MODE_GCM)
token_encrypted = cipher_token.encrypt(token_data)

# Problema: mesma chave = vulnerabilidade se um for quebrado
```

### **5. Chaves em Logs**

```python
# ❌ VULNERÁVEL
import logging
logging.info(f"Iniciando criptografia com chave: {key}")
logging.debug(f"Token de API: {api_key}")

# ❌ VULNERÁVEL
print(f"[DEBUG] Secret key: {secret_key}")

# Logs são frequentemente:
# - Armazenados em texto plano
# - Acessíveis a múltiplos sistemas
# - Enviados para serviços centralizados
# - Expostos em data breaches
```

---

## ✅ Soluções Seguras

### **1. Usar Variáveis de Ambiente**

```python
# ✅ SEGURO - Variáveis de ambiente
import os
from dotenv import load_dotenv

load_dotenv()  # Carregar de .env (NÃO commitado)

SECRET_KEY = os.getenv('SECRET_KEY')
DATABASE_PASSWORD = os.getenv('DATABASE_PASSWORD')
API_KEY = os.getenv('API_KEY')

if not SECRET_KEY:
    raise ValueError("SECRET_KEY não configurada!")

# .env (adicionado ao .gitignore)
# SECRET_KEY=uma_chave_criptografica_forte_e_aleatoria
# DATABASE_PASSWORD=strong_db_password_123
```

### **2. Usar Secrets Manager**

```python
# ✅ SEGURO - AWS Secrets Manager
import boto3
import json

client = boto3.client('secretsmanager', region_name='us-east-1')

try:
    response = client.get_secret_value(SecretId='prod/db/password')
    secret = json.loads(response['SecretString'])
    db_password = secret['password']
except Exception as e:
    print(f"Erro ao buscar secret: {e}")
```

### **3. Usar HSM (Hardware Security Module)**

```python
# ✅ SEGURO - HSM via PKCS#11
import PyKCS11

pkcs11 = PyKCS11.PyKCS11Lib()
pkcs11.load('/usr/lib/softhsm/libsofthsm2.so')  # Ou HSM real

slot = pkcs11.getSlotList()[0]
session = pkcs11.openSession(slot)
session.login('1234')  # PIN

# Chave nunca deixa o HSM
key_handle = session.generateKeyPair(
    PyKCS11.CKM_AES_KEY_GEN,
    {PyKCS11.CKA_TOKEN: True, PyKCS11.CKA_SENSITIVE: True}
)
```

### **3. Gerar Chaves com Entropia Adequada**

```python
# ✅ SEGURO - Gerador criptográfico
from Crypto.Random import get_random_bytes
import secrets

# Para criptografia
key = get_random_bytes(32)  # 256 bits
iv = get_random_bytes(16)   # 128 bits

# Para tokens
token = secrets.token_urlsafe(32)

# Python 3.6+
random_bytes = secrets.token_bytes(32)
```

### **4. Usar Chaves Derivadas**

```python
# ✅ SEGURO - Derivar chaves de master key
from Crypto.Protocol.KDF import PBKDF2
from Crypto.Random import get_random_bytes
from Crypto.Hash import SHA256

# Master key (armazenada seguramente)
master_key = get_random_bytes(32)

# Diferentes chaves para diferentes propósitos
salt_user = get_random_bytes(16)
salt_token = get_random_bytes(16)

key_user = PBKDF2(master_key, salt_user, dkLen=32, count=100000, hmac_hash_module=SHA256)
key_token = PBKDF2(master_key, salt_token, dkLen=32, count=100000, hmac_hash_module=SHA256)

# Chaves diferentes derivadas do mesmo master key
```

### **5. Rotação de Chaves**

```python
# ✅ SEGURO - Suportar múltiplas chaves
class KeyRotation:
    def __init__(self):
        self.keys = {
            1: b'current_key_v1_32_bytes_length!!',
            0: b'previous_key_v0_32_bytes_length!'  # Para descriptografar antigos
        }
        self.current_version = 1
    
    def encrypt(self, data):
        key = self.keys[self.current_version]
        # ... criptografia ...
        return version, ciphertext
    
    def decrypt(self, version, ciphertext):
        # Suporta descrever com qualquer versão antiga
        key = self.keys.get(version)
        if not key:
            raise ValueError(f"Chave versão {version} não encontrada")
        # ... descriptografia ...
        return plaintext
    
    def rotate_key(self, new_key):
        """Adiciona nova chave"""
        self.current_version = max(self.keys.keys()) + 1
        self.keys[self.current_version] = new_key
```

---

## 🛡️ Checklist de Segurança

```
Geração:
✅ Use CSPRNG (Crypto.Random, secrets)
✅ Mínimo 256 bits para chaves simétricas
✅ Nunca reutilize IVs/nonces
✅ Unique salt por derivação

Armazenamento:
✅ Nunca hardcode no código
✅ Use variáveis de ambiente
✅ Considere HSM para chaves mestras
✅ Criptografe chaves em repouso

Gerenciamento:
✅ Implemente rotação de chaves
✅ Revogue chaves comprometidas
✅ Mantenha registro de uso
✅ Limite acesso via permissões

Operações:
✅ Nunca registre chaves em logs
✅ Use HTTPS/TLS para transportar
✅ Implemente timeouts de sessão
✅ Aplique princípio do menor privilégio
```

---

## 📚 Referências

- [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [CWE-321: Use of Hard-Coded Cryptographic Key](https://cwe.mitre.org/data/definitions/321.html)
- [CWE-798: Use of Hard-Coded Credentials](https://cwe.mitre.org/data/definitions/798.html)
- [OWASP Secret Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [Google Cloud Secret Manager](https://cloud.google.com/secret-manager)
- [Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault/)

---

**Status:** ✅ Completado  
**Última atualização:** 2026-08-13  
**Dificuldade:** ⭐⭐⭐⭐ Avançado
