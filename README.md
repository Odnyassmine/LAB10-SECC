

---

 Étape 1 : Installation de Frida sur la machine hôte

Installation de Python

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

