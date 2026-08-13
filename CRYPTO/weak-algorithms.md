# 🔐 Weak Cryptographic Algorithms

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 9.1 (Crítico) 🔴 |
| **OWASP Top 10** | #2 - Cryptographic Failures |
| **CWE** | CWE-327 - Use of a Broken/Risky Cryptographic Algorithm |
| **Status** | 🔴 Crítico e ativo |
| **Impacto** | Confidencialidade comprometida |

---

## 🔍 O que é?

Uso de algoritmos criptográficos fracos, obsoletos ou quebrados para proteger dados sensíveis.

---

## ❌ Vulnerabilidades Comuns

### **1. MD5 (Completamente Quebrado)**

```python
# ❌ VULNERÁVEL
import hashlib
password = "admin123"
hashed = hashlib.md5(password.encode()).hexdigest()
# Resultado: 0192023a7bbd73250516f069df18b500
# Reversível em: https://md5.gromweb.com
```

**Por que é ruim:**
- Colisões encontradas (2004)
- Rainbow tables disponíveis
- Computacionalmente trivial de quebrar
- Adequado apenas para checksums, NÃO para senhas

### **2. SHA-1 (Deprecado)**

```python
# ❌ VULNERÁVEL
import hashlib
hashed = hashlib.sha1(password.encode()).hexdigest()
# Resultado: f7c3bc1d808e04732d93603db643fe92b3e2e0b1
# Colisões práticas conhecidas
```

**Por que é ruim:**
- Colisões encontradas (2017, Google SHAttered)
- Tamanho de digest insuficiente (160 bits)
- Removido de navegadores moderno
- NIST deprecated desde 2011

### **3. DES & 3DES (Tamanho de Chave Pequeno)**

```python
# ❌ VULNERÁVEL
from Crypto.Cipher import DES3
key = b'24_byte_key_must_be_this_long!!'
cipher = DES3.new(key, DES3.MODE_ECB)
ciphertext = cipher.encrypt(b'plaintext16bytes')
# Chave efetiva: apenas 168 bits (3DES) ou 56 bits (DES)
```

**Por que é ruim:**
- DES: apenas 56 bits (bruteforce em horas)
- 3DES: muito lento, já substituído
- Blocos pequenos (64 bits) → colisões

### **4. RC4 (Stream Cipher Quebrado)**

```python
# ❌ VULNERÁVEL - Nunca use para TLS
from Crypto.Cipher import ARC4
key = b'secret_key'
cipher = ARC4.new(key)
ciphertext = cipher.encrypt(b'sensitive data')
# Biases conhecidos na geração de keystream
```

---

## ✅ Soluções Seguras

### **Para Hashing de Senhas: Argon2**

```python
# ✅ SEGURO - Argon2
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError

ph = PasswordHasher()

# Hash da senha
password = "MySecurePassword123!"
hash = ph.hash(password)
# Resultado: $argon2id$v=19$m=65536,t=3,p=4$...

# Verificar
try:
    ph.verify(hash, password)
    print("Senha correta!")
except VerifyMismatchError:
    print("Senha incorreta!")
```

**Por que Argon2 é melhor:**
- Memory-hard (resiste a GPUs/ASICs)
- Configurável (tempo, memória, paralelismo)
- Vencedor da Password Hashing Competition (2015)
- OWASP recomenda desde 2018

### **Para Hashing de Senhas: bcrypt**

```python
# ✅ SEGURO - bcrypt
import bcrypt

password = b"MyPassword123!"

# Gerar hash com salt automático
salt = bcrypt.gensalt(rounds=12)  # 2^12 = 4096 rounds
hashed_password = bcrypt.hashpw(password, salt)
# Resultado: b'$2b$12$...'

# Verificar
is_correct = bcrypt.checkpw(password, hashed_password)
```

**Por que bcrypt é bom:**
- Incorpora salt automaticamente
- Adaptativo (rounds configuráveis)
- Implementação simples e confiável
- Resistente a timing attacks

### **Para Criptografia Simétrica: AES-256-GCM**

```python
# ❌ VULNERÁVEL - ECB mode (padrão visível)
from Crypto.Cipher import AES
key = b'0123456789abcdef0123456789abcdef'  # 256 bits
cipher = AES.new(key, AES.MODE_ECB)
ciphertext = cipher.encrypt(plaintext)
# Problema: padrões no plaintext são visíveis no ciphertext

# ✅ SEGURO - GCM mode (com autenticação)
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

key = get_random_bytes(32)  # 256 bits
iv = get_random_bytes(12)   # 96 bits
cipher = AES.new(key, AES.MODE_GCM, nonce=iv)

plaintext = b"Dados sensíveis"
ciphertext, tag = cipher.encrypt_and_digest(plaintext)

# Descriptografar (verifica autenticidade)
cipher_dec = AES.new(key, AES.MODE_GCM, nonce=iv)
plaintext_dec = cipher_dec.decrypt_and_verify(ciphertext, tag)
```

**Por que AES-GCM é melhor:**
- Modo authenticated encryption (AEAD)
- Detecta adulteração de dados
- Nenhuma vulnerabilidade conhecida
- Recomendado por NIST e OWASP

### **Para Hash Genérico: SHA-256 ou SHA-3**

```python
# ❌ VULNERÁVEL - SHA-1
import hashlib
digest = hashlib.sha1(data).hexdigest()

# ✅ SEGURO - SHA-256
import hashlib
digest = hashlib.sha256(data).hexdigest()

# ✅ AINDA MELHOR - SHA-3
import hashlib
digest = hashlib.sha3_256(data).hexdigest()
```

---

## 🔄 Tabela Comparativa

| Algoritmo | Tipo | Segurança | Uso Recomendado |
|---|---|---|---|
| **MD5** | Hash | ❌ Quebrado | ~~Nunca~~ |
| **SHA-1** | Hash | ❌ Deprecado | ~~Nunca~~ |
| **SHA-256** | Hash | ✅ Seguro | Checksums, integridade |
| **SHA-3** | Hash | ✅ Excelente | Novo padrão |
| **DES** | Simétrico | ❌ 56 bits | ~~Nunca~~ |
| **3DES** | Simétrico | ⚠️ Lento | Legado apenas |
| **AES-128** | Simétrico | ✅ Seguro | Dados com validade curta |
| **AES-256** | Simétrico | ✅ Excelente | Dados sensíveis |
| **Argon2** | Senha | ✅ Excelente | Hashing de senhas |
| **bcrypt** | Senha | ✅ Excelente | Hashing de senhas |
| **scrypt** | Senha | ✅ Muito bom | Hashing de senhas |

---

## 🛡️ Checklist de Segurança

```
Para Senhas:
✅ Use Argon2, bcrypt ou scrypt
✅ Nunca use MD5 ou SHA-1
✅ Salt deve ser único e aleatório
✅ Armazene apenas o hash, nunca a senha

Para Dados em Repouso:
✅ Use AES-256-GCM ou ChaCha20-Poly1305
✅ Nunca use ECB mode
✅ Use IVs/nonces únicos e aleatórios

Para Checksums/Integridade:
✅ Use SHA-256 ou SHA-3
✅ SHA-1 aceitável apenas se compatibilidade
✅ MD5 apenas se não for segurança-crítico

Para TLS/Comunicação:
✅ Use apenas TLS 1.3
✅ Ciphers modernos (AES-GCM, ChaCha20)
✅ Evite RC4, DES, 3DES
```

---

## 📚 Referências

- [NIST Cryptographic Algorithm Selection Guidance](https://csrc.nist.gov/)
- [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [CWE-327: Use of a Broken/Risky Cryptographic Algorithm](https://cwe.mitre.org/data/definitions/327.html)
- [Password Hashing Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Argon2 Official](https://github.com/P-H-C/phc-winner-argon2)
- [Google SHAttered Project](https://shattered.io/)

---

**Status:** ✅ Completado  
**Última atualização:** 2026-08-13  
**Dificuldade:** ⭐⭐⭐ Intermediário
