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

<img width="870" height="361" alt="1lab10" src="https://github.com/user-attachments/assets/966ee7a3-ff78-42fe-ad34-130098847c4a" />

