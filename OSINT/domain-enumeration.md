# 🔎 Domain & Subdomain Enumeration

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **OSINT Fase** | 1 - Reconnaissance |
| **Dificuldade** | ⭐⭐⭐ Intermediário |
| **Legalidade** | ✅ 100% Legal |
| **Status** | 🟢 Essencial |
| **Impacto** | Mapeia surface de ataque |

---

## 🔍 O que é?

Processo de descobrir informações sobre um domínio (registrante, DNS, servidores) e enumerar subdomínios para mapeamento de surface de ataque.

---

## 🛠️ Técnicas de Reconhecimento de Domínio

### **1. WHOIS - Informações de Registração**

```bash
# Informações do domínio
whois google.com

# Resultado (resumido):
# Registrar: RegistrarCorp
# Registrant: Google Inc.
# Created: 1997-09-15
# Updated: 2023-06-01
# Expires: 2028-09-13
# Nameservers: ns1.google.com, ns2.google.com

# Informações de IP
whois -h whois.arin.net 142.250.185.46

# Resultado:
# CIDR: 142.250.0.0/15
# Organization: Google LLC
# Country: US
# Phone: +1-650-253-0000
```

### **2. DNS Enumeration - dig & nslookup**

```bash
# Básico
dig google.com
dig google.com +short

# Consultas específicas
dig google.com MX      # Mail servers
dig google.com NS      # Nameservers
dig google.com SOA     # Start of Authority
dig google.com TXT     # Text records (SPF, etc)
dig google.com CNAME   # Alias
dig google.com A       # IPv4
dig google.com AAAA    # IPv6

# Transferência de zona (se permitida)
dig @ns1.google.com google.com AXFR

# Reverse DNS
dig -x 142.250.185.46

# nslookup
nslookup google.com
nslookup -type=MX google.com
nslookup -type=NS google.com
```

### **3. Ferramentas Especializadas**

```bash
# Online WHOIS (busca avançada)
# https://www.whois.com/
# https://www.arin.net/
# https://mxtoolbox.com/

# DNS Lookup Online
# https://mxtoolbox.com/
# https://www.nslookup.io/

# Port 53 para transferência de zona
nmap --script dns-zone-transfer -p 53 ns1.target.com
```

---

## 🛠️ Enumeração de Subdomínios

### **1. Enumeração Passiva (sem contato direto)**

```bash
# Certificate Transparency Logs
# https://crt.sh/
# Exemplo: curl -s "https://crt.sh/?q=google.com&output=json" | jq

# Online databases
# https://www.shodan.io/     - IP search
# https://censys.io/         - Certificates & IPs
# https://urlscan.io/        - URL submissions
# https://securitytrails.com/ - DNS history
```

### **2. TheHarvester (Email + Subdomain)**

```bash
# Instalar
pip install theharvester

# Executar
theHarvester -d google.com -l 100 -b google
# -d: domínio
# -l: limite de resultados
# -b: fonte (google, bing, linkedin, etc)

# Resultado:
# google-analytics.google.com
# mail.google.com
# docs.google.com
# maps.google.com
```

### **3. Subfinder (Rápido e Eficiente)**

```bash
# Instalar
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# Executar
subfinder -d google.com

# Com múltiplas fontes
subfinder -d google.com -all

# Salvar em arquivo
subfinder -d google.com -o subdomains.txt
```

### **4. Amass (Mais Completo)**

```bash
# Instalar
apt install amass

# Enumeração básica
amass enum -d google.com

# Modo passivo (sem escanear)
amass enum -passive -d google.com

# Com config file
amass enum -config config.ini -d google.com

# Resultado:
# mail.google.com
# docs.google.com
# drive.google.com
# photos.google.com
# ...
```

### **5. Assetfinder (Leve)**

```bash
# Instalar
go install github.com/tomnomnom/assetfinder@latest

# Executar
assetfinder google.com
assetfinder --subs-only google.com  # Apenas subdomínios

# Combinado com outras ferramentas
assetfinder google.com | tee targets.txt
```

---

## 📝 Fluxo Completo de Enumeração

```bash
#!/bin/bash

TARGET="example.com"

echo "[1] WHOIS Lookup"
whois $TARGET

echo "[2] DNS Records"
dig $TARGET +short
dig $TARGET MX +short
dig $TARGET NS +short
dig $TARGET TXT +short

echo "[3] Subdomínios - Subfinder"
subfinder -d $TARGET -silent

echo "[4] Subdomínios - Amass"
amass enum -passive -d $TARGET

echo "[5] Subdomínios - Assetfinder"
assetfinder --subs-only $TARGET

echo "[6] Reverse DNS"
dig -x $(dig $TARGET +short | head -1)

echo "[7] Certificados (crt.sh)"
curl -s "https://crt.sh/?q=%25.$TARGET&output=json" | jq -r '.[].name_value' | sed 's/\*.//g' | sort -u

echo "[8] Juntar e Limpar"
cat subdomains_subfinder.txt subdomains_amass.txt subdomains_assetfinder.txt | sort -u | tee all_subdomains.txt

echo "[9] Validar Subdomínios Ativos"
while read domain; do
    if dig +short $domain | grep -q .; then
        echo "✓ $domain"
    fi
done < all_subdomains.txt | tee active_subdomains.txt

echo "[10] Scan Ports Abertos"
nmap -sV $(cat active_subdomains.txt | tr '\n' ' ')
```

---

## 🔍 Informações Adicionais

### **SPF Records (Email Security)**

```bash
dig google.com TXT +short

# Resultado contém:
# "v=spf1 include:_spf.google.com ~all"

# Verificar servidor de mail autorizado
dig _spf.google.com TXT +short
```

### **DNSSEC**

```bash
# Verificar se DNSSEC está habilitado
dig +dnssec google.com

# Se houver flag "ad" = DNSSEC verificado
```

### **Reverse DNS Enumeration**

```bash
# Descobrir nomes de host por IP
for i in {1..254}; do
    dig -x 192.168.1.$i +short
done

# Ou com nmap
nmap -sL 192.168.1.0/24 | grep "Nmap scan report"
```

---

## 🎯 Checklist de Enumeração

```
Informações do Domínio:
✅ WHOIS - Registrante, criação, expiration
✅ DNS MX - Mail servers
✅ DNS NS - Nameservers
✅ DNS TXT - SPF, DKIM, DMARC
✅ A/AAAA - Endereços IP

Subdomínios:
✅ Certificate Transparency (crt.sh)
✅ Subfinder (múltiplas fontes)
✅ Amass (enumeração completa)
✅ Assetfinder (leve)
✅ TheHarvester (emails + subdomínios)

Validação:
✅ Testar resolução DNS
✅ Verificar se está ativo
✅ Descobrir IP associado
✅ Escanear portas abertas

Documentação:
✅ Salvar todos os subdomínios
✅ Listar apenas ativos
✅ Mapear IPs e serviços
✅ Anotar tecnologias identificadas
```

---

## 📚 Referências

- [WHOIS Information](https://www.whois.com/)
- [Certificate Transparency Logs](https://crt.sh/)
- [Subfinder Documentation](https://github.com/projectdiscovery/subfinder)
- [Amass Project](https://github.com/OWASP/Amass)
- [Assetfinder](https://github.com/tomnomnom/assetfinder)
- [TheHarvester](https://github.com/laramies/theHarvester)
- [OWASP Testing Guide - Footprinting](https://owasp.org/www-project-web-security-testing-guide/)

---

**Status:** ✅ Completado  
**Última atualização:** 2026-08-13  
**Dificuldade:** ⭐⭐⭐ Intermediário  
**Legalidade:** ✅ 100% Legal
