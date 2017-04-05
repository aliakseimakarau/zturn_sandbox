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

### Installation de Vivado (à bouger)
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

On peut lancer Vivado depuis le menu Ubuntu !

### Importation du projet d'exemple
On se base sur la doc 
`01-Document/UserManual/English/Z-Turn Board Programmable Logic Development Manual.pdf`

La doc est centrée sur la version 7010 de la carte, or nous avons la 7020. Attention donc.
Dans `3.1.1 Open Xilinx SDK`, la doc ne dit pas que des projets d'exemple sont disponibles sur le CD :
* 05-ProgrammableLogic_Source/7Z020/
On va essayer avec `mys-xc7z020-trd.rar`.
```bash
cp mys-xc7z020-trd.rar ~/documents/marcel/zturn_sandbox/template/
unrar x mys-xc7z020-trd.rar
```
J'ai copié ce que j'appelle pour l'instant `template` dans tests/test1.
On lance Vivado, puis `Open project` et on va chercher le bon fichier : 
`~/documents/marcel/zturn_sandbox/tests/test1/mys-xc7z020-trd/mys-xc7z020-trd.xpr`

On peut maintenant utiliser la fameuse `Method One, From Existing`.
* `File > Launch SDK`
* On laisse tout à `< Local to Project >`
Vivado suggère de ré-exporter le design qui n'est plus à jour. Pourquoi pas ?
> Ça a pas été long finalement, ça a pas dû tout recompiler. Ouf.

### Compilation du FSBL
* Nettoyage du projet
`Project > Clean`, puis `Clean all projects`
> Il y a l'air d'avoir pas mal de problèmes de path (chemins absolus windows, lol).
> On va tenter de mixer avec la `Method Two, Create a new project` pour créer un FSBL saint.

On supprime les projets `fsbl` et `fsbl_bsp`, puis : 
`File > New > Application Project`
> Au lieu de suivre la doc et de recréer un nouveau `Target Hardware` (il y en a déjà deux), on peut directement sélectionner `design_1_wrapper_hw_platform_0`

On appuye maintenant sur `Finish`.

On peut (enfin) compiler le FSBL :
`Project > Build All`
> Tout a l'air de s'être bien passé, le dossier 
> `~/documents/marcel/zturn_sandbox/tests/test1/mys-xc7z020-trd/mys-xc7z020-trd.sdk/fsbl/Debug`
> contient bien un fichier `fsbl.elf`.

### Génération du U-boot
La doc nous renvoie vers une autre doc : 
`01-Document/UserManual/English/Z-turn Board Linux Development Manual`.

Avant d'attaquer la partie `3.1 Compile Bootloader`, il faut setup l'environnement de cross-compilation.
On saute donc au `Chapter 2 Build Linux Development Environment` de la même doc.

À la racine du projet :
```bash
mkdir linux
cp -r <DVD>/04-Linux_Source/* linux/
chmod +w -R linux/	# Le DVD est en lecture seule
cd linux/Toolchain
tar -xvjf Sourcery_CodeBench_Lite_for_Xilinx_GNU_Linux.tar.bz2
export ZTURN_SANDBOX=$HOME/documents/marcel/zturn_sandbox
export PATH=$PATH:$ZTURN_SANDBOX/linux/Toolchain/CodeSourcery/Sourcery_CodeBench_Lite_for_Xilinx_GNU_Linux/bin
export CROSS_COMPILE=arm-xilinx-linux-gnueabi-
```
> TODO : mettre ce qu'il faut dans le `.bashrc`

> Il y en a pour 1.4G, donc le dossier `linux` est ajouté dans le `.gitignore`.

Installer les différents outils :
```bash
sudo apt-get install build-essential git-core libncurses5-dev \
flex bison texinfo zip tar zlib1g-dev gettext \
gperf libsdl-dev libesd0-dev libwxgtk2.6-dev \
uboot-mkimage \
g++ xz-utils
```

> `libwxgtk2.6-dev` n'existe pas, on tente `libwxgtk3.0-dev` à la place.

> `uboot-mkimage` remplacé par `u-boot-tools`.

Ça donne :
```bash
sudo apt-get install build-essential git-core libncurses5-dev \
flex bison texinfo zip tar zlib1g-dev gettext \
gperf libsdl-dev libesd0-dev libwxgtk3.0-dev \
u-boot-tools \
g++ xz-utils
```

On peut maintenant continuer vers la compilation de u-boot :
```bash
cd linux/Bootloader/
tar -jxvf u-boot-xlnx.tar.bz2
cd u-boot-xlnx
#Compile the source code
make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- distclean
make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- zynq_zturn_config
make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi-
```
Après compilation, renommer l'exécutable `u-boot` en `u-boot.elf`.
> TODO : en fait on fera ça en le copiant plus tard

> On peut alors retourner sur la doc principale 
> `01-Document/UserManual/English/Z-Turn Board Programmable Logic Development Manual.pdf`.

### Génération du Bitstream
* La doc nous dit d'extraire le fichier `mys_xc7z020_trd.zip` qu'on a déjà extrait plus haut (wut?),
* On retourne sous Vivado (normalement, on l'a pas fermé),
* On clique sur `Generate Bitstream`,
* And now, we wait...

Une fois le bitstream généré, on ré-exporte le projet : `File > Export > Export Hardware`, 
et on coche `Include Bitstream`.

### Création du boot.bin
À la racine du projet :
```bash
mkdir boot
cp tests/test1/mys-xc7z020-trd/mys-xc7z020-trd.sdk/fsbl/Debug/fsbl.elf boot/

cd boot
```
Créez un fichier `boot.bif` :
```
the_ROM_image:
{
    [bootloader]./fsbl.elf
    ./u-boot.elf
}
```
Puis exécutez
```bash
bootgen -image boot.bif -o boot.bin -w on
```

> TODO : je sais pas trop quoi faire par la suite...

## Programmation "bare metal"
Doc : 01-Document/UserManual/English/MYiR Zynq FPGA Training.pdf

## Setup l'environement de cross-compile
> TODO : Détaillé dans 01-Document/UserManual/English/Z-turn Board Linux Development Manual.pdf

## Refaire une image 
> TODO : Détaillé dans 01-Document/UserManual/English/Z-turn Board Linux Development Manual.pdf


