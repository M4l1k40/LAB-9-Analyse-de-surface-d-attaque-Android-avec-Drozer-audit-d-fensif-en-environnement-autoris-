# Analyse des Risques — Services Exportés
**Application :** `com.android.insecurebankv2`

---

## Résultat de l'Analyse

```
dz> run app.service.info -a com.android.insecurebankv2
→ Aucun service exporté détecté.
```

**InsecureBankv2 n'expose aucun Service Android.**

---

## Conclusion

| Critère | Résultat |
|---|---|
| Nombre de services exportés | 0 |
| Surface d'attaque (services) | ✅ Nulle |
| Risque identifié | ✅ Aucun |

L'absence de services exportés est une **bonne pratique** respectée sur ce point.  
L'application concentre sa logique métier dans les Activities et un BroadcastReceiver,
ce qui explique l'absence de services exposés.

---

## Note Pédagogique — Risques théoriques des services exportés

Dans d'autres applications vulnérables, un service exporté sans protection pourrait permettre :

| Risque | Description |
|---|---|
| Exécution de tâches privilégiées | Démarrer un service d'upload, de synchronisation ou de chiffrement |
| DoS (Denial of Service) | Binder un service pour épuiser ses ressources |
| Fuite d'informations | Un service retournant des données via `IBinder` sans auth |

**Référence :** OWASP Mobile M1 – Improper Platform Usage  
**CWE-926 :** Improper Export of Android Application Components
