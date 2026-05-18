# LAB10-SECC
LABORATOIRE FRIDA - RAPPORT D'INSTALLATION ET DE VALIDATION

Environnement: Windows 11 / PowerShell
Cible Android: Émulateur Android (emulator-5554)

Table des matières
Objectifs du laboratoire

Prérequis et environnement

Installation du client Frida

Déploiement de frida-server

Vérification de la connexion

Test d'injection - Échec et diagnostic

Problèmes identifiés et correctifs

Conclusion

Annexes

1. Objectifs du laboratoire <a name="objectifs"></a>
Installer et configurer Frida (client Python + CLI) sur Windows

Déployer frida-server sur un émulateur Android

Établir une connexion fonctionnelle entre le client et le serveur

Injecter un script JavaScript minimal pour valider l'environnement

Diagnostiquer les problèmes courants d'installation

2. Prérequis et environnement <a name="prerequis"></a>
Composant	Version
Système d'exploitation	Windows 11
Shell	PowerShell
Python	3.14
Pip	Dernière version
ADB (Platform Tools)	Dernière version
Émulateur Android	AVD (API 34)
Cible	emulator-5554
3. Installation du client Frida <a name="installation-client"></a>
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

4. Déploiement de frida-server sur Android <a name="deploiement-server"></a>
4.1 Vérification de la connexion ADB
powershell
> adb devices
List of devices attached
emulator-5554    device
✅ Statut: Émulateur détecté et autorisé

4.2 Copie du binaire frida-server
powershell
> adb push frida-server /data/local/tmp/
frida-server: 1 file pushed, 0 skipped. 11.0 MB/s (114296792 bytes in 9.950s)
4.3 Attribution des droits d'exécution
powershell
> adb shell chmod 755 /data/local/tmp/frida-server
✅ Statut: Binaire déployé avec succès

5. Vérification de la connexion <a name="verification-connexion"></a>
5.1 Liste des processus via frida-ps -U
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
...
✅ Statut: Connexion établie avec succès - Plus de 50 processus détectés

5.2 Applications système identifiées
Application	PID
Settings	7136
Phone	6829
Contacts	6714
Calendar	6606
Gmail	6861
6. Test d'injection - Échec et diagnostic <a name="test-injection"></a>
6.1 Création du script hello.js
powershell
> cd "$env:USERPROFILE\Desktop"
> New-Item -Path hello.js -ItemType File -Force
Fichier créé: C:\Users\setup game\Desktop\hello.js (0 octet - fichier vide)

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

7. Problèmes identifiés et correctifs <a name="depannage"></a>
7.1 Problème #1: "Failed to load script: the connection is closed"
Symptôme:

text
Failed to load script: the connection is closed
Cause:
frida-server n'est pas actif sur l'appareil Android.

Diagnostic:

powershell
> adb shell ps | findstr frida
# (Aucun résultat - le serveur ne tourne pas)
Solution appliquée:

Lancer frida-server dans un terminal dédié:

powershell
> adb shell /data/local/tmp/frida-server -l 0.0.0.0
Configurer la redirection des ports:

powershell
> adb forward tcp:27042 tcp:27042
> adb forward tcp:27043 tcp:27043
Vérifier l'état:

powershell
> frida-ps -U  # Doit fonctionner
7.2 Problème #2: Script hello.js vide
Symptôme:
Le script ne contient aucun code JavaScript à exécuter.

Solution:
Ajouter le contenu suivant dans hello.js:

javascript
Java.perform(function () {
    console.log("[+] ======================================");
    console.log("[+] Frida Java.perform OK");
    console.log("[+] Injection réussie !");
    console.log("[+] Test validé pour le laboratoire");
    console.log("[+] ======================================");
});
7.3 Procédure de correction complète
powershell
# Étape 1: Démarrer frida-server (Terminal 1)
adb shell /data/local/tmp/frida-server -l 0.0.0.0

# Étape 2: Configurer les ports (Terminal 2)
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043

# Étape 3: Vérifier la connexion
frida-ps -U

# Étape 4: Injecter le script
frida -U -f com.android.settings -l hello.js
8. Conclusion <a name="conclusion"></a>
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

9. Annexes <a name="annexes"></a>
Annexe A: Commandes utiles
powershell
# Client
frida --version
frida-ps -U          # Liste des processus
frida-ps -Uai        # Liste des applications

# Serveur
adb shell /data/local/tmp/frida-server -l 0.0.0.0
adb forward tcp:27042 tcp:27042

# Diagnostic
adb devices
adb shell ps | findstr frida
Annexe B: Architecture du laboratoire
text
┌─────────────────┐     USB/ADB      ┌─────────────────┐
│   PC Windows    │ ◄──────────────► │   Émulateur     │
│                 │                   │   Android       │
│  ┌───────────┐  │    tcp:27042     │                 │
│  │ Frida     │  │ ◄──────────────► │ ┌───────────┐   │
│  │ Client    │  │                   │ │ frida-    │   │
│  │ 17.9.10   │  │                   │ │ server    │   │
│  └───────────┘  │                   │ │ 17.9.10   │   │
│                 │                   │ └───────────┘   │
└─────────────────┘                   └─────────────────┘
Annexe C: Versions logicielles
Logiciel	Version
Windows	11
Python	3.14
Frida	17.9.10
Frida-tools	14.8.2
Android Emulator	API 34
