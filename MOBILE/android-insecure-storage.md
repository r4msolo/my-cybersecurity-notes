# 📱 Insecure Data Storage (Android)

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 8.9 (Alto) 🟠 |
| **Plataforma** | Android |
| **OWASP Mobile Top 10** | #2 - Insecure Data Storage |
| **CWE** | CWE-312, CWE-549 |
| **Status** | 🔴 Crítico e ativo |
| **Impacto** | Roubo de dados sensíveis |

---

## 🔍 O que é?

Armazenamento inseguro de dados sensíveis (senhas, tokens, dados pessoais) em Android sem criptografia adequada.

---

## ❌ Vulnerabilidades Comuns

### **1. SharedPreferences em Texto Plano**

```kotlin
// ❌ VULNERÁVEL - Acesso a plaintext
val sharedPref = context.getSharedPreferences("user_prefs", Context.MODE_PRIVATE)

// Qualquer um pode ler:
val password = sharedPref.getString("password", "")
val token = sharedPref.getString("api_token", "")
val creditCard = sharedPref.getString("cc_number", "")

// Via adb:
// $ adb shell
// $ cat /data/data/com.example.app/shared_prefs/user_prefs.xml
```

**Impacto:**
- Credenciais expostas
- Tokens de sessão roubados
- Dados financeiros comprometidos

### **2. Salvar Dados em Arquivos Sem Proteção**

```kotlin
// ❌ VULNERÁVEL
val file = File(context.filesDir, "sensitive_data.txt")
file.writeText("password=admin123\ntoken=abc123xyz")

// Legível via:
// $ adb shell
// $ cat /data/data/com.example.app/files/sensitive_data.txt
```

### **3. Intent/Bundle com Dados Sensíveis**

```kotlin
// ❌ VULNERÁVEL - Dados em Intent
val intent = Intent(context, ResultActivity::class.java)
intent.putExtra("password", "secret123")
intent.putExtra("credit_card", "1234-5678-9012-3456")
startActivity(intent)

// Problema: Intents podem ser interceptados ou acessados via dumpsys
// $ adb shell dumpsys activity intents
```

### **4. Logs com Dados Sensíveis**

```kotlin
// ❌ VULNERÁVEL
Log.d("Auth", "Usuário: $username, Senha: $password")
Log.i("Payment", "Cartão: $creditCard, CVV: $cvv")

// Logs acessíveis via:
// $ adb logcat
```

### **5. Dados em WebView Cache**

```kotlin
// ❌ VULNERÁVEL
val webView = findViewById<WebView>(R.id.webview)
webView.settings.domStorageEnabled = true  // Ativa local storage
webView.settings.databaseEnabled = true
// HTML5 local storage salvo em plaintext

// JavaScript pode acessar:
val data = localStorage.getItem("session_token")
```

---

## ✅ Soluções Seguras

### **1. EncryptedSharedPreferences (Recomendado)**

```kotlin
// ✅ SEGURO - Dados criptografados automaticamente
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKey

val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedSharedPreferences = EncryptedSharedPreferences.create(
    context,
    "secret_shared_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

// Salvar dados (criptografados automaticamente)
encryptedSharedPreferences.edit()
    .putString("password", "admin123")
    .putString("api_token", "token_abc123")
    .apply()

// Ler dados (descriptografados automaticamente)
val password = encryptedSharedPreferences.getString("password", "")
val token = encryptedSharedPreferences.getString("api_token", "")
```

**Vantagens:**
- Criptografia transparente (AES-256-GCM)
- Chave mestrada protegida pelo Android Keystore
- API simples (substituir SharedPreferences)
- Suportado desde Android 4.1

### **2. Android Keystore para Chaves**

```kotlin
// ✅ SEGURO - Chaves protegidas pelo hardware
import android.security.keystore.KeyGenParameterSpec
import android.security.keystore.KeyProperties
import java.security.KeyStore
import javax.crypto.KeyGenerator

fun generateKey() {
    val keyStore = KeyStore.getInstance("AndroidKeyStore")
    keyStore.load(null)
    
    val keyGenerator = KeyGenerator.getInstance(
        KeyProperties.KEY_ALGORITHM_AES,
        "AndroidKeyStore"
    )
    
    keyGenerator.init(
        KeyGenParameterSpec.Builder(
            "myKey",
            KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
        )
        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
        .setKeySize(256)
        .build()
    )
    
    keyGenerator.generateKey()
}

// Usar chave para criptografar
val keyStore = KeyStore.getInstance("AndroidKeyStore")
keyStore.load(null)
val key = keyStore.getKey("myKey", null)

val cipher = Cipher.getInstance("AES/GCM/NoPadding")
cipher.init(Cipher.ENCRYPT_MODE, key)

val ciphertext = cipher.doFinal("dados sensíveis".toByteArray())
```

**Benefícios:**
- Chaves nunca saem do Keystore
- Hardware-backed em dispositivos modernos
- Impossível exportar/roubar chaves
- Proteção contra malware

### **3. Encriptar Arquivos**

```kotlin
// ✅ SEGURO - Arquivos criptografados
import androidx.security.crypto.EncryptedFile
import androidx.security.crypto.MasterKey
import java.io.File

val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val file = File(context.filesDir, "sensitive_data.txt")

val encryptedFile = EncryptedFile.Builder(
    context,
    file,
    masterKey,
    EncryptedFile.FileEncryptionScheme.AES256_GCM_HKDF_4KB
).build()

// Escrever dados criptografados
encryptedFile.openFileOutput().use { output ->
    output.write("password=admin123".toByteArray())
}

// Ler dados descriptografados
val plaintext = encryptedFile.openFileInput().bufferedReader().readText()
```

### **4. Segurança de Intent**

```kotlin
// ✅ SEGURO - Dados sensíveis NÃO em Intent
val intent = Intent(context, ResultActivity::class.java)

// Armazenar em local seguro
val prefs = context.getSharedPreferences("temp", Context.MODE_PRIVATE)
prefs.edit().putString("temp_password", "secret123").apply()

// Apenas passar ID temporário
intent.putExtra("data_id", "temp_password")

// Recuperar no outra activity
class ResultActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val dataId = intent.getStringExtra("data_id")
        val prefs = getSharedPreferences("temp", Context.MODE_PRIVATE)
        val password = prefs.getString(dataId, "")
        
        // Limpar dados temp
        prefs.edit().remove(dataId).apply()
    }
}
```

### **5. Desabilitar Logs em Produção**

```kotlin
// ✅ SEGURO
object SecureLog {
    private val isDebugBuild = BuildConfig.DEBUG
    
    fun d(tag: String, msg: String) {
        if (isDebugBuild) {
            Log.d(tag, msg)
        }
    }
    
    fun secureLog(tag: String, sensitiveData: String) {
        if (isDebugBuild) {
            Log.d(tag, sensitiveData)
        } else {
            Log.d(tag, "[REDACTED]")
        }
    }
}

// Uso
SecureLog.secureLog("Payment", "CC: 1234-5678")  // Produção: "[REDACTED]"
```

### **6. WebView Seguro**

```kotlin
// ✅ SEGURO
val webView = findViewById<WebView>(R.id.webview)

webView.settings.apply {
    // Desabilitar local storage inseguro
    domStorageEnabled = false
    databaseEnabled = false
    
    // Desabilitar cache de arquivo
    setAppCacheEnabled(false)
    cacheMode = WebSettings.LOAD_NO_CACHE
    
    // Desabilitar acesso ao filesystem
    allowFileAccess = false
    allowContentAccess = false
    
    // Desabilitar 3rd party cookies
    CookieManager.getInstance().setAcceptThirdPartyCookies(webView, false)
}

// Usar HTTPS apenas
webView.loadUrl("https://example.com")
```

---

## 🛡️ Checklist de Segurança

```
SharedPreferences:
✅ Use EncryptedSharedPreferences
✅ Nunca salve senhas/tokens em plaintext
✅ Limpar dados sensíveis ao logout

Arquivos:
✅ Use EncryptedFile
✅ Nunca salve credenciais em plaintext
✅ Excluir após uso

Intents/Bundles:
✅ Não passe dados sensíveis
✅ Use IDs temporários
✅ Limpar após uso

Logs:
✅ Nunca registre senhas/tokens
✅ Desabilitar logs em produção
✅ Usar [REDACTED] para dados sensíveis

WebView:
✅ Desabilitar local storage
✅ Desabilitar cache
✅ HTTPS apenas
✅ Sem acesso ao filesystem
```

---

## 📚 Referências

- [OWASP Mobile #2: Insecure Data Storage](https://owasp.org/www-project-mobile-top-10/)
- [Android Security: EncryptedSharedPreferences](https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences)
- [Android Keystore System](https://developer.android.com/training/articles/keystore)
- [EncryptedFile Documentation](https://developer.android.com/reference/androidx/security/crypto/EncryptedFile)
- [CWE-312: Cleartext Storage of Sensitive Information](https://cwe.mitre.org/data/definitions/312.html)

---

**Status:** ✅ Completado  
**Última atualização:** 2026-08-13  
**Dificuldade:** ⭐⭐⭐ Intermediário
