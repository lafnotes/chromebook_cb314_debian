**Date**
03/09/2026

**Description**
Passage du ChromeBook CB314 (Celeron) sous Debian 13

### Pré requis
Intel ! (Celeron)

### Préparatifs
**Téléchargement de l'image depuis un autre PC**
https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/

**Graver l'iso sur une clé usb**
Utiliser rufus ou balena etcher

**Avec rufus:**
Taille de partition persistente : 0
Schéma : GPT
Système de fichier FAT32
Le reste par défaut

**A noter**
Quand on retire la clé USB et remet, elle n est plus détectée (os Windows), c'est normal.

### Actions
#### Ouverture du capot et débranchement du connecteur côté carte mère
Sur ce Chromebook, pour installer un OS différent de ChromeOS, il faut ouvrir le capot et débrancher le câble reliant la batterie à la carte mère !
Une fois ceci effectué, brancher le chargeur secteur.
Ne pas remonter le capot, la batterie sera rebranchée après la manipulation de flashage du bios UEFI.

#### Actions logicielles
Chromebook éteint, maintenir Échap + Actualiser (flèche circulaire) puis appuyer brièvement sur Alimentation.
À l'écran d'avertissement, appuie sur Ctrl + D puis valide avec Entrée
Laisse le Chromebook réinitialiser le système

Une fois réinitialisé, connecter le Wi-Fi (inutile de se connecter à son compte Google).
Ouvrir la console VT2 : presser simultanément Ctrl + Alt + F2 (ou Ctrl + Alt + Flèche vers l'avant).
Saisir l'identifiant : chronos (pas de mot de passe requis).
Attention le clavier sera en qwerty. Prendre une equivalencer Azerty au besoin.
Exécuter le script officiel MrChromebox :
```
cd
curl -LO mrchromebox.tech/firmware-util.sh
sudo bash firmware-util.sh
```

Dans le menu qui s'ouvre, tape le numéro correspondant à Install/Update Full ROM Firmware et valide
Tape le numéro correspondant à Install/Update Full ROM Firmware (UEFI) (c'est généralement le choix 1 ou 2) et valide avec Entrée.

Le script va demander s'il faut faire une sauvegarde du BIOS d'origine. Répondre N (No) pour aller au plus vite, sauf si deuxième clé USB sous la main et que souhait de la conserver.
Il va demander une confirmation de sécurité : taper Y (Yes) puis Entrée (ou réécrir la phrase de confirmation demandée, comme I ACCEPT).

Le script va télécharger et installer le nouveau BIOS UEFI (cela prend environ 1 à 2 minutes). Ne pas éteindre l'appareil pendant cette phase.

Une fois l'opération terminée, éteins l'appareil (sudo poweroff).

#### Rebranchement du connecteur côté carte mère et fermeture du capot 
Rebrancher le connecteur entre la batterie et la carte mère.
Fermer le capot.
Brancher la clé usb contenant le boot Linux.

Brancher le chargeur secteur USB-C sur le Chromebook.
Regarder le petit voyant LED à côté du port USB-C (il devrait s'allumer en orange ou en blanc).
Appuyer sur le bouton d'alimentation.

S'il ne démarre pas:
S'assurer que le chargeur secteur est bien branché.
Maintenir enfoncée la touche Actualiser (la flèche en cercle sur la ligne du haut).
Tout en maintenant Actualiser, appuyer sur le bouton Alimentation pendant 2 secondes, puis relâcher les deux.

#### Install !
Au démarrage, le logo du nouveau BIOS UEFI apparaît (lapin).
Se rendre dans les paramètres puis sélectionner la clé USB dans le menu de boot.
Lancer "Start installer"

Suivre l'assistant :

    Langue / Clavier : Français / AZERTY.

    Réseau : son Wi-Fi.

    Partitionnement : Choisis Utiliser le disque entier (sélectionne la mémoire interne eMMC du Chromebook, généralement nommée mmcblk0) sans LVM.

    Sélection des logiciels : Quand la grille de choix s'affiche, vérifie que XFCE est bien coché (décoche GNOME ou KDE si cochés) ainsi que Utilitaires standard du système.


A un moment il va demander un driver pour Intel/SOF/community/sof-glk.ri.
Cliquer sur Non (No) ou Ignorer lorsque l'installateur demande d'insérer un média externe pour ce fichier.
Poursuivre l'installation
Installer le pilote une fois sur Debian.
Pas besoin de domaine.
Valider l'installation du secteur de démarrage (GRUB) sur le disque principal.
Retirer la clé USB à la fin et redémarre.

### Config Post Installation

**2 problèmes constatés :**
- Internet ne fonctionne pas
- Pas de son


#### Correction du Problème Internet
Ce problème vient du fait que l'OS n'est pas à l'heure.
Pour le mettre  l'heure
Ouvrir le temrinal
```
sudo apt update
sudo apt install systemd-timesyncd
sudo systemctl enable --now systemd-timesyncd
sudo timedatectl set-ntp true
```
Rebooter.
Au reboot, taper 
```
timedatectl
```
Les champs doivent être correctement renseignés.


#### Correction du Problème de son

```
sudo sed -i 's/main/main non-free non-free-firmware/g' /etc/apt/sources.list
sudo apt update
sudo apt install -y firmware-intel-sound firmware-linux-nonfree pipewire-audio
sudo reboot
```

Tester le son
```
speaker-test -c 2 -t wav
```

Le probleme est qu'ALSA (le système audio Linux) ne trouve pas le bon dossier de réglages parce que le nom a une micro-différence (sof-glkrt5682ma au lieu de sof-glk-rt5682).

Vérification
```
ls -d /usr/share/alsa/ucm2/Intel/sof-glk*
```
Vérifier ce qui s'affiche à l'écran
Par ex
/usr/share/alsa/ucm2/Intel/sof-glkda7219max

Dans ce cas, le Chromebook utilise un codec Dialog DA7219 + Maxim (sof-glkda7219max) et non Realtek.
Pour corriger :
```
ln -s /usr/share/alsa/ucm2/Intel/sof-glkda7219max /usr/share/alsa/ucm2/Intel/sof-glkrt5682ma
alsactl init
exit
reboot
```
(L'erreur `-2` ne doit plus apparaître sur alsainit).
Au redémarrage le son devrait fonctionner.

