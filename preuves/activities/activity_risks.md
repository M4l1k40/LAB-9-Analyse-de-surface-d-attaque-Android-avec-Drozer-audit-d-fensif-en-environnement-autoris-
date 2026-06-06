# Analyse des Risques — Activités Exportées
**Application :** `com.android.insecurebankv2`  
**Catégorie OWASP Mobile :** M1 – Improper Platform Usage / M3 – Insecure Communication

---

## Composants Analysés

### 1. `LoginActivity`
| Attribut | Valeur |
|---|---|
| Nom complet | `com.android.insecurebankv2.LoginActivity` |
| Exportée | ✅ Oui (intent-filter MAIN/LAUNCHER) |
| Permission | `null` |
| Justification export | Point d'entrée légitime de l'application |

**Risque :** Faible sur ce composant spécifique.  
**Note :** Son existence rend obsolète la protection des autres activités — un attaquant contourne LoginActivity entièrement.

---

### 2. `PostLogin` ⚠️ CRITIQUE
| Attribut | Valeur |
|---|---|
| Nom complet | `com.android.insecurebankv2.PostLogin` |
| Exportée | ✅ Oui (`android:exported="true"`) |
| Permission | `null` ❌ |
| Intent-Filter | Aucun |

**Risque :** Accès complet au dashboard bancaire sans authentification.

**Scénario d'attaque :**
```
Attaquant installe une app malveillante sur l'appareil
↓
Envoie un Intent explicite vers PostLogin
↓
Accède au compte bancaire de la victime sans aucun identifiant
```

**Commande Drozer :**
```bash
dz> run app.activity.start --component com.android.insecurebankv2 \
    com.android.insecurebankv2.PostLogin
```

**Impact :** Contournement total de l'authentification → CVSS estimé : 9.1 (Critical)

---

### 3. `DoTransfer` ⚠️ CRITIQUE
| Attribut | Valeur |
|---|---|
| Nom complet | `com.android.insecurebankv2.DoTransfer` |
| Exportée | ✅ Oui (`android:exported="true"`) |
| Permission | `null` ❌ |
| Intent-Filter | Aucun |

**Risque :** Initiation de virements bancaires frauduleux sans authentification.

**Scénario d'attaque :**
```
App malveillante lance DoTransfer avec extras :
  - beneficiary = compte_attaquant
  - amount      = 99999
↓
Virement exécuté sans validation utilisateur
```

**Commande Drozer :**
```bash
dz> run app.activity.start --component com.android.insecurebankv2 \
    com.android.insecurebankv2.DoTransfer
```

**Impact :** Fraude financière directe → CVSS estimé : 9.8 (Critical)

---

### 4. `ChangePassword` ⚠️ CRITIQUE
| Attribut | Valeur |
|---|---|
| Nom complet | `com.android.insecurebankv2.ChangePassword` |
| Exportée | ✅ Oui (`android:exported="true"`) |
| Permission | `null` ❌ |
| Intent-Filter | Aucun |

**Risque :** Modification du mot de passe d'un compte sans connaître l'ancien.

**Scénario d'attaque :**
```
Attaquant lance ChangePassword → saisit un nouveau MDP
↓
Victime ne peut plus accéder à son compte
↓
Prise de contrôle complète du compte bancaire
```

**Impact :** Account takeover → CVSS estimé : 9.8 (Critical)

---

### 5. `ViewStatement` ⚠️ ÉLEVÉ
| Attribut | Valeur |
|---|---|
| Nom complet | `com.android.insecurebankv2.ViewStatement` |
| Exportée | ✅ Oui (`android:exported="true"`) |
| Permission | `null` ❌ |

**Risque :** Consultation des relevés bancaires (historique transactions) sans auth.  
**Impact :** Violation RGPD — données financières exposées → CVSS estimé : 7.5 (High)

---

### 6. `FilesActivity` ⚠️ ÉLEVÉ
| Attribut | Valeur |
|---|---|
| Nom complet | `com.android.insecurebankv2.FilesActivity` |
| Exportée | ✅ Oui (`android:exported="true"`) |
| Permission | `null` ❌ |

**Risque :** Accès aux fichiers stockés localement par l'application.  
**Impact :** Fuite de fichiers sensibles (logs, exports PDF, tokens) → CVSS estimé : 7.2 (High)

---

## Recommandations de Remédiation

```xml
<!-- CORRECTION : Ajouter android:exported="false" sur toutes les activités internes -->
<activity android:name=".PostLogin"
          android:exported="false"/>   <!-- ✅ Corrigé -->

<activity android:name=".DoTransfer"
          android:exported="false"/>   <!-- ✅ Corrigé -->

<!-- OU protéger par une permission custom -->
<activity android:name=".DoTransfer"
          android:exported="true"
          android:permission="com.android.insecurebankv2.AUTHENTICATED"/>
```

**Alternative recommandée :** Vérifier l'état de session dans `onCreate()` de chaque activité sensible et rediriger vers `LoginActivity` si non authentifié.

---

## Référence
- OWASP Mobile Top 10 : M1 – Improper Platform Usage
- CWE-926 : Improper Export of Android Application Components
- Android Security : https://developer.android.com/guide/topics/manifest/activity-element#exported
