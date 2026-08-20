# Vulnerabilidade: Web Cache Poisoning

## a) Estrutura Profissional
- **CVSS Score:** 8.1 (High) - CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N
- **OWASP Category:** A05:2021-Security Misconfiguration
- **CWE:** [CWE-444](https://cwe.mitre.org/data/definitions/444.html) (Inconsistent Interpretation of HTTP Requests) / [CWE-16](https://cwe.mitre.org/data/definitions/16.html) (Configuration)
- **Status:** Vulnerabilidade Crítica de Manipulação de Cache
- **Última atualização:** 2026-08-20

![Severity: High](https://img.shields.io/badge/Severity-High-red)
![Status: Active](https://img.shields.io/badge/Status-Active-green)

---

## b) Exemplos Práticos

### Payloads Comuns (Header Manipulation)
1. `X-Forwarded-Host: evil.com`
2. `X-Host: attacker-site.com`
3. `X-Forwarded-Scheme: http` (forçar downgrade de HTTPS)
4. `X-Original-URL: /admin`
5. `X-Rewrite-URL: /admin`
6. `Cache-Control: max-age=0`
7. `X-Forwarded-Proto: http`
8. `X-Proxy-Backend: backend-interno.local`
9. `Origin: http://evil.com`
10. `User-Agent: <script>alert(1)</script>` (Se refletido na resposta em cache)

### Comparação: Vulnerável vs. Seguro
- **Vulnerável:** O servidor de cache inclui headers não padronizados na chave do cache ou reflete headers de entrada na resposta sem validação.
- **Seguro:** A aplicação utiliza uma lista de permissões (*whitelist*) estrita de headers de entrada e garante que headers sensíveis (como `X-Forwarded-Host`) nunca sejam refletidos ou utilizados para gerar links dinâmicos sem higienização.

---

## c) Contramedidas

### Prevenção em Código
- **Desabilitar Headers Desnecessários:** Remover suporte a headers de tunelamento ou proxy que não são utilizados pela aplicação.
- **Cache Key Normalization:** Garantir que apenas headers esperados e validados compõem a chave do cache.

### Configuração Segura
- **Configuração de CDN:** Desabilitar a funcionalidade de cache para requisições que contenham headers de cabeçalho desconhecidos.
- **WAF:** Bloquear requisições contendo headers suspeitos de manipulação de cache.

---

## d) Referências Citadas
1. [PortSwigger Academy - Web Cache Poisoning](https://portswigger.net/web-security/web-cache-poisoning)
2. [OWASP - Web Cache Poisoning](https://owasp.org/www-community/attacks/Web_Cache_Poisoning)
3. [HackTricks - Web Cache Poisoning](https://book.hacktricks.xyz/pentesting-web/web-cache-poisoning)
4. [PayloadsAllTheThings - Cache Poisoning](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Web%20Cache%20Poisoning)
5. [CWE/CVE Database](https://cve.mitre.org/)

---

## e) Recursos Educacionais

### Roadmap & Plataformas
- **Roadmap:** Focar em HTTP Request Smuggling, análise de Cache Keys e manipulação de headers.
- **Plataformas de Prática:**
    - [PortSwigger Labs - Web Cache Poisoning](https://portswigger.net/web-security/web-cache-poisoning/lab-web-cache-poisoning-with-an-unkeyed-header)
    - [TryHackMe](https://tryhackme.com/)
    - [HackTheBox](https://hackthebox.com/)
- **Certificações:** OSWE, KLCP.
- **Comunidades:** Twitter (Sec researchers), r/netsec, Bug Bounty Writeups (HackerOne/Intigriti).
