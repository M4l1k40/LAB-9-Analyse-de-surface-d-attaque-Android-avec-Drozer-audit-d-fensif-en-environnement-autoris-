# Analyse des Risques — Broadcast Receivers Exportés
**Application :** `com.android.insecurebankv2`  
**Catégorie OWASP Mobile :** M1 – Improper Platform Usage

---

## Composant Analysé

### `MyBroadCastReceiver` ⚠️ ÉLEVÉ

| Attribut | Valeur |
|---|---|
| Nom complet | `com.android.insecurebankv2.MyBroadCastReceiver` |
| Exporté | ✅ Oui (`android:exported="true"`) |
| Permission requise | `null` ❌ |
| Action écoutée | `theBroadcast` |
| Extras traités | `phonenumber` (String), `newpass` (String) |

---

## Comportement du Receiver

À la réception de l'action `theBroadcast`, le receiver exécute la séquence suivante :

```
1. Lit l'extra "phonenumber" → numéro cible fourni par l'émetteur
2. Lit l'extra "newpass"     → nouveau mot de passe fourni par l'émetteur
3. Envoie un SMS vers "phonenumber" contenant "newpass"
4. Met potentiellement à jour le mot de passe en base locale
```

**Le receiver n'effectue AUCUNE vérification :**
- Pas de vérification de l'identité de l'expéditeur
- Pas de validation du numéro de téléphone
- Pas de confirmation utilisateur
- Pas de token d'autorisation

---

## Scénario d'Attaque Complet

```
[ÉTAPE 1] L'attaquant installe une app malveillante sur l'appareil de la victime
          (ou exploite une app déjà installée avec permission SEND_BROADCAST)
          ↓
[ÉTAPE 2] L'app envoie un broadcast avec l'action "theBroadcast"
          Extras : phonenumber=<numéro_attaquant>, newpass=<mdp_choisi>
          ↓
[ÉTAPE 3] MyBroadCastReceiver traite l'intent sans vérification
          ↓
[ÉTAPE 4] Un SMS est envoyé vers le numéro de l'attaquant avec le nouveau MDP
          + Le MDP du compte bancaire est potentiellement modifié
          ↓
[RÉSULTAT] Prise de contrôle du compte bancaire + exfiltration des credentials
```

---

## Extrait du Manifeste Vulnérable

```xml
<!-- VULNÉRABLE : aucune permission, action publique -->
<receiver android:name=".MyBroadCastReceiver"
          android:exported="true">
    <intent-filter>
        <action android:name="theBroadcast"/>
    </intent-filter>
</receiver>
```

---

## Remédiation Recommandée

```xml
<!-- OPTION 1 : Restreindre avec une permission custom -->
<receiver android:name=".MyBroadCastReceiver"
          android:exported="true"
          android:permission="com.android.insecurebankv2.INTERNAL_BROADCAST">
    <intent-filter>
        <action android:name="theBroadcast"/>
    </intent-filter>
</receiver>

<!-- OPTION 2 (préférable) : Ne pas exporter si usage interne uniquement -->
<receiver android:name=".MyBroadCastReceiver"
          android:exported="false">
    ...
</receiver>

<!-- OPTION 3 : Utiliser LocalBroadcastManager pour les broadcasts internes -->
<!-- → Remplacer sendBroadcast() par LocalBroadcastManager.sendBroadcast() -->
```

---

## Évaluation du Risque

| Critère | Évaluation |
|---|---|
| Vecteur d'attaque | Local (app malveillante sur l'appareil) |
| Complexité | Faible |
| Privilèges requis | Aucun |
| Interaction utilisateur | Aucune |
| Impact confidentialité | Élevé (exfiltration MDP) |
| Impact intégrité | Élevé (modification MDP) |
| **Score CVSS estimé** | **8.4 (High)** |

---

## Références
- OWASP Mobile Top 10 : M1 – Improper Platform Usage
- CWE-925 : Improper Verification of Intent by Broadcast Receiver
- Android Security : https://developer.android.com/guide/components/broadcasts#restricting_broadcasts_with_permissions
