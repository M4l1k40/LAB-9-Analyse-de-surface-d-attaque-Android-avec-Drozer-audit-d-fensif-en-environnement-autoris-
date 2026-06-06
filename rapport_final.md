# Rapport d'Audit de Sécurité — Application Android

---

## Informations générales

| Champ | Valeur |
|---|---|
| **Application** | InsecureBankv2 |
| **Package** | `com.android.insecurebankv2` |
| **Version** | 1.0 |
| **Target SDK** | 22 (Android 5.1 — Lollipop) |
| **Min SDK** | 15 (Android 4.0.3) |
| **UID Système** | 10080 |
| **APK Path** | `/data/app/com.android.insecurebankv2-[hash]/base.apk` |
| **Date d'audit** | [DATE] |
| **Auditeur** | [NOM] |
| **Environnement** | Android SDK x86 — Android 8.1.0 (API 27) |
| **Outil principal** | Drozer Framework v2.x — port 31415 |

---

## Résumé exécutif

L'application InsecureBankv2 présente de **multiples vulnérabilités critiques** touchant l'ensemble de ses composants Android exposés. L'audit, conduit avec Drozer sur un émulateur Android 8.1 (API 27), a révélé :

- **5 activités** exportées sans aucune permission, permettant un **contournement total de l'authentification**
- **1 broadcast receiver** déclenchable sans restriction depuis n'importe quelle application tierce, permettant une **exfiltration de credentials par SMS**
- **1 content provider** (`TrackUserContentProvider`) accessible en lecture et écriture sans permission, exposant la **table `trackerusers`**
- Un ciblage SDK obsolète (`targetSdkVersion=22`) et des **permissions système dangereuses** excessives (SEND_SMS, READ_CALL_LOG, READ_PHONE_STATE)

L'application est **intentionnellement vulnérable** à des fins pédagogiques (InsecureBankv2). Ces vulnérabilités constituent des violations directes de l'OWASP Mobile Top 10 (M1, M2, M9) et du référentiel MASVS.

**Impact global estimé : CRITIQUE** — En contexte réel, ces failles permettraient la prise de contrôle complète de comptes bancaires sans aucune interaction utilisateur.

---

## Méthodologie

L'audit a suivi la séquence suivante :

1. **Préparation de l'environnement** — Installation du Drozer Agent sur l'émulateur, port forwarding ADB (`adb forward tcp:31415 tcp:31415`), connexion via `drozer console connect`
2. **Reconnaissance** — Inventaire des paquets installés (`app.package.list`), identification du paquet cible
3. **Cartographie des composants** — Analyse des activités, services, receivers et providers avec `app.*.info`
4. **Analyse du manifeste** — Inspection des flags (`allowBackup`, `debuggable`), permissions, flags d'export
5. **Vérification des protections** — Scan des URIs accessibles (`scanner.provider.finduris`, `app.provider.finduri`)
6. **Analyse des risques** — Évaluation de l'impact et faisabilité pour chaque composant exposé

> Toutes les commandes ont été exécutées sur un environnement de test isolé. Aucune donnée réelle n'a été manipulée.

---

## Environnement technique

```
Appareil   : Android SDK built for x86
Version OS : Android 8.1.0 (Oreo)
SDK Level  : API 27
Build      : sdk_gphone_x86-userdebug 8.1.0 OSM1.180201.037 6739391 dev-keys
Fabricant  : Google
```

*Source : captures 06 — `run shell.exec "getprop ro.build.version.release"` et commandes associées*

---

## Découvertes principales

### Vulnérabilité 1 — Activités internes exportées sans protection ⚠️ CRITIQUE

**Composants affectés :** `PostLogin`, `DoTransfer`, `ChangePassword`, `ViewStatement`

**Preuve Drozer :**
```
dz> run app.activity.info -a com.android.insecurebankv2
Package: com.android.insecurebankv2
  com.android.insecurebankv2.LoginActivity    → Permission: null
  com.android.insecurebankv2.PostLogin        → Permission: null
  com.android.insecurebankv2.DoTransfer       → Permission: null
  com.android.insecurebankv2.ViewStatement    → Permission: null
  com.android.insecurebankv2.ChangePassword   → Permission: null
```

*Source : capture 09 — `run app.activity.info -a com.android.insecurebankv2`*

**Impact :** N'importe quelle application installée sur l'appareil peut lancer directement `DoTransfer` ou `ChangePassword` sans passer par `LoginActivity`, contournant ainsi l'authentification de manière triviale.

**Classification :** OWASP M1 — CWE-926

---

### Vulnérabilité 2 — Content Provider sans permissions R/W ⚠️ CRITIQUE

**Composant affecté :** `TrackUserContentProvider`

**Preuve Drozer :**
```
dz> run app.provider.info -a com.android.insecurebankv2
  Authority     : com.android.insecurebankv2.TrackUserContentProvider
  Read Permission  : null   ← CRITIQUE
  Write Permission : null   ← CRITIQUE
  Multiprocess Allowed : False
  Grant Uri Permissions : False
```
```
dz> run scanner.provider.finduris -a com.android.insecurebankv2
  For sure accessible content URIs:
    content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers
```

*Source : captures 10, 12, 13 — `app.provider.info` et `scanner.provider.finduris`*

**Impact :** La table `trackerusers` est lisible et modifiable sans aucune permission. Un attaquant peut lire, insérer, modifier ou supprimer les enregistrements utilisateur via un simple `ContentResolver`.

**Exploitation possible :**
```bash
dz> run app.provider.query \
    content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers \
    --selection ""
```

**Classification :** OWASP M2 — CWE-284

---

### Vulnérabilité 3 — Broadcast Receiver sans protection ⚠️ ÉLEVÉ

**Composant affecté :** `MyBroadCastReceiver`

**Preuve Drozer :**
```
dz> run app.broadcast.info -a com.android.insecurebankv2
  com.android.insecurebankv2.MyBroadCastReceiver
    Permission: null
```

*Source : capture 10 — `run app.broadcast.info -a com.android.insecurebankv2`*

**Impact :** Toute app peut envoyer l'action `theBroadcast` avec les extras `phonenumber` et `newpass`. Le receiver envoie alors un SMS contenant le nouveau mot de passe vers le numéro de l'attaquant.

**Classification :** OWASP M1 — CWE-925

---

### Vulnérabilité 4 — Permissions sensibles excessives ⚠️ ÉLEVÉ

**Preuve Drozer :**
```
dz> run app.package.info -a com.android.insecurebankv2
Uses Permissions:
  - android.permission.INTERNET
  - android.permission.WRITE_EXTERNAL_STORAGE
  - android.permission.SEND_SMS              ← utilisée pour exfiltration
  - android.permission.USE_CREDENTIALS
  - android.permission.GET_ACCOUNTS
  - android.permission.READ_PROFILE
  - android.permission.READ_CONTACTS
  - android.permission.READ_PHONE_STATE
  - android.permission.READ_CALL_LOG         ← accès historique appels
  - android.permission.ACCESS_NETWORK_STATE
  - android.permission.ACCESS_COARSE_LOCATION
  - android.permission.READ_EXTERNAL_STORAGE
```

*Source : capture 08 — `run app.package.info -a com.android.insecurebankv2`*

**Impact :** `SEND_SMS` est exploitée par `MyBroadCastReceiver`. `READ_CALL_LOG` et `READ_PHONE_STATE` donnent accès à des données très sensibles sans justification fonctionnelle visible.

**Classification :** OWASP M9 — CWE-250

---

### Vulnérabilité 5 — SDK cible obsolète et flags dangereux ⚠️ MOYEN

**Preuve Drozer :**
```
dz> run app.package.manifest com.android.insecurebankv2
<manifest versionCode="1" versionName="1.0"
          package="com.android.insecurebankv2"
          platformBuildVersionName="5.1.1-1819727">
  <uses-sdk minSdkVersion="15" targetSdkVersion="22">
```

*Source : captures 11, 12 — `run app.package.manifest com.android.insecurebankv2`*

**Détail :**

| Flag | Valeur | Risque |
|---|---|---|
| `targetSdkVersion` | 22 (Android 5.1, 2015) | Protections modernes non appliquées |
| `minSdkVersion` | 15 (Android 4.0.3, 2012) | Surface d'attaque élargie |
| `android:allowBackup` | `true` (implicite) | Extraction via `adb backup` sans root |
| `android:debuggable` | `true` (build userdebug) | Attachement debugger possible |

**Classification :** OWASP M9 — CWE-489

---

## Tableau récapitulatif des composants

| Composant | Type | Exporté | Permission | Risque |
|---|---|---|---|---|
| `LoginActivity` | Activity | ✅ Oui | null | 🟡 Faible (entry point légitime) |
| `PostLogin` | Activity | ✅ Oui | null | 🔴 Critique |
| `DoTransfer` | Activity | ✅ Oui | null | 🔴 Critique |
| `ViewStatement` | Activity | ✅ Oui | null | 🟠 Élevé |
| `ChangePassword` | Activity | ✅ Oui | null | 🔴 Critique |
| *(services)* | Service | ❌ Aucun | — | ✅ Nul |
| `MyBroadCastReceiver` | Receiver | ✅ Oui | null | 🟠 Élevé |
| `TrackUserContentProvider` | Provider | ✅ Oui | R: null / W: null | 🔴 Critique |

---

## Recommandations prioritaires

### 1. Désactiver l'export des activités internes
```xml
<!-- Dans AndroidManifest.xml -->
<activity android:name=".PostLogin"      android:exported="false"/>
<activity android:name=".DoTransfer"     android:exported="false"/>
<activity android:name=".ChangePassword" android:exported="false"/>
<activity android:name=".ViewStatement"  android:exported="false"/>
```
**ET** vérifier l'état de session dans `onCreate()` de chaque activité sensible.

### 2. Protéger le Content Provider
```xml
<provider android:name=".TrackUserContentProvider"
          android:readPermission="com.android.insecurebankv2.READ_DATA"
          android:writePermission="com.android.insecurebankv2.WRITE_DATA"
          android:exported="false"/>
```

### 3. Sécuriser le Broadcast Receiver
```xml
<!-- Utiliser LocalBroadcastManager (usage interne uniquement) -->
<receiver android:name=".MyBroadCastReceiver"
          android:exported="false"/>
```
Remplacer `sendBroadcast()` par `LocalBroadcastManager.getInstance(this).sendBroadcast()`.

### 4. Mettre à jour le SDK cible
```xml
<uses-sdk android:minSdkVersion="24"
          android:targetSdkVersion="34"/>
```
Appliquer `android:allowBackup="false"` et supprimer `android:debuggable="true"` du build de production.

### 5. Réduire les permissions
Supprimer du manifeste : `READ_CALL_LOG`, `READ_PHONE_STATE`, `READ_CONTACTS`, `ACCESS_COARSE_LOCATION` si non justifiées fonctionnellement.

---

## Annexes

### Annexe A — Tableau de triage complet
Voir fichier `triage.csv`

### Annexe B — Captures d'écran des preuves

| # | Fichier | Contenu |
|---|---|---|
| 01 | `captures/01_drozer_help.png` | Commandes Drozer disponibles |
| 02 | `captures/02_drozer_agent_ON.png` | Agent Drozer actif sur l'émulateur (port 31415) |
| 03 | `captures/03_adb_forward.png` | Port forwarding ADB |
| 04 | `captures/04_drozer_console_connect.png` | Connexion console Drozer |
| 05 | `captures/05_drozer_list_modules.png` | Liste des modules disponibles |
| 06 | `captures/06_device_info.png` | Informations système (Android 8.1 / API 27) |
| 07 | `captures/07_package_list.png` | Liste des paquets installés |
| 08 | `captures/08_package_info_permissions.png` | 12 permissions de InsecureBankv2 |
| 09 | `captures/09_activity_info.png` | 5 activités exportées, toutes `Permission: null` |
| 10 | `captures/10_service_broadcast_provider_info.png` | Services (0), receiver + provider exposés |
| 11 | `captures/11_manifest_part1.png` | Manifeste — en-tête + permissions |
| 12 | `captures/12_manifest_part2.png` | Manifeste — provider + permissions (suite) |
| 13 | `captures/13_scanner_provider_finduris.png` | URI `/trackerusers` accessible confirmée |
| 14 | `captures/14_app_provider_finduri.png` | URIs déclarées dans le code |

### Annexe C — Mapping OWASP MASVS

| ID Vuln | Vulnérabilité | OWASP Mobile Top 10 | MASVS | CWE |
|---|---|---|---|---|
| V1 | Activités exportées sans auth | M1 – Improper Platform Usage | MSTG-PLATFORM-1 | CWE-926 |
| V2 | Content Provider sans permissions | M2 – Insecure Data Storage | MSTG-STORAGE-6 | CWE-284 |
| V3 | Broadcast Receiver non protégé | M1 – Improper Platform Usage | MSTG-PLATFORM-1 | CWE-925 |
| V4 | Permissions excessives | M9 – Reverse Engineering | MSTG-PLATFORM-2 | CWE-250 |
| V5 | SDK obsolète / flags dangereux | M9 – Reverse Engineering | MSTG-CODE-5 | CWE-489 |
