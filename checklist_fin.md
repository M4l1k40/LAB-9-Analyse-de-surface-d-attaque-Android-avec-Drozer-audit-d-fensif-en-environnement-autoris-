# Checklist de fin d'audit — InsecureBankv2

**Application :** `com.android.insecurebankv2`  
**Auditeur :** [NOM]  
**Date :** [DATE]

---

## Conformité de l'audit

- [x] Toutes les étapes du lab ont été suivies (Étapes 1 à 6)
- [x] Tous les composants Android ont été analysés
  - [x] Activities → 5 exportées, toutes `Permission: null`
  - [x] Services → 0 exporté (`No exported services.`)
  - [x] Broadcast Receivers → 1 exporté (`MyBroadCastReceiver`, `Permission: null`)
  - [x] Content Providers → 1 exporté (`TrackUserContentProvider`, R/W null)
- [x] Le manifeste complet a été extrait et analysé (`app.package.manifest`)
- [x] Les URIs de Content Providers ont été scannées (`scanner.provider.finduris`)
- [x] Le tableau de triage est complet (10 entrées V1–V10)
- [x] Les remédiations proposées sont spécifiques et applicables
- [x] Le mapping OWASP MASVS est correct (M1, M2, M9 — MSTG-PLATFORM-1/2, MSTG-STORAGE-6)

---

## Absence de données sensibles

- [x] Aucune donnée utilisateur réelle n'est présente dans le rapport
- [x] Aucun mot de passe ou clé n'est inclus dans le rapport
- [x] Les captures d'écran ne contiennent pas de données personnelles réelles (environnement de test isolé)
- [x] Les chemins APK complets avec hash ont été anonymisés dans le rapport (`[hash]`)
- [x] Les identifiants personnels ont été supprimés ou remplacés par `[NOM]` / `[DATE]`
- [x] Les commandes d'exploitation dans les fichiers `.txt` utilisent `[NUMERO_REDACTE]` et `[MDP_REDACTE]`

---

## Qualité du rapport

- [x] Le rapport est bien structuré (résumé exécutif → méthodologie → découvertes → recommandations → annexes)
- [x] Les vulnérabilités sont clairement expliquées avec preuve Drozer, impact et classification
- [x] Les recommandations sont précises et actionnables (code XML de correction fourni)
- [x] La documentation est complète (rapport + triage CSV + dossier preuves + captures)
- [x] Le format des livrables est conforme : `rapport_final.md`, `triage.csv`, `checklist_fin.md`

---

## Complétude des preuves

- [x] 14 captures d'écran Drozer fournies et référencées dans le rapport
- [x] Chaque vulnérabilité critique est associée à au moins une capture
- [x] Les dossiers de preuves sont organisés par type de composant

| Capture | Contenu | Référencée dans |
|---|---|---|
| 01_drozer_help.png | Commandes Drozer | README.md |
| 02_drozer_agent_ON.png | Agent actif port 31415 | README.md |
| 03_adb_forward.png | Port forwarding ADB | rapport_final.md |
| 04_drozer_console_connect.png | Connexion console | rapport_final.md — Méthodologie |
| 05_drozer_list_modules.png | Modules disponibles | rapport_final.md — Méthodologie |
| 06_device_info.png | Infos système Android 8.1 | rapport_final.md — Environnement |
| 07_package_list.png | Liste paquets + insecurebankv2 | rapport_final.md — Méthodologie |
| 08_package_info_permissions.png | 12 permissions | V4 + V7 + V9 |
| 09_activity_info.png | 5 activités Permission: null | V1 + V2 + V3 + V5 + V10 |
| 10_service_broadcast_provider_info.png | Services/Receiver/Provider | V3 + V4 + V6 |
| 11_manifest_part1.png | Manifeste — SDK + permissions | V8 |
| 12_manifest_part2.png | Manifeste — provider + permissions suite | V2 + V8 |
| 13_scanner_provider_finduris.png | URI /trackerusers accessible | V4 |
| 14_app_provider_finduri.png | URIs déclarées dans le code | V4 |

---

## Résumé de conformité

| Critère | Statut |
|---|---|
| Audit complet | ✅ |
| Données sensibles absentes | ✅ |
| Rapport structuré | ✅ |
| Triage complet | ✅ (10 vulnérabilités) |
| Captures associées | ✅ (14/14) |
| Mapping OWASP | ✅ |
| **Prêt pour dépôt GitHub** | ✅ |
