# Lab 7 — Analyse dynamique Android avec MobSF (Windows, sans Docker)

> Analyse statique et dynamique de l'APK vulnérable **DIVA** (Damn Insecure and Vulnerable App)  
> à l'aide de **Mobile Security Framework v4.5** sur Windows avec un émulateur AVD rooté.

---

## Prérequis

- Windows 10/11 64-bit
- Python 3.10
- Java JDK 17
- Git
- Android Studio + SDK Platform-Tools (ADB)
- MobSF cloné localement :
  ```powershell
  git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
  cd Mobile-Security-Framework-MobSF
  .\setup.bat
  ```

---

## Étape 1 — Lancement de MobSF (Docker ou natif)

### Avec Docker

```powershell
docker run -it --rm -p 8000:8000 -e MOBSF_ANALYZER_IDENTIFIER=emulator-5554 opensecurity/mobile-security-framework-mobsf:latest
```

MobSF démarre et affiche ses informations de configuration :
<img width="781" height="401" alt="Capture d&#39;écran 2026-05-02 151833" src="https://github.com/user-attachments/assets/9c68ef6f-4bb9-4d85-b2c4-8f97df51b30c" />

> Informations visibles au démarrage :
> - **Version** : MobSF v4.5
> - **REST API Key** : générée automatiquement
> - **OS** : Linux (WSL2 via Docker)
> - **Python** : 3.13.12
> - **CPU** : 8 cores / 16 threads — RAM : 7.63 GB
> - Téléchargement automatique de **JADX v1.5.0** (décompilateur Java)
> - Vérification des mises à jour : aucune mise à jour disponible

### Sans Docker (natif)

```powershell
.\run.bat
```

---

## Étape 2 — Connexion à l'interface web

Ouvre un navigateur et accède à :

```
http://127.0.0.1:8000
```

Connecte-toi avec les identifiants par défaut :

| Champ       | Valeur  |
|-------------|---------|
| Utilisateur | `mobsf` |
| Mot de passe | `mobsf` |


<img width="431" height="249" alt="Capture d&#39;écran 2026-05-02 151857" src="https://github.com/user-attachments/assets/ab6c8405-bf1d-4dad-90f3-633b6afee469" />

---

## Étape 3 — Règle de pare-feu Windows pour ADB

Pour permettre la communication entre MobSF et l'émulateur Android, une règle de pare-feu entrante doit être créée pour le port **5555** (ADB).

Ouvre PowerShell **en tant qu'administrateur** et exécute :

```powershell
New-NetFirewallRule -DisplayName "ADB Emulator MobSF" -Direction Inbound -LocalPort 5555 -Protocol TCP -Action Allow
```

La règle est bien créée et activée :
<img width="987" height="353" alt="Capture d&#39;écran 2026-05-02 152022" src="https://github.com/user-attachments/assets/6fc50ec1-5158-437c-936b-33eabf63cc11" />


> Paramètres confirmés :
> - **Direction** : Inbound
> - **Action** : Allow
> - **Port** : 5555
> - **Protocole** : TCP
> - **Status** : OK — règle active

---

## Étape 4 — Lancement de l'émulateur AVD rooté

MobSF fournit un script PowerShell qui lance l'émulateur avec le system partition en mode écriture (requis pour Frida et le proxy HTTPS).

```powershell
.\scripts\start_avd.ps1
```

Sélectionne ton AVD dans la liste (ici `Pixel_4a_2`, API 30, Google APIs x86_64).

<img width="676" height="330" alt="Capture d&#39;écran 2026-05-02 152637" src="https://github.com/user-attachments/assets/6051eee9-fd3d-4c53-b26c-b63703c9e370" />

> Informations de démarrage :
> - AVD : `Pixel_4a_2` — port **5554**
> - Android emulator version : **36.4.10.0**
> - Graphics backend : **gfxstream**
> - System image : `android-30/google_apis/x86_64`
> - **`System image is writable`** ✓ — requis par MobSF pour l'analyse dynamique
> - Serveur IPv4 détecté : `192.168.100.1`

**Vérification ADB** dans un autre terminal :

```powershell
adb devices
# emulator-5554   device
```

> L'émulateur doit impérativement être lancé **avant** MobSF.

---

## Étape 5 — Analyse statique de l'APK DIVA

Dans MobSF → **Upload & Analyze** → dépose le fichier `app-debug.apk`.

L'analyse statique s'effectue automatiquement et produit le rapport suivant :

<img width="871" height="393" alt="Capture d&#39;écran 2026-05-02 151949" src="https://github.com/user-attachments/assets/f6311b2d-d470-44b1-9365-becaef001d7d" />

### Résultats de l'analyse statique

#### App Scores
| Métrique | Valeur |
|----------|--------|
| Security Score | **43 / 100** |
| Trackers Detected | **0 / 432** |

#### File Information
| Champ | Valeur |
|-------|--------|
| File Name | `app-debug.apk` |
| Size | `5.27 MB` |
| MD5 | `a04c7e62510ff7eeb697ffb4c6f5365e` |
| SHA1 | `cfeb5bc53666e059ec4b64bcd7f3b3153cea9d62cf` |
| SHA256 | `45309d13fbce2c9f9be3eae2fb4a797ba6725d49ad67e073c32ea7b9b2af211` |

#### App Information
| Champ | Valeur |
|-------|--------|
| App Name | **Diva** |
| Package Name | `jakhar.aseem.diva` |
| Main Activity | `jakhar.aseem.diva.MainActivity` |
| Target SDK | **11** |
| Min SDK | **8** |
| Max SDK | **`<non défini>`** |
| Android Version | `3.0` |

#### Composants exposés
| Type | Résultat |
|------|----------|
| Exported Activities | **2 / 17** |
| Exported Services | **0 / 0** |
| Exported Receivers | **1 / 1** ⚠️ |
| Exported Providers | **1 / 2** ⚠️ |

#### Certificat
```
Binary is signed
v1 signature: True
v2 signature: True
```

#### Code décompilé disponible
- `View AndroidManifest.xml`
- `View Source` (Java)
- `View Smali`
- `Download Java Code`
- `Download Smali Code`
- `Download APK`

---


Réalisé par Ibtissam El Bekkali

