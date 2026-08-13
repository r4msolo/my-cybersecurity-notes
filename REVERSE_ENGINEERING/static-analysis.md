# 🔍 Static Binary Analysis

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **Tópico** | Análise estática de binários |
| **Dificuldade** | ⭐⭐⭐⭐⭐ Muito Alto |
| **Ferramentas** | IDA Pro, Ghidra, Radare2, Cutter |
| **Status** | ⚠️ Ativo e essencial |
| **Aplicações** | Malware analysis, Vulnerability research |

---

## 🔍 O que é?

Análise estática de binários compilados ou executáveis sem execução, usando disassemblers e decompilers para entender funcionamento e identificar vulnerabilidades.

---

## 🛠️ Ferramentas

### **IDA Pro (Padrão Ouro)**

```bash
# Download IDA Pro Freeware
wget https://www.hex-rays.com/download/
# Ou
brew install ida-pro  # macOS

# Usar via CLI
idat64 -A -S"./analysis_script.py" malware.exe
```

**Vantagens:**
- Melhor suporte a arquiteturas
- Scripts Python poderosos
- Análise gráfica excelente

### **Ghidra (Gratuito - NSA)**

```bash
# Linux/macOS
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.0_build/ghidra_11.0_PUBLIC.zip
unzip ghidra_11.0_PUBLIC.zip
./ghidra/ghidraRun

# Criar novo projeto
ghidra_analyzeHeadless /path/to/project ProjectName -import /path/to/binary
```

**Vantagens:**
- Gratuito e open source
- Excelente para Windows/Linux binários
- Comunidade ativa

### **Radare2 (CLI Poderoso)**

```bash
# Instalar
sudo apt install radare2

# Análise básica
r2 /path/to/binary

# Comandos principais
aaa                    # Auto-analyze all
s main                 # Seek to main
pd 10                  # Print 10 instructions
af                     # Analyze function
afr@0x401000           # Analyze function at address
```

### **Cutter (GUI para Radare2)**

```bash
# Instalar
sudo apt install cutter

# Ou via GitHub
wget https://github.com/rizinorg/cutter/releases/download/v2.3.1/Cutter-v2.3.1-x86_64.AppImage
chmod +x Cutter-v2.3.1-x86_64.AppImage
./Cutter-v2.3.1-x86_64.AppImage
```

---

## 📝 Fluxo de Análise Estática

### **1. Reconhecimento do Binário**

```bash
# Verificar tipo de arquivo
file malware.exe
# Resultado: PE32 executable (GUI) Intel 80386, for MS Windows

# Verificar dependências
ldd ./program.bin
objdump -p program.bin

# Strings potencialmente interessantes
strings program.bin | grep -i "password\|key\|secret\|admin"

# Símbolos do binário
nm program.bin | grep main
objdump -t program.bin | head -20
```

### **2. Disassembly em Ghidra**

```
1. Abrir Ghidra
2. File → New Project
3. Importar binário
4. Aceitar configurações padrão
5. Duplo-clique para analisar
6. Window → Listing para ver disassembly
7. Procurar por funções principais
```

**Exemplo: Encontrar função main**

```
Ghidra Listing Window:
─────────────────────────────────────────
Address    Instruction
─────────────────────────────────────────
401000     push rbp
401001     mov rbp, rsp
401004     mov edi, 0x402000      ; "Bem-vindo"
401009     call 0x401100          ; puts()
40100c     mov eax, 0x0
40100f     pop rbp
401010     ret
```

### **3. Análise de Funções**

```
Procurar por:
├─ main() - Ponto de entrada
├─ strlen, strcpy, gets - Buffer overflow?
├─ printf - Format string?
├─ system, exec - RCE?
├─ fopen, fwrite - File access?
├─ socket, connect - Network?
└─ malloc, free - Memory management?
```

---

## 💡 Exemplos de Análise

### **Exemplo 1: Descobrir Senha Hardcoded**

```
Disassembly (Ghidra):
─────────────────────────────────────────
401050   mov esi, 0x402050       ; String pointer
401053   lea rdi, [rbp-0x20]     ; Local buffer
401057   call <strcpy>           ; Copiar string
40105a   lea rdi, [rbp-0x20]
40105e   lea rsi, [rbp-0x40]
401062   call <strcmp>           ; Comparar strings
401065   cmp eax, 0x0
401067   jne 0x401080            ; Se diferente, falha

Step 1: Clicar em 0x402050 para ver string
Step 2: Strings window mostra: "admin123password"
```

**Resultado:** Senha hardcoded encontrada: `admin123password`

### **Exemplo 2: Identificar Buffer Overflow**

```
Disassembly:
─────────────────────────────────────────
401100   mov rdi, [rbp+0x10]     ; Parâmetro (user input)
401104   lea rsi, [rbp-0x10]     ; Buffer local (16 bytes)
401108   call <strcpy>           ; ❌ Cópia sem limite!
40110b   ...

Análise:
- strcpy() sem limite de tamanho
- Buffer local: 16 bytes
- Input externo sem validação
- VULNERÁVEL A BUFFER OVERFLOW
```

---

## 🔍 Padrões de Vulnerabilidade

### **1. Buffer Overflow (strcpy)**

```assembly
; ❌ VULNERABLE
mov rdi, [rbp+0x8]    ; user input
lea rsi, [rbp-0x20]   ; buffer 32 bytes
call strcpy           ; Sem limite!

; ✅ SEGURO
mov rdx, 0x20         ; Tamanho máximo
call strncpy          ; Com limite
```

### **2. Format String (printf)**

```assembly
; ❌ VULNERABLE
lea rdi, [rbp+0x8]    ; Arg do usuário
call printf           ; Sem formato especificado!
; Pode ler/escrever memória

; ✅ SEGURO
lea rdi, format_string  ; Formato fixo
lea rsi, [rbp+0x8]
call printf
```

### **3. Integer Overflow**

```assembly
; ❌ VULNERABLE
mov eax, [user_input]   ; 32-bit
add eax, 0x1000
mov [buffer], eax       ; Pode overflow

; ✅ SEGURO
mov rax, [user_input]   ; 64-bit
add rax, 0x1000
cmp rax, MAX_VALUE
jge error
```

---

## 🛡️ Checklist de Análise

```
Reconhecimento:
✅ Identificar tipo de binário (PE, ELF, Mach-O)
✅ Verificar se strips ou com símbolos
✅ Encontrar main() e funções principais
✅ Mapear fluxo de execução

Vulnerabilidades:
✅ Procurar por strcpy, gets, scanf (sem limite)
✅ Verificar sprintf com format string
✅ Identificar divisão por zero
✅ Validar integer arithmetic

Segurança:
✅ Procurar por hardcoded secrets
✅ Verificar validação de input
✅ Buscar por encryption/hashing
✅ Analisar chamadas de sistema

Funcionalidade:
✅ Rastrear estruturas de dados
✅ Mapear lógica de negócio
✅ Identificar estados de máquina
✅ Documentar fluxo crítico
```

---

## 📚 Referências

- [Ghidra Tutorial](https://ghidra-sre.org/)
- [IDA Pro Documentation](https://www.hex-rays.com/products/ida/support/)
- [Radare2 Manual](https://github.com/radareorg/radare2)
- [CWE - Common Weakness Enumeration](https://cwe.mitre.org/)
- [OWASP Binary Analysis](https://owasp.org/)

---

**Status:** ✅ Completado  
**Última atualização:** 2026-08-13  
**Dificuldade:** ⭐⭐⭐⭐⭐ Muito Alto
