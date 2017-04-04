# zturn_sandbox
## Prise en main "soft"
* L'objectif ici est simplement de faire exécuter à la carte un petit programme en C.

La carte démarre sous linux. La LED blanche clignote.
On peut avoir un terminal série en se connectant en USB sur le connecteur USB_UART, avec minicom.

Quand on connecte un cable ethernet, la carte récupère une adresse IP par DHCP. 
Pour se connecter en ssh, il faut installer openssh-server.
Il faut également donner un mot de passe avec passwd.

J'ai testé les périphériques (le buzzer, niark niark).
Les sources sont dans le CD : 04-Linux_Source/Examples/buzzer/
J'ai copié les sources sur la carte, je les ai compilé dessus et j'ai testé.
Ça fait pouit.

Par contre, parfois ça fait freezer la carte.
D'ailleurs la carte freeze souvent. À voir.

## Prise en main "hard"
* L'objectif est de mettre une IP sur la zone reconfigurable

* Selon la doc : 01-Document/UserManual/English/Z-turn Board Product User Manual.pdf : 
"""The RGB LEDs are controlled directly by software logic PL; If SW4 is set "HIGH",
the on/off of RGB component is controlled by the SW1, SW2, and SW3, respectively;
and if SW4 is set “LOW”, the RGB LEDs is accordance with PS."""

* La doc utilise Vivado 2014.4 : on va pas faire les malins et faire la même chose.
* Après bataille pour retrouver les identifiants/mots de passes, on télécharge :
	* Vivado 2014.4 Full Image for Linux with SDK (TAR/GZIP - 4.91 GB) 
	* https://xilinx-ax-dl.entitlenow.com/dl/ul/2014/11/19/R209863483/Xilinx_Vivado_SDK_Lin_2014.4_1119_1.tar.gz/618aabdffd2705c4089e260c19c68606/58E3E6B8?akdm=0&filename=Xilinx_Vivado_SDK_Lin_2014.4_1119_1.tar.gz

* Installation :
	* On choisit donc "Vivado System Edition"
	* On sélectionne "Software Development Kit", mais on dégage les FPGA inutiles (7-series et Ultrascale).

* License :
	* https://www.xilinx.com/getlicense
	* ISE WebPACK License
	* Load License -> Copy License

* On peut lancer Vivado depuis le menu Ubuntu !


## Programmation "bare metal"
Doc : 01-Document/UserManual/English/MYiR Zynq FPGA Training.pdf

## Setup l'environement de cross-compile
TODO : Détaillé dans 01-Document/UserManual/English/Z-turn Board Linux Development Manual.pdf

## Refaire une image 
TODO : Détaillé dans 01-Document/UserManual/English/Z-turn Board Linux Development Manual.pdf


