# Installation et prise en main de Frida pour l’analyse dynamique Android

## Objectif

L’objectif de ce laboratoire est de mettre en place **Frida**, un framework d’instrumentation dynamique, afin de réaliser une analyse de sécurité sur une application Android.  
Ce TP couvre :

- l’installation du client Frida sur la machine hôte ;
- le déploiement de `frida-server` sur un appareil Android ;
- la validation de la communication entre le PC et l’appareil ;
- l’injection de scripts JavaScript dans un processus Android ;
- l’observation de comportements applicatifs liés au réseau, au stockage local et à certaines vérifications de sécurité.

---

# Environnement de travail

## Matériel utilisé

- PC hôte : Windows 10/11
- Smartphone Android / Émulateur Android
- Câble USB
- Connexion Internet

## Logiciels utilisés

- Python 3.10+
- pip
- ADB (Android Platform Tools)
- Frida
- frida-tools
- PowerShell / CMD
- Android Emulator (AVD)
- 7-Zip (pour extraction `.xz`)

---

# Étape 1 : Installation de Frida sur la machine hôte

## Installation de Python

Vérification de l’installation :

```bash
python --version
pip --version
Installation du client Frida
pip install --upgrade frida frida-tools
Vérification
frida --version
frida-ps --version
python -c "import frida; print(frida.__version__)"
Résultat attendu

Affichage de la version installée :

<img width="870" height="361" alt="1lab10" src="https://github.com/user-attachments/assets/b9825f29-6e55-4d74-abe9-386a99b57d6c" />

Étape 2 : Installation et vérification d’ADB

ADB est utilisé pour communiquer avec l’appareil Android.

Vérification
adb version
adb devices
Résultat attendu


L’appareil doit apparaître comme :

device

et non :

unauthorized
Étape 3 : Déploiement de frida-server sur Android
3.1 Vérification de l’architecture
adb shell getprop ro.product.cpu.abi
Exemple
x86_64
3.2 Téléchargement de frida-server

Téléchargement depuis la page officielle :

Frida Releases (GitHub)

Exemple :

frida-server-17.9.1-android-x86_64.xz
3.3 Extraction

Sous Windows :

Extraction avec 7-Zip

Le fichier obtenu est renommé :

frida-server
3.4 Copie sur l’appareil
adb push frida-server /data/local/tmp/
3.5 Attribution des permissions
adb shell chmod 755 /data/local/tmp/frida-server
3.6 Lancement du serveur
adb shell /data/local/tmp/frida-server -l 0.0.0.0

ou en arrière-plan :

adb shell "nohup /data/local/tmp/frida-server -l 0.0.0.0 >/dev/null 2>&1 &"
3.7 Vérification
adb shell ps | grep frida
3.8 Redirection de ports
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043

<img width="642" height="113" alt="2lab10" src="https://github.com/user-attachments/assets/8074609f-d842-4fc0-b235-c67bf7228bc0" />

Étape 4 : Vérification de la connexion

Depuis le PC :

frida-ps -U

ou :

frida-ps -Uai
Résultat attendu

Liste des processus ou applications Android détectés.
<img width="747" height="887" alt="3lab10" src="https://github.com/user-attachments/assets/fbe161f2-6d8a-425b-9ec4-e17f7d789d32" />


Étape 5 : Test d’injection
5.1 Script Java simple

<img width="470" height="196" alt="4lab10" src="https://github.com/user-attachments/assets/ae008a3e-1a4f-4610-bb2b-2e161a3ccc85" />

<img width="711" height="241" alt="5lab10" src="https://github.com/user-attachments/assets/e5adcd98-4ba7-4868-9956-bdf2228fcc7b" />

Création de hello.js

Java.perform(function () {
    console.log("[+] Frida Java.perform OK");
});

Lancement :

frida -U -f com.example.app -l hello.js

Puis :

%resume
Résultat attendu
[+] Frida Java.perform OK
5.2 Script natif simple

Création de hello_native.js

console.log("[+] Script chargé");

Interceptor.attach(Module.getExportByName(null, "recv"), {
    onEnter(args) {
        console.log("[+] recv appelée");
    }
});

Injection :

frida -U -n "NomDuProcessus" -l hello_native.js
Résultat attendu
[+] Script chargé
[+] recv appelée

<img width="614" height="291" alt="6lab10" src="https://github.com/user-attachments/assets/e652f884-9502-4f88-b3a7-8ff827a597e4" />
