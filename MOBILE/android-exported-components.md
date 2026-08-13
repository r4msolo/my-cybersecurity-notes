# 📱 Exported Components (Android)

## 📊 Metadados

| Propriedade | Valor |
|---|---|
| **CVSS Score** | 8.1 (Alto) 🟠 |
| **Plataforma** | Android |
| **OWASP Mobile Top 10** | #1 - Improper Platform Usage |
| **CWE** | CWE-927 - Use of Implicit Intent for Sensitive Communication |
| **Status** | 🔴 Crítico e ativo |
| **Impacto** | Acesso não autorizado a componentes, data leakage |

---

## 🔍 O que é?

Componentes Android (Activities, Services, Broadcast Receivers, Content Providers) exportados sem proteção, permitindo que qualquer app interaja com eles.

---

## ❌ Vulnerabilidades Comuns

### **1. Activity Exportada Sem Proteção**

```xml
<!-- ❌ VULNERÁVEL - AndroidManifest.xml -->
<manifest ...>
    <application>
        <activity
            android:name=".AdminActivity"
            android:exported="true">  <!-- Exportada sem verificação! -->
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

**Ataque:**
```kotlin
// Qualquer app pode abrir AdminActivity
val intent = Intent()
intent.setClassName("com.vulnerable.app", "com.vulnerable.app.AdminActivity")
context.startActivity(intent)  // Abre activity sem autorização
```

### **2. Broadcast Receiver Exportado**

```xml
<!-- ❌ VULNERÁVEL -->
<manifest ...>
    <application>
        <receiver
            android:name=".LocationReceiver"
            android:exported="true">  <!-- Qualquer um pode enviar broadcasts -->
            <intent-filter>
                <action android:name="com.vulnerable.LOCATION_UPDATE" />
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

**Ataque:**
```kotlin
// Qualquer app malicioso pode enviar dados falsos
val intent = Intent("com.vulnerable.LOCATION_UPDATE")
intent.putExtra("latitude", 0.0)
intent.putExtra("longitude", 0.0)
context.sendBroadcast(intent)  // Envia para o receiver exportado
```

### **3. Content Provider Exportado**

```xml
<!-- ❌ VULNERÁVEL -->
<manifest ...>
    <application>
        <provider
            android:name=".UserDataProvider"
            android:authorities="com.vulnerable.userprovider"
            android:exported="true" />  <!-- Acesso público ao banco de dados! -->
    </application>
</manifest>
```

**Ataque:**
```kotlin
// Qualquer app pode fazer queries
val uri = Uri.parse("content://com.vulnerable.userprovider/users")
val cursor = context.contentResolver.query(uri, null, null, null, null)

while (cursor?.moveToNext() == true) {
    val name = cursor.getString(cursor.getColumnIndex("name"))
    val email = cursor.getString(cursor.getColumnIndex("email"))
    val password = cursor.getString(cursor.getColumnIndex("password"))
    
    println("Dados roubados: $name, $email, $password")
}
```

### **4. Service Exportado Sem Autenticação**

```xml
<!-- ❌ VULNERÁVEL -->
<manifest ...>
    <application>
        <service
            android:name=".PaymentService"
            android:exported="true" />  <!-- Qualquer app pode se conectar -->
    </application>
</manifest>
```

**Ataque:**
```kotlin
// Qualquer app pode chamar métodos do service
class MaliciousApp : Service() {
    override fun onCreate() {
        super.onCreate()
        
        val intent = Intent(this, PaymentService::class.java)
        intent.putExtra("action", "transfer_money")
        intent.putExtra("amount", 1000.0)
        intent.putExtra("destination", "attacker_account")
        
        startService(intent)  // Executa operação no PaymentService!
    }
}
```

---

## ✅ Soluções Seguras

### **1. Nunca Exportar Sem Necessidade**

```xml
<!-- ✅ SEGURO - Não exportado (padrão) -->
<manifest ...>
    <application>
        <activity
            android:name=".AdminActivity"
            android:exported="false">  <!-- Apenas nossa app pode abrir -->
        </activity>
    </application>
</manifest>
```

### **2. Exportar com Proteção de Permissão**

```xml
<!-- ✅ SEGURO - Requer permissão -->
<manifest ...>
    <uses-permission android:name="com.vulnerable.ADMIN_PERMISSION" />
    
    <application>
        <activity
            android:name=".AdminActivity"
            android:exported="true"
            android:permission="com.vulnerable.ADMIN_PERMISSION">  <!-- Proteção! -->
        </activity>
    </application>
</manifest>
```

**Declarar a permissão:**
```xml
<permission
    android:name="com.vulnerable.ADMIN_PERMISSION"
    android:protectionLevel="signature" />  <!-- Apenas apps do mesmo developer -->
```

### **3. Broadcast Receiver Seguro**

```xml
<!-- ✅ SEGURO - Com permissão -->
<manifest ...>
    <permission
        android:name="com.vulnerable.LOCATION_PERMISSION"
        android:protectionLevel="signature" />
    
    <application>
        <receiver
            android:name=".LocationReceiver"
            android:exported="true"
            android:permission="com.vulnerable.LOCATION_PERMISSION">
            <intent-filter>
                <action android:name="com.vulnerable.LOCATION_UPDATE" />
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

### **4. Content Provider com Validação**

```xml
<!-- ✅ SEGURO - Sem exportação (usa URI específica) -->
<manifest ...>
    <application>
        <provider
            android:name=".UserDataProvider"
            android:authorities="com.vulnerable.userprovider"
            android:exported="false" />  <!-- Não exportado -->
    </application>
</manifest>
```

**Ou, se precisar exportar:**

```kotlin
// ✅ SEGURO - Validação em código
class UserDataProvider : ContentProvider() {
    private val db: DatabaseHelper? = null
    
    override fun query(
        uri: Uri,
        projection: Array<String>?,
        selection: String?,
        selectionArgs: Array<String>?,
        sortOrder: String?
    ): Cursor? {
        // 1. Verificar caller
        val callerPackage = callingPackage
        if (!isAllowedCaller(callerPackage)) {
            throw SecurityException("Caller não autorizado: $callerPackage")
        }
        
        // 2. Validar URI
        val match = uriMatcher.match(uri)
        if (match < 0) {
            throw IllegalArgumentException("URI desconhecida: $uri")
        }
        
        // 3. Apenas um usuário por query (seu próprio ID)
        val userId = getCurrentUserId()
        val modifiedSelection = "user_id = $userId"
        
        // 4. Never retornar senhas!
        val safeProjection = projection?.filter { it != "password" }?.toTypedArray()
        
        return db?.readableDatabase?.query(
            "users",
            safeProjection,
            modifiedSelection,
            selectionArgs,
            null,
            null,
            sortOrder
        )
    }
    
    private fun isAllowedCaller(packageName: String?): Boolean {
        val allowedPackages = setOf(
            "com.vulnerable.trustedapp1",
            "com.vulnerable.trustedapp2"
        )
        return packageName in allowedPackages
    }
    
    private fun getCurrentUserId(): Int {
        // Retornar apenas dados do usuário atual
        return Binder.getCallingUid()
    }
    
    override fun insert(uri: Uri, values: ContentValues?): Uri? = null
    override fun update(uri: Uri, values: ContentValues?, selection: String?, selectionArgs: Array<String>?): Int = 0
    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int = 0
    override fun getType(uri: Uri): String? = null
    override fun onCreate(): Boolean = true
}
```

### **5. Service Seguro com Autenticação**

```kotlin
// ✅ SEGURO - Service protegido
class PaymentService : Service() {
    override fun onBind(intent: Intent?): IBinder {
        return PaymentBinder()
    }
    
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // 1. Verificar permissão
        if (checkSelfPermission(Manifest.permission.PAYMENT_SERVICE) != PackageManager.PERMISSION_GRANTED) {
            throw SecurityException("Permissão negada")
        }
        
        // 2. Verificar caller
        val callerUid = Binder.getCallingUid()
        if (!isAuthorizedApp(callerUid)) {
            throw SecurityException("App não autorizada")
        }
        
        val action = intent?.getStringExtra("action") ?: return START_NOT_STICKY
        
        // 3. Validar ação
        when (action) {
            "transfer_money" -> {
                val amount = intent.getDoubleExtra("amount", 0.0)
                
                // Validações de segurança
                if (amount <= 0 || amount > 10000.0) {
                    return START_NOT_STICKY  // Rejeitar
                }
                
                performTransfer(amount)
            }
            else -> throw IllegalArgumentException("Ação desconhecida: $action")
        }
        
        return START_STICKY
    }
    
    private fun isAuthorizedApp(uid: Int): Boolean {
        val allowedUids = setOf(1001, 1002)  // UIDs de apps confiáveis
        return uid in allowedUids
    }
    
    private fun performTransfer(amount: Double) {
        // Executar apenas se todas as validações passaram
        println("Transferência de R$$amount autorizada")
    }
    
    inner class PaymentBinder : Binder() {
        fun getService(): PaymentService = this@PaymentService
    }
}
```

---

## 🛡️ Checklist de Segurança

```
AndroidManifest.xml:
✅ Todos componentes com android:exported="false" por padrão
✅ Exportar apenas componentes necessários
✅ Exportação requer android:permission em todas

Validações em Código:
✅ Verificar permissão do caller
✅ Validar package name do app chamador
✅ Verificar UID/PID do processo
✅ Nunca confiar em dados de caller

Content Providers:
✅ Nunca exportar banco de dados completo
✅ Implementar granular access control
✅ Nunca retornar dados sensíveis (senhas, tokens)
✅ Validar todos os parâmetros

Broadcast Receivers:
✅ Usar LocalBroadcastManager para dados internos
✅ Verificar sender de broadcasts públicos
✅ Assinar broadcasts com permissão

Services:
✅ Autenticar clients
✅ Validar chamadas de métodos
✅ Usar permissões apropriadas
✅ Logging de acesso
```

---

## 📚 Referências

- [Android Security: Exported Components](https://developer.android.com/guide/topics/manifest/activity-element#exported)
- [OWASP Mobile #1: Improper Platform Usage](https://owasp.org/www-project-mobile-top-10/)
- [CWE-927: Use of Implicit Intent](https://cwe.mitre.org/data/definitions/927.html)
- [Android Permissions Best Practices](https://developer.android.com/training/permissions)
- [Content Provider Security](https://developer.android.com/guide/topics/providers/content-providers)

---

**Status:** ✅ Completado  
**Última atualização:** 2026-08-13  
**Dificuldade:** ⭐⭐⭐⭐ Avançado
