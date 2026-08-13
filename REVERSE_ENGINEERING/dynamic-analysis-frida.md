# 🔌 Dynamic Analysis with Frida

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **Tópico** | Instrumentação dinâmica de processos |
| **Dificuldade** | ⭐⭐⭐⭐⭐ Muito Alto |
| **Ferramenta** | Frida, Frida-tools |
| **Status** | ⚠️ Ativo e essencial |
| **Aplicações** | Malware behavior, Vulnerability exploitation |

---

## 🔍 O que é?

Análise dinâmica de processos em execução usando Frida para instrumentar código, interceptar chamadas e injetar lógica customizada sem reiniciar o processo.

---

## 🛠️ Setup Frida

### **Instalação**

```bash
# Python tools
pip install frida-tools frida

# Android (via adb)
adb push frida-server-16.1.3-android-arm64 /data/local/tmp/frida-server
adb shell chmod +x /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server

# iOS (via SSH)
scp frida-server-16.1.3-ios-arm64e mobile@target:/usr/local/bin/frida-server
ssh mobile@target "chmod +x /usr/local/bin/frida-server && frida-server"
```

### **Verificação**

```bash
# Listar processos disponíveis
frida-ps -U            # Android via USB
frida-ps -H localhost  # Host local
frida-ps -R            # iOS remoto

# Conectar e abrir shell
frida-repl -U com.app.package
```

---

## 📝 Exemplos de Frida

### **1. Interceptar Função**

```python
# ❌ Aplicação original (Kotlin/Android)
class LoginActivity {
    fun validateLogin(username: String, password: String): Boolean {
        return username == "admin" && password == "secret123"
    }
}

# ✅ Script Frida para interceptar
import frida
import sys

def on_message(message, data):
    if message['type'] == 'send':
        print(f"[*] {message['payload']}")

device = frida.get_usb_device()
pid = device.spawn(["com.vulnerable.app"])
session = device.attach(pid)

script_source = """
Java.perform(function() {
    // Hook da função validateLogin
    const LoginActivity = Java.use("com.vulnerable.app.LoginActivity");
    
    LoginActivity.validateLogin.implementation = function(username, password) {
        console.log("[*] validateLogin chamado");
        console.log("[*] Username: " + username);
        console.log("[*] Password: " + password);  // Senha capturada!
        
        // Sempre retornar true (bypass do login)
        return true;
    };
});
"""

script = session.create_script(script_source)
script.on('message', on_message)
script.load()
device.resume(pid)

sys.stdin.read()
```

### **2. Capturar Chamadas de API Criptográfica**

```python
# Script Frida para interceptar criptografia
script_source = """
Java.perform(function() {
    // Hook AES encryption
    const Cipher = Java.use("javax.crypto.Cipher");
    
    Cipher.doFinal.overload('java.nio.ByteBuffer', 'java.nio.ByteBuffer').implementation = function(input, output) {
        console.log("[*] Cipher.doFinal chamado");
        
        // Ler plaintext antes da criptografia
        const plaintext = Java.use("java.nio.ByteBuffer")
            .allocate(input.remaining());
        input.get(plaintext.array());
        
        console.log("[*] Plaintext: " + plaintext.array().toString());
        
        // Chamar função original
        return this.doFinal(input, output);
    };
    
    // Hook getMessage() de exceções
    const Exception = Java.use("java.lang.Exception");
    Exception.getMessage.implementation = function() {
        console.log("[*] Exception message: " + this.getMessage());
        console.log("[*] Stack trace: " + Java.use("android.util.Log").getStackTraceString(this));
        return this.getMessage();
    };
});
"""
```

### **3. Patch de Bytecode (Bypass de Verificação)**

```python
# Bypass de verificação de app rooteado
script_source = """
Java.perform(function() {
    const Runtime = Java.use("java.lang.Runtime");
    
    // Hook de exec() para detectar comandos de root check
    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        console.log("[*] Runtime.exec: " + cmd);
        
        // Detectar verificações de root
        if (cmd.includes("su") || cmd.includes("magisk")) {
            console.log("[!] Bloqueando verificação de root");
            throw new Error("Comando bloqueado");
        }
        
        return this.exec(cmd);
    };
});
"""

# Bypass de verificação de certificado SSL
script_source = """
Java.perform(function() {
    const TrustManager = Java.use("javax.net.ssl.TrustManager");
    const X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    
    // Criar TrustManager que aceita qualquer certificado
    const FakeTrustManager = Java.registerClass({
        name: "FakeTrustManager",
        implements: [X509TrustManager],
        methods: {
            checkClientTrusted: function() {},
            checkServerTrusted: function() {},
            getAcceptedIssuers: function() { return null; }
        }
    });
    
    const SSLContext = Java.use("javax.net.ssl.SSLContext");
    SSLContext.init.overload('javax.net.ssl.KeyManager[]', 'javax.net.ssl.TrustManager[]', 'java.security.SecureRandom').implementation = function(km, tm, sr) {
        console.log("[*] Instalando FakeTrustManager");
        
        const fake = FakeTrustManager.$new();
        return this.init(km, [fake], sr);
    };
});
"""
```

### **4. Injetar Código JavaScript em WebView**

```python
# Interceptar comunicação JavaScript-Java
script_source = """
Java.perform(function() {
    const WebView = Java.use("android.webkit.WebView");
    
    WebView.loadUrl.overload('java.lang.String').implementation = function(url) {
        console.log("[*] WebView.loadUrl: " + url);
        
        // Injetar JavaScript malicioso
        const maliciousJS = "alert('Página comprometida');";
        this.loadUrl("javascript:" + maliciousJS);
        
        return this.loadUrl(url);
    };
    
    // Hook de addJavascriptInterface
    WebView.addJavascriptInterface.implementation = function(obj, interfaceName) {
        console.log("[*] JavaScript interface adicionada: " + interfaceName);
        console.log("[*] Objeto: " + obj);
        
        return this.addJavascriptInterface(obj, interfaceName);
    };
});
"""
```

### **5. Extrair Dados de Memória**

```python
# Script para extrair chaves e dados sensíveis
script_source = """
Java.perform(function() {
    // Hook de método que usa chave criptográfica
    const SecretKeySpec = Java.use("javax.crypto.spec.SecretKeySpec");
    
    SecretKeySpec.$init.overload('[B', 'java.lang.String').implementation = function(key, algo) {
        console.log("[*] SecretKeySpec criado");
        console.log("[*] Algoritmo: " + algo);
        
        // Imprimir chave em hex
        const hexKey = Java.use("java.util.Arrays").toString(key);
        console.log("[*] Chave: " + hexKey);
        
        // Enviar para script Python
        send({
            type: "key",
            algorithm: algo,
            key_bytes: key,
            key_hex: Java.use("javax.xml.bind.DatatypeConverter").printHexBinary(key)
        });
        
        return this.$init(key, algo);
    };
});
"""
```

---

## 🎯 Casos de Uso Comuns

### **1. Capturar Credenciais**

```python
# Interceptar login
script_source = """
Java.perform(function() {
    const LoginManager = Java.use("com.app.auth.LoginManager");
    
    LoginManager.authenticate.implementation = function(username, password) {
        console.log("[+] Credenciais capturadas!");
        console.log("[+] Usuário: " + username);
        console.log("[+] Senha: " + password);
        
        // Salvar em arquivo
        const FileWriter = Java.use("java.io.FileWriter");
        const fw = FileWriter.$new("/sdcard/Download/credentials.txt");
        fw.write("Usuario: " + username + "\\nSenha: " + password);
        fw.close();
        
        return this.authenticate(username, password);
    };
});
"""
```

### **2. Monitorar Requisições de Rede**

```python
# Hook de HttpURLConnection
script_source = """
Java.perform(function() {
    const HttpURLConnection = Java.use("java.net.HttpURLConnection");
    
    HttpURLConnection.getInputStream.implementation = function() {
        console.log("[*] Requisição HTTP");
        console.log("[*] URL: " + this.getURL());
        console.log("[*] Método: " + this.getRequestMethod());
        
        // Ler headers
        const headers = this.getHeaderFields();
        const keySet = headers.keySet().iterator();
        while (keySet.hasNext()) {
            const key = keySet.next();
            console.log("[*] Header: " + key + " = " + headers.get(key));
        }
        
        return this.getInputStream();
    };
});
"""
```

### **3. Contornar Proteções**

```python
# Bypass de verificação de debugging
script_source = """
Java.perform(function() {
    const Debug = Java.use("android.os.Debug");
    
    Debug.isDebuggerConnected.implementation = function() {
        console.log("[*] isDebuggerConnected chamado");
        return false;  // Sempre retornar que não está debuggado
    };
});
"""

# Bypass de verificação de emulador
script_source = """
Java.perform(function() {
    const Build = Java.use("android.os.Build");
    
    Build.getFingerprint.implementation = function() {
        return "google/blueline/blueline:12/SQ1A.210205.011/6869899:user/release-keys";
    };
    
    Build.getModel.implementation = function() {
        return "Pixel 3";
    };
});
"""
```

---

## 🛡️ Detecção de Frida

```python
# Detectar injeção de Frida
script_source = """
// ❌ VULNERABLE - Sem detecção
Java.perform(function() {
    // ... hooks ...
});

// ✅ SEGURO - Detectar Frida
try {
    Java.use("frida.Frida");
    console.log("[!] Frida detectado!");
    exit(1);
} catch (e) {
    // Frida não está presente
}
"""
```

---

## 📚 Referências

- [Frida Official Documentation](https://frida.re/)
- [Frida API Reference](https://frida.re/docs/home/)
- [OWASP Mobile Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Frida Codeshare](https://codeshare.frida.re/)

---

**Status:** ✅ Completado  
**Última atualização:** 2026-08-13  
**Dificuldade:** ⭐⭐⭐⭐⭐ Muito Alto
