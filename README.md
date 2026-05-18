# LABORATOIRE FRIDA - RAPPORT D'INSTALLATION ET DE VALIDATION

---

## 📚 Table des matières

1. [Objectifs du laboratoire](#1-objectifs-du-laboratoire)
2. [Prérequis et environnement](#2-prérequis-et-environnement)
3. [Installation du client Frida](#3-installation-du-client-frida)
4. [Déploiement de frida-server](#4-déploiement-de-frida-server-sur-android)
5. [Vérification de la connexion](#5-vérification-de-la-connexion)
6. [Test d'injection - Échec et diagnostic](#6-test-dinjection---échec-et-diagnostic)
7. [Problèmes identifiés et correctifs](#7-problèmes-identifiés-et-correctifs)
8. [Conclusion](#8-conclusion)
9. [Annexes](#9-annexes)

---

## 1. Objectifs du laboratoire

| # | Objectif | Statut |
|---|----------|--------|
| 1 | Installer et configurer Frida (client Python + CLI) sur Windows | ✅ |
| 2 | Déployer `frida-server` sur un émulateur Android | ✅ |
| 3 | Établir une connexion fonctionnelle entre le client et le serveur | ✅ |
| 4 | Injecter un script JavaScript minimal pour valider l'environnement | ⚠️ |
| 5 | Diagnostiquer les problèmes courants d'installation | ✅ |

---

## 2. Prérequis et environnement

| Composant | Version |
|-----------|---------|
| Système d'exploitation | Windows 11 |
| Shell | PowerShell |
| Python | 3.14 |
| Pip | Dernière version |
| ADB (Platform Tools) | Dernière version |
| Émulateur Android | AVD (API 34) |
| Cible | emulator-5554 |

### Configuration Android requise

```bash
# Activer sur l'appareil
- Mode Développeur (7 taps sur "Numéro de build")
- Débogage USB activé

# Vérifier l'architecture
adb shell getprop ro.product.cpu.abi
# Résultat: x86_64
3. Installation du client Frida
3.1 Commandes exécutées
powershell
pip install --upgrade frida frida-tools
3.2 Résultat de l'installation
text
Successfully installed:
- frida-17.9.10
- frida-tools-14.8.2
- colorama-0.4.6
- prompt-toolkit-3.0.52
- pygments-2.20.0
- websockets-13.1
- wcwidth-0.7.0
3.3 Vérification des versions
powershell
> frida --version
17.9.10

> python -c "import frida; print(frida.__version__)"
17.9.10
✅ Statut: Succès - Client Frida correctement installé


<img width="870" height="361" alt="1lab10" src="https://github.com/user-attachments/assets/cccd55dc-8fe2-4e80-9162-83fa4703b8c9" />

4. Déploiement de frida-server sur Android
4.1 Vérification de la connexion ADB
powershell
> adb devices
List of devices attached
emulator-5554    device
✅ Statut: Émulateur détecté et autorisé

4.2 Téléchargement de frida-server
powershell
# URL: https://github.com/frida/frida/releases
# Fichier: frida-server-17.9.10-android-x86_64.xz
# Extraction avec 7-Zip
4.3 Copie du binaire
powershell
> adb push frida-server /data/local/tmp/
frida-server: 1 file pushed, 0 skipped. 11.0 MB/s (114296792 bytes in 9.950s)
4.4 Attribution des droits d'exécution
powershell
> adb shell chmod 755 /data/local/tmp/frida-server
✅ Statut: Binaire déployé avec succès
<img width="642" height="113" alt="2lab10" src="https://github.com/user-attachments/assets/4a4b1d06-f493-4f29-be8b-cc974abbd798" />

4.5 Lancement du serveur
powershell
# Terminal 1 (reste ouvert)
> adb shell /data/local/tmp/frida-server -l 0.0.0.0
5. Vérification de la connexion
5.1 Configuration des ports
powershell
# Terminal 2
> adb forward tcp:27042 tcp:27042
> adb forward tcp:27043 tcp:27043
5.2 Liste des processus via frida-ps -U
powershell
> frida-ps -U
PID    Name
----   ----
6606   Calendar
7100   Chrome
6797   Clock
6714   Contacts
6861   Gmail
2108   Google
1820   Messages
6829   Phone
6421   Photos
1057   SIM Toolkit
7136   Settings
6937   YouTube
6497   YouTube Music
5461   adbd
✅ Statut: Connexion établie avec succès - Plus de 50 processus détectés
<img width="747" height="887" alt="3lab10" src="https://github.com/user-attachments/assets/fe25b28a-5fa1-4c1d-bc29-3e90c8324628" />

5.3 Applications système identifiées
Application	PID
Settings	7136
Phone	6829
Contacts	6714
Calendar	6606
Gmail	6861
6. Test d'injection - Échec et diagnostic
6.1 Création du script hello.js
powershell
> cd "$env:USERPROFILE\Desktop"
> New-Item -Path hello.js -ItemType File -Force
Fichier créé: C:\Users\setup game\Desktop\hello.js (0 octet - fichier vide)
<img width="470" height="196" alt="4lab10" src="https://github.com/user-attachments/assets/3da8f601-9939-4dc4-bad4-fe5121e7d5a1" />


<img width="711" height="241" alt="5lab10" src="https://github.com/user-attachments/assets/59f1e0b3-a1f0-4eb7-9ee3-fce6e620fedc" />

6.2 Tentative d'injection
powershell
> frida -U -f com.android.settings -l hello.js
Connected to Android Emulator 5554 (id=emulator-5554)
Failed to load script: the connection is closed
6.3 Analyse de l'erreur
Élément	Statut	Observation
Connexion USB	✅ OK	frida-ps -U fonctionne
frida-server	❌ KO	Le serveur n'était pas lancé
Script source	⚠️ Partiel	Fichier hello.js vide
Cause racine: frida-server n'était pas en cours d'exécution sur l'émulateur. La commande frida-ps -U fonctionnait grâce à une session précédente, mais le serveur a été arrêté entre les tests.

7. Problèmes identifiés et correctifs
7.1 Problème #1: "Failed to load script: the connection is closed"
Symptôme:

text
Failed to load script: the connection is closed
Cause: frida-server n'est pas actif sur l'appareil Android.

Diagnostic:

powershell
> adb shell ps | findstr frida
# (Aucun résultat - le serveur ne tourne pas)
Solution appliquée:

powershell
# Étape 1: Lancer frida-server (Terminal 1)
adb shell /data/local/tmp/frida-server -l 0.0.0.0

# Étape 2: Configurer les ports (Terminal 2)
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043

# Étape 3: Vérifier la connexion
frida-ps -U  # Doit fonctionner
7.2 Problème #2: Script hello.js vide
Symptôme: Le script ne contient aucun code JavaScript à exécuter.

Solution: Ajouter le contenu suivant dans hello.js:

javascript
Java.perform(function () {
    console.log("[+] ======================================");
    console.log("[+] Frida Java.perform OK");
    console.log("[+] Injection réussie !");
    console.log("[+] Test validé pour le laboratoire");
    console.log("[+] ======================================");
});
7.3 Problème #3: Version mismatch
Symptôme:

text
unable to communicate with remote frida-server; 
please ensure that major versions match
Cause: Version du client Frida (PC) ≠ Version de frida-server (Android)

Solution:

powershell
# Vérifier les versions
frida --version
adb shell /data/local/tmp/frida-server --version

# Aligner les versions (télécharger la bonne version)
# ou mettre à jour le client
pip install --upgrade frida frida-tools
7.4 Procédure de correction complète
powershell
# === TERMINAL 1 (Serveur) ===
adb shell /data/local/tmp/frida-server -l 0.0.0.0

# === TERMINAL 2 (Client) ===
# Nettoyer les anciens forwards
adb forward --remove-all

# Configurer les ports
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043

# Vérifier la connexion
frida-ps -U

# Injecter le script
frida -U -f com.android.settings -l hello.js
8. Conclusion
8.1 Éléments validés
Composant	Statut
Installation client Frida (pip)	✅ Réussi
Frida CLI accessible	✅ Réussi
Module Python frida importable	✅ Réussi
Connexion ADB à l'émulateur	✅ Réussi
Déploiement frida-server	✅ Réussi
frida-ps -U fonctionnel	✅ Réussi
Injection de script	⚠️ À reprendre
8.2 Recommandations
Pour les prochains tests: Toujours vérifier que frida-server est actif avant d'injecter un script.

Utiliser deux terminaux:

Terminal 1: adb shell /data/local/tmp/frida-server -l 0.0.0.0

Terminal 2: Commandes Frida

Script de vérification rapide:

powershell
# Vérifier l'état du serveur
adb shell ps | findstr frida
8.3 Points d'amélioration
Automatiser le lancement de frida-server avec un script batch

Utiliser des scripts JavaScript plus élaborés pour le hooking

Tester l'injection sur différentes applications (YouTube, Chrome, etc.)
<img width="614" height="291" alt="6lab10" src="https://github.com/user-attachments/assets/6788c9fb-8c6e-46ae-af3f-5e1f3d30eeeb" />

Explorer les hooks natifs et Java avancés

9. Annexes
Annexe A: Commandes utiles
powershell
# Client Frida
frida --version                    # Version du client
frida-ps -U                        # Liste des processus USB
frida-ps -Uai                      # Liste des applications
frida-ps -R                        # Appareils réseau

# Serveur Frida
adb shell /data/local/tmp/frida-server -l 0.0.0.0
adb forward tcp:27042 tcp:27042    # Redirection port principal
adb forward tcp:27043 tcp:27043    # Redirection port secondaire

# Diagnostic ADB
adb devices                        # Liste des appareils
adb shell ps | findstr frida       # Vérifier processus serveur
adb shell getprop ro.product.cpu.abi  # Architecture CPU

# Injection
frida -U -f <package> -l <script.js>   # Lancer et injecter
frida -U -n <process> -l <script.js>   # Attacher à processus existant
Annexe B: Architecture du laboratoire
text
┌─────────────────────────────────────────────────────────────────┐
│                         PC HÔTE (Windows 11)                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Client Frida                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │   │
│  │  │ frida CLI   │  │ frida-tools │  │ Python frida    │  │   │
│  │  │ (17.9.10)   │  │ (14.8.2)    │  │ module (17.9.10)│  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                    ADB Forward (tcp:27042)                       │
│                              │                                   │
├──────────────────────────────┼───────────────────────────────────┤
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              ÉMULATEUR ANDROID (emulator-5554)           │   │
│  │  ┌─────────────────────────────────────────────────────┐│   │
│  │  │                   frida-server                       ││   │
│  │  │                   (17.9.10)                          ││   │
│  │  │            /data/local/tmp/frida-server              ││   │
│  │  └─────────────────────────────────────────────────────┘│   │
│  │                                                          │   │
│  │  Applications cibles:                                    │   │
│  │  - com.android.settings (PID: 7136)                      │   │
│  │  - com.android.contacts (PID: 6714)                      │   │
│  │  - com.google.android.apps.youtube.music (PID: 6497)    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

