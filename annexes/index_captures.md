# Index des captures d'écran — Preuves Drozer

**Application :** `com.android.insecurebankv2`  
**Outil :** Drozer Framework  
**Émulateur :** Android 8.1.0 (API 27) — Google SDK x86

---

| # | Fichier | Commande Drozer | Contenu clé |
|---|---|---|---|
| 01 | `01_drozer_help.png` | `drozer` | Help — sous-commandes disponibles |
| 02 | `02_drozer_agent_ON.png` | *(GUI)* | Agent Drozer actif, Embedded Server ON, port 31415 |
| 03 | `03_adb_forward.png` | `adb forward tcp:31415 tcp:31415` | Port forwarding confirmé (retour : 31415) |
| 04 | `04_drozer_console_connect.png` | `drozer console connect` | Connexion réussie — SDK x86 8.1.0 sélectionné |
| 05 | `05_drozer_list_modules.png` | `dz> list` | Modules disponibles : app.activity.*, app.provider.*, scanner.* |
| 06 | `06_device_info.png` | `shell.exec "getprop ..."` | Android 8.1.0 / API 27 / Google / SDK built for x86 |
| 07 | `07_package_list.png` | `run app.package.list` | Liste paquets — com.android.insecurebankv2 visible |
| 08 | `08_package_info_permissions.png` | `run app.package.info -a com.android.insecurebankv2` | 12 permissions dont SEND_SMS, READ_CALL_LOG |
| 09 | `09_activity_info.png` | `run app.activity.info -a com.android.insecurebankv2` | **5 activités exportées, toutes Permission: null** |
| 10 | `10_service_broadcast_provider_info.png` | `run app.service.info` + `app.broadcast.info` + `app.provider.info` | Services: 0 \| Receiver: MyBroadCastReceiver null \| Provider: TrackUserContentProvider R/W null |
| 11 | `11_manifest_part1.png` | `run app.package.manifest com.android.insecurebankv2` | targetSDK=22 / minSDK=15 + permissions |
| 12 | `12_manifest_part2.png` | *(suite manifeste)* | Provider + permissions supplémentaires |
| 13 | `13_scanner_provider_finduris.png` | `run scanner.provider.finduris -a com.android.insecurebankv2` | **URI accessible confirmée : /trackerusers** |
| 14 | `14_app_provider_finduri.png` | `run app.provider.finduri com.android.insecurebankv2` | URIs déclarées dans le bytecode de l'app |
