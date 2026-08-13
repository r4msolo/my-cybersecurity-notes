# 🔑 Weak Password Hashing

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 9.0 (Crítico) 🔴 |
| **OWASP Top 10** | #2 - Cryptographic Failures |
| **CWE** | CWE-326, CWE-328 |
| **Status** | 🔴 Crítico e ativo |
| **Impacto** | Compromise de contas de usuário |

---

## 🔍 O que é?

Implementação inadequada de hashing de senhas usando algoritmos fraco, sem salt, com configurações inadequadas ou sem proteção contra ataques otimizados.

---

## ❌ Vulnerabilidades Comuns

### **1. Plain Text Passwords (Pior Cenário)**

```python
# ❌ COMPLETAMENTE VULNERÁVEL
user_db = {
    'username': 'admin',
    'password': 'admin123'  # Armazenado em plaintext!
}

# Numa data breach: acesso imediato a todas as contas
```

**Impacto:**
- Acesso imediato a todas as contas
- Reutilização de senhas em outros sites
- Crimes de identidade

### **2. MD5 ou SHA-1 sem Salt**

```python
# ❌ VULNERÁVEL - Sem salt
import hashlib

def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()

user_db['password_hash'] = hash_password('admin123')
# Resultado: 0192023a7bbd73250516f069df18b500

# Problema: Rainbow tables contêm todas as combinações comuns
# https://md5.gromweb.com → encontra em 0.00001 segundos
```

### **3. SHA-256 sem Salt**

```python
# ❌ VULNERÁVEL - SHA-256 puro sem salt
import hashlib

def hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()

# Mesmos problemas que MD5/SHA-1:
# Tabelas precalculadas contêm bilhões de hashes
# Ataques de dicionário ultra-rápidos
```

**Por que é ruim:**
- Sem salt: mesma senha = mesmo hash
- Precomputed tables com 10+ bilhões de entradas
- Ataques de dicionário em GPU: 500 bilhões de tentativas/segundo

### **4. Salt Fraco ou Reutilizado**

```python
# ❌ VULNERÁVEL - Salt estático
import hashlib

SALT = "salt123"  # Mesmo salt para todas as senhas!

def hash_password(password):
    return hashlib.sha256((password + SALT).encode()).hexdigest()

# Ou salt muito pequeno:
import os
salt = os.urandom(4)  # Apenas 32 bits

# Com salt de 32 bits: 2^32 = 4 bilhões de possibilidades
# Tabelas de 500GB contêm todas as combinações
```

### **5. Múltiplas Camadas Fracas**

```python
# ❌ VULNERÁVEL
import hashlib

def hash_password(password):
    h = password
    for _ in range(10):  # Apenas 10 iterações!
        h = hashlib.sha256(h.encode()).hexdigest()
    return h

# Computacionalmente trivial em GPU moderna
# 10 iterações SHA-256: milissegundo em GPU
```

### **6. Sem Verificação de Força de Senha**

```python
# ❌ VULNERÁVEL - Qualquer coisa é aceita
def register_user(username, password):
    if len(password) < 1:  # Sem requisitos mínimos
        return False
    
    user_db[username] = hash_password(password)
    return True

# Senhas fracas: "1", "password", "123456"
# Vulneráveis a ataques de dicionário mesmo com salt
```

---

## ✅ Soluções Seguras

### **1. Argon2 (Recomendado OWASP)**

```python
# ✅ SEGURO - Argon2id (vencedor Password Hashing Competition 2015)
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError, InvalidHashError

ph = PasswordHasher()

# Registrar usuário
def register_user(username, password):
    # Validar força
    if len(password) < 12:
        return False, "Senha deve ter pelo menos 12 caracteres"
    
    hash = ph.hash(password)
    # Resultado: $argon2id$v=19$m=65536,t=3,p=4$...$...
    # m=65536: 65MB de memória
    # t=3: 3 iterações
    # p=4: 4 threads paralelos
    
    user_db[username] = hash
    return True, "Usuário registrado"

# Fazer login
def login(username, password):
    try:
        hash_stored = user_db[username]
        ph.verify(hash_stored, password)
        return True, "Login bem-sucedido"
    except VerifyMismatchError:
        return False, "Senha incorreta"
    except InvalidHashError:
        return False, "Hash inválido"

# Teste
register_user('alice', 'MyStrongPassword123!@#')
login('alice', 'MyStrongPassword123!@#')  # True
```

**Por que Argon2 é superior:**
- Memory-hard: resiste a GPUs e ASICs
- Time-hard: múltiplas iterações (t=3+)
- Configurável: m (memória), t (tempo), p (paralelismo)
- Padrão OWASP desde 2018
- Adequado até 2050+

### **2. bcrypt (Alternativa Confiável)**

```python
# ✅ SEGURO - bcrypt
import bcrypt

def register_user(username, password):
    if len(password) < 12:
        return False
    
    # Salt gerado automaticamente com 12 rounds (2^12 = 4096)
    salt = bcrypt.gensalt(rounds=12)  # 12+ recomendado
    hashed = bcrypt.hashpw(password.encode(), salt)
    # Resultado: b'$2b$12$...'
    
    user_db[username] = hashed
    return True

def login(username, password):
    hash_stored = user_db[username]
    
    # Verificar com proteção contra timing attacks
    if bcrypt.checkpw(password.encode(), hash_stored):
        return True
    return False

# Teste
register_user('bob', 'AnotherStrongPassword456!@#')
login('bob', 'AnotherStrongPassword456!@#')  # True
```

**Vantagens:**
- Implementação simplificada
- Salt automático
- Rounds configuráveis
- Adaptativo (mais lento a cada ano)

### **3. scrypt (Alternativa Forte)**

```python
# ✅ SEGURO - scrypt
from Crypto.Protocol.KDF import scrypt
from Crypto.Random import get_random_bytes
import base64

def register_user(username, password):
    if len(password) < 12:
        return False
    
    salt = get_random_bytes(32)  # 256 bits
    
    # N=2^14=16384, r=8, p=1 (recomendado)
    # Usa 64MB de memória, ~100ms em CPU moderna
    key = scrypt(
        password,
        salt,
        key_len=32,
        N=2**14,  # CPU/memory cost
        r=8,      # block size
        p=1       # parallelization
    )
    
    hash_stored = base64.b64encode(salt + key)
    user_db[username] = hash_stored
    return True

def login(username, password):
    hash_stored = base64.b64decode(user_db[username])
    salt = hash_stored[:32]
    stored_key = hash_stored[32:]
    
    key = scrypt(password, salt, 32, N=2**14, r=8, p=1)
    
    # Comparação segura (proteção contra timing attacks)
    return hmac.compare_digest(key, stored_key)
```

### **4. Implementação Segura Completa**

```python
# ✅ SEGURO - Implementação robusta
import re
from argon2 import PasswordHasher
from argon2.exceptions import InvalidHashError, VerifyMismatchError

class SecureAuthSystem:
    def __init__(self):
        self.ph = PasswordHasher()
        self.min_password_length = 12
        self.password_regex = re.compile(
            r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{12,}$'
        )
    
    def validate_password_strength(self, password):
        """Validar força de senha"""
        if len(password) < self.min_password_length:
            return False, "Mínimo 12 caracteres"
        
        if not self.password_regex.match(password):
            return False, "Deve conter maiúscula, minúscula, número e símbolo"
        
        # Verificar palavras comuns
        common_passwords = ['password', 'admin', '123456', 'qwerty']
        if any(w in password.lower() for w in common_passwords):
            return False, "Senha muito comum"
        
        return True, "Senha forte"
    
    def register(self, username, password):
        """Registrar novo usuário"""
        valid, msg = self.validate_password_strength(password)
        if not valid:
            return False, msg
        
        hash = self.ph.hash(password)
        return True, hash
    
    def verify(self, hash, password):
        """Verificar login"""
        try:
            self.ph.verify(hash, password)
            return True
        except (VerifyMismatchError, InvalidHashError):
            return False

# Usar
auth = SecureAuthSystem()

# Registro
success, hash = auth.register('user@example.com', 'MySecurePassword123!@#')
print(success)  # True

# Login
verified = auth.verify(hash, 'MySecurePassword123!@#')
print(verified)  # True
```

---

## 🔄 Tabela Comparativa

| Algoritmo | Segurança | Velocidade | Memory-Hard | Configurável | Uso |
|---|---|---|---|---|---|
| **MD5** | ❌ Quebrado | ⚡⚡⚡ | ❌ | ❌ | ~~Nunca~~ |
| **SHA-1** | ❌ Vulnerável | ⚡⚡⚡ | ❌ | ❌ | ~~Nunca~~ |
| **SHA-256** | ⚠️ Fraco | ⚡⚡⚡ | ❌ | ❌ | ~~Nunca para senhas~~ |
| **bcrypt** | ✅ Excelente | ⚡⚡ | ✅ | ✅ rounds | Recomendado |
| **scrypt** | ✅✅ Muito bom | ⚡ | ✅ | ✅ N,r,p | Excelente |
| **Argon2** | ✅✅✅ Melhor | ⚡ | ✅ | ✅ m,t,p | Padrão OWASP |

---

## 🛡️ Checklist de Segurança

```
Requisitos de Senha:
✅ Mínimo 12-16 caracteres
✅ Maiúscula + minúscula + número + símbolo
✅ Não em listas de passwords comuns
✅ Validação em tempo real no frontend

Hashing:
✅ Use Argon2 ou bcrypt
✅ Único salt por senha (gerado aleatoriamente)
✅ Rounds configuráveis e aumentados ao longo dos anos
✅ Timeouts para prevenção de bruteforce

Armazenamento:
✅ Nunca armazene plaintext
✅ Nunca reutilize salt
✅ Hash em banco de dados protegido
✅ Chaves de banco criptografadas

Operações:
✅ Comparação segura (timing-safe)
✅ Limite de tentativas de login
✅ 2FA/MFA para contas críticas
✅ Alertas de mudança de senha
```

---

## 📚 Referências

- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Argon2 Oficial](https://github.com/P-H-C/phc-winner-argon2)
- [bcrypt Documentação](https://pypi.org/project/bcrypt/)
- [CWE-326: Inadequate Encryption Strength](https://cwe.mitre.org/data/definitions/326.html)
- [NIST SP 800-63B: Authentication and Lifecycle Management](https://pages.nist.gov/800-63-3/sp800-63b.html)

---

**Status:** ✅ Completado  
**Última atualização:** 2026-08-13  
**Dificuldade:** ⭐⭐⭐ Intermediário
