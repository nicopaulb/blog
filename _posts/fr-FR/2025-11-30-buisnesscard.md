---
title: "Carte de visite PCB avec NFC"
description: "Une carte de visite électronique épurée intégrant un QR Code et un tag NFC."
date: 2025-11-30
categories: [Embedded]
tags: [stm32, c, nfc]
media_subpath: /assets/img/posts/buisnesscard
lang: fr-FR
image:
  path: cover.jpg
math: true
---

En parcourant divers projets DIY d’électronique sur internet, je suis tombé sur une drôle d'idée : fabriquer une **carte de visite directement sous forme de PCB**. L’avantage d’un PCB est qu’il permet d’ajouter des fonctionnalités interactives, ce qui en fait quelque chose de bien plus original qu’une simple carte en papier. Et dès que l’on se met à chercher, on découvre une multitude de créations et de designs très inspirants.

Parmi toutes ces idées, celle qui m’a le plus convaincu était d’intégrer une **puce NFC**. Ainsi, il suffit d’approcher son smartphone de la carte pour ouvrir automatiquement un site web ou importer ses coordonnées. Voici par exemple une réalisation qui m’a inspiré :
[https://www.instructables.com/PCB-Business-Card-With-NFC/](https://www.instructables.com/PCB-Business-Card-With-NFC/)

Le projet est assez simple, et comme j’avais un week-end libre, j’ai décidé de le réaliser en **deux jours seulement**.

Objectifs du projet :
  - **Créer le PCB** avec toutes mes informations de contact sur la couche sérigraphique
  - **Intégrer une puce NFC** avec récupération d’énergie et une antenne PCB adaptée
  - Ajouter en bonus un **QR Code** pointant vers mes liens personnels

## Création du PCB

Comme d’habitude, j’ai utilisé **KiCad** pour le schéma et le routage.
La première étape fut de choisir le format. Je voulais que la carte tienne dans un portefeuille, j’ai donc repris les dimensions **standard d'une carte bancaire** : ISO ID-1 (85,60 × 53,98 mm).
Pour l’épaisseur, le plus fin est la meilleure option mais j’ai choisi en fonction du coût au moment de la commande.

Une fois le contour défini, j’ai ajouté toutes mes informations personnelles (nom, fonction, email, téléphone, réseaux) sur la face avant, en gardant un style **sobre et épuré**. Je ne voulais pas d’un design trop chargé.
J’ai aussi ajouté un petit symbole NFC pour indiquer que la carte peut être lue sans contact.

![Face avant du PCB](pcb_top.png){: w="600" h="600" }
_Face avant du PCB_

## Ajout du NFC

L’essentiel du travail se situe sur la face arrière du PCB, où se trouvent la puce NFC et l’antenne.

Mais avant cela, un rappel rapide sur la technologie.

### Technologie NFC

Le **NFC (Near Field Communication)** est une technologie de communication à courte portée permettant d’échanger des données lorsque deux appareils sont à seulement **quelques centimètres** l’un de l’autre. Elle fonctionne à **13,56 MHz**.

Par exemple, un smartphone peut lire ou écrire des informations sur un **tag NFC** en la rapprochant simplement.

![Technologie NFC](nfc.png){: w="700" h="700" }
_Technologie NFC_

Pour les systèmes embarqués, le NFC est particulièrement intéressant car le champ généré par le **lecteur** suffit à alimenter un **tag NFC passif**, ce qui évite toute alimentation externe.

### La puce NFC

Pour simplifier l'assemblage, je voulais que **JLCPCB** assemble les composants. J’ai donc parcouru leur bibliothèque de composants et choisi uniquement des puces disponibles en stock.

Mon choix s’est porté sur la **NT3H2111W0FTTJ** de NXP, qui intègre la **récupération d’énergie**, lui permettant de s’alimenter uniquement via le champ NFC.

![Diagramme fonctionnel NT3H2111](NT3H2111_2211_diagram.png){: w="700" h="700" }
_Diagramme fonctionnel NT3H2111_

La puce expose huit broches :
  - **LA/LB** pour l'antenne.
  - **VCC/GND** pour l'alimentation.
  - **VOUT** pour la tension produite via la récupération d'énergie.
  - **SDA/SCL** pour l'interface I2C.
  - **FD** pour la détection de champ.

Pour ce projet, puisque je n’avais besoin que d’un **tag NFC passif** contenant une URL, la communication I2C et le pin de détection de champ sont inutiles, donc ces broches sont laissées non-connectées.

Pour alimenter la puce, j’ai simplement relié **VCC et VOUT** ensemble, en s’appuyant entièrement sur la récupération d’énergie.

Je voulais également ajouter une **LED** qui s’allume lorsque la carte est scannée. Avec environ 3 V disponibles depuis VOUT et une LED C84256 (2V forward drop, ~20mA forward current), j’ai calculé la résistance série :

$$
R = \frac{V_{CC} - V_{LED}}{I_{LED}} = \frac{3 - 2}{0.020} = 50 \Omega
$$

J’ai donc choisi une résistance de **47Ω**.

Le datasheet recommande également un **condensateur de 150–220nF** entre VOUT et GND pour stabiliser la tension pendant la modulation RF.

Toutes ces informations sont résumé dans le schéma d'exemple donné dans la datasheet :

![Energy harvesting example circuit](energy_harvesting_example.png){: w="700" h="700" }
_Exemple de circuit de récupération d’énergie_

Voici mon schéma final :

![PCB Buisness Card Schema](pcb_schema.png){: w="700" h="700" }
_Schéma de la carte_

### Antenne NFC

Pour fonctionner, la puce nécessite une antenne. Plutôt que d’en utiliser une externe, j’ai choisi une **antenne PCB en boucle**.

Le calcul de l’inductance dépend de nombreux paramètres (épaisseur, largeur de piste, espacement, nombre de tours…), mais NXP fournit des **antennes exemples prêtes à l’emploi** dans la note d’application [AN11276](https://community.nxp.com/pwmxy87654/attachments/pwmxy87654/nfc/6155/1/AN11276.pdf).
J’ai donc importé la géométrie fournie dans KiCad et l’ai simplement connectée à LA/LB.

Une règle importante : **aucun plan de masse ou d’alimentation ne doit passer sous ou au-dessus de l’antenne**, sous peine de bloquer le champ magnétique NFC.

![PCB Buisness Card](pcb.png){: w="700" h="700" }
_Carte de visite PCB_

## Ajout du QR Code

J’ai aussi intégré un **QR Code** sur la face arrière, pour les personnes dont le téléphone ne gère pas le NFC.
La LED se trouve au centre du QR Code, uniquement pour des raisons esthétique.

Le QR Code a été généré via [https://www.qrcode-monkey.com](https://www.qrcode-monkey.com) et pointe vers un de mes **sous-domaine** : **[qr.beniserv.fr](https://qr.beniserv.fr)**.

Grâce à cette redirection, je peux modifier la destination finale à tout moment sans avoir à modifier le QR Code et refaire le PCB.

## Fabrication

![PCB Buisness Card 3D Render](pcb_3d.png){: w="700" h="700" }
_Rendu 3D du PCB_

Comme souvent, j’ai ensuite commandé le PCB chez **JLCPCB**. Ils se sont occupés d’assembler les composants NFC grâce au BOM et aux fichiers de placement générés automatiquement via le très utile [plugin KiCAD pour JLCPCB](https://github.com/Bouni/kicad-jlcpcb-tools).

J’ai choisi une finition **noire** pour un look professionnel. Pour l’épaisseur, je voulais me rapprocher d’une carte réelle, et la meilleure option économique était **1 mm**.

JLCPCB m’a aussi demandé d’ajouter deux trous de fixation afin que le PCB puisse être correctement maintenu pendant l’assemblage

> Le projet KiCad complet et les fichiers de fabrication sont disponibles ici : [https://github.com/nicopaulb/PCBBuisnessCard](https://github.com/nicopaulb/PCBBuisnessCard).

Deux semaines plus tard, les cartes sont arrivées. J’ai programmé la puce NFC via l’application **NXP TagWriter** sur Android, pour y ajouter mes informations (nom/prénom, numéro de téléphone, adresse email et site internet).

![Photo de la carte de visite](photo.jpg){: w="500" h="500" }
_Photo de la carte de visite_

## Démonstration

{% include embed/youtube.html id='LewoWHGI1Rg' %}
