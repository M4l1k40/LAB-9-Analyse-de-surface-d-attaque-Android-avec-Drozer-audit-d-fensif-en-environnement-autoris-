# Audit de Sécurité Android — InsecureBankv2

> **Travaux Pratiques — Sécurité Mobile**  
> Outil : Drozer Framework | Cible : InsecureBankv2 (Android 8.1 / API 27)

---

## Structure du dépôt

```
audit-insecurebankv2/
├── rapport_final.md          ← Rapport complet d'audit
├── triage.csv                ← Tableau de triage des vulnérabilités
├── preuves/
│   ├── activities/           ← Activités exposées
│   ├── services/             ← Services (surface nulle)
│   ├── receivers/            ← Broadcast receivers
│   ├── providers/            ← Content providers
│   └── manifest/             ← Analyse du manifeste
└── annexes/
    └── captures/             ← 14 captures d'écran Drozer
```

## Application auditée

| Champ | Valeur |
|---|---|
| Nom | InsecureBankv2 |
| Package | `com.android.insecurebankv2` |
| Version | 1.0 |
| Target SDK | 22 (Android 5.1) |
| Min SDK | 15 |
| UID | 10080 |

## Environnement de test

| Composant | Valeur |
|---|---|
| Émulateur | Android SDK built for x86 |
| Version Android | 8.1.0 (Oreo) |
| API Level | 27 |
| Fabricant | Google |
| Outil d'analyse | Drozer Framework v2.x |
| Port | 31415 |

## Résumé des découvertes

| Sévérité | Nombre |
|---|---|
| 🔴 Critique | 4 |
| 🟠 Élevé | 3 |
| 🟡 Moyen | 2 |
| 🟢 Faible | 1 |
| **Total** | **10** |
