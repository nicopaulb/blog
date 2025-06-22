---
title: "Garmin Bus Tracker : Widget pour montres"
description: "Un widget compatible avec les montres Garmin pour suivre tous les bus DeLijn en Flandre."
date: 2025-01-30
categories: [Embedded]
tags : [c, network, garmin, watch]
media_subpath : /assets/img/posts/garminDeLijn
lang : fr-FR
image : 
    path : garmin_screen.png
---

Un an après mon dernier projet Bus Tracker pour les bus à Toulouse, j'ai déménagé en Belgique et j'ai décidé de créer un outil similaire, mais pour les montres Garmin.

> If you want to learn more about my **Bus Tracker** project, you can read more about it [here]({% post_url en/2023-02-10-bustracker %}).
{: .prompt-tip }

Je porte personnellement une Forunner 245 et j'ai toujours voulu pouvoir savoir exactement quand le bus allait venir me chercher.
J'ai donc créé cette application pour suivre tous les bus DeLijn en Flandre et afficher l'heure d'arrivée du prochain bus.

## API de données de bus DeLijn

La première étape a consisté à obtenir l'accès à l'API DeLijn pour recevoir toutes les informations en temps réel sur les bus.
Heureusement, son utilisation est gratuite et il suffit de créer un compte sur le [portail De Lijn Open Data](https://data.delijn.be/).

Une fois mes clés en main, DeLijn m'a fourni trois API différentes :
- **GTFS Static** : API standard pour obtenir des informations statiques sur les transports publics
- **GTFS Realtime** : API standard pour obtenir des informations en temps réel sur les transports publics
- **Open Data Services** : API personnalisée pour obtenir des informations en temps réel et statiques sur les transports publics

D'après cette brève description, vous pouvez déduire que pour obtenir des informations en temps réel, il est possible d'utiliser soit l'API GTFS Realtime, soit l'API Open Data Services. 
Mais avant de vous dire laquelle j'ai choisi d'utiliser, examinons la différence entre les deux API.

### GTFS

General Transit Feed Specification (GTFS) est un format standard pour les horaires des transports publics créé par Google en 2005. Initialement créé pour intégrer les données de transport dans Google Maps, il a depuis été adopté par la plupart des autres services de navigation (Apple Maps, Moovit, ...) et services de transport public.
Il permet aux agences de transport de publier leurs horaires, leurs itinéraires et leurs données d'arrêts dans un format que les services de navigation peuvent facilement intégrer.

Un flux GTFS est un ensemble de fichiers qui décrivent le réseau de transports publics, y compris les itinéraires, les arrêts et les horaires. Cependant, un flux GTFS ne suffit pas pour obtenir des informations en temps réel, car il ne contient que les horaires statiques/planifiés et non les retards en temps réel ou les mises à jour des trajets.

Pour obtenir ces informations supplémentaires en temps réel, vous devez récupérer un autre flux, GTFS-RT (GTFS Realtime), qui est une extension de GTFS. Ce flux ne contient que les retards relatifs pour chaque trajet et non l'heure d'arrivée absolue. Vous devez donc combiner les résultats des deux API GTFS pour calculer l'heure d'arrivée/de départ absolue des véhicules de transport public.

> Pour plus d'informations, consultez le [site web officiel de GTFS](https://gtfs.org).
{: .prompt-info }

### Services de données ouvertes

L'API Open Data Services est une API personnalisée de DeLijn qui fournit des informations en temps réel et statiques sur les transports publics. Elle n'est pas standard, mais offre les mêmes informations que les API GTFS au format JSON et ne nécessite pas de requêtes supplémentaires pour récupérer les informations en temps réel.

> Pour plus d'informations, consultez le [site web DeLijn Data](https://data.delijn.be/product#product=5978abf6e8b4390cc83196ad).
{: .prompt-info }

Dans un environnement intégré avec une mémoire et une puissance de traitement limitées, l'API Open Data Services m'a semblé être le meilleur choix. La réponse étant un objet JSON, elle peut être facilement analysée et ne nécessite pas de naviguer entre différents fichiers et d'exécuter différentes requêtes HTTP comme les API GTFS.

## SDK Garmin

Pour développer une application pour un produit Garmin, vous devez utiliser le [Connect IQ SDK](https://developer.garmin.com/connect-iq/overview/) de Garmin.
Le SDK permet de créer des applications natives, des widgets et des champs de données pour toutes les montres connectées Garmin. 

### Monkey C

Garmin a développé son propre langage de programmation appelé **Monkey C** pour utiliser le SDK. La syntaxe est dérivée de plusieurs langages (C, C#, Java, Javascript, ...) et est assez facile à comprendre.

![Monkey C](monkeyc.png){: w="300"}
_Monkey C_

> Pour plus d'informations, consultez [Garmin Monkey C](https://developer.garmin.com/connect-iq/monkey-c/).
{: .prompt-info }

L'une des premières choses que j'ai faites a été d'installer l'extension officielle [Monkey C language support extension for VS Code](https://marketplace.visualstudio.com/items?itemName=garmin.monkey-c). 

En plus d'offrir la coloration syntaxique et la complétion de code, elle facilite également l'interaction avec le Connect IQ SKD en fournissant quelques commandes utiles (créer un nouveau projet, compiler, ouvrir des exemples, ouvrir le gestionnaire SDK, ...).

### Émulateur

Le gestionnaire SDK est également fourni avec des émulateurs de périphériques permettant d'exécuter directement l'application depuis l'ordinateur.
Il est très pratique pour exécuter rapidement l'application sans avoir à l'installer sur la montre ou pour pouvoir la tester sur différentes montres.

![Émulateur Garmin](emulator.png){: w="300"}
_Émulateur Garmin_

## Application

### Architecture
The application is quite simple, it fetchs the time of arrival of the next bus from the DeLijn Open Data Services API and displays a countdown on the watch.
The countdown is updated every second and a new API request is made every minute or, if the refresh button is pressed, to correct the countdown if the bus is delayed or advanced. This way, the informations displayed are always up to date.

![Architecture Diagram](schema.png){: w="300"}
_Architecture Diagram_

### Settings
In the settings, accessible from the Connect IQ application, you can select which bus stop and bus lines you want to track. You should also set your own DeLijn API key, so you don't have to worry about rate limits.

![Settings](settings.png){: w="200"}
_Settings_

The API request interval can be changed as well. 

### Interface
The countdown (in minutes) to the next bus is displayed. The interface can shows up to two bus stop information, one on each line.

The countdown color indicate if the estimated bus arrival time (based on the GPS position) is on schedule, late or early comparerly to the static timetable information :
- <span style="color:purple;font-weight:bold">Purple</span> : Bus is estimated to arrive earlier than expected
- <span style="color:green;font-weight:bold">Green</span>: Bus is on time.
- <span style="color:red;font-weight:bold">Red</span> : Bus is estimated to arrive later than expected.

![Interface](interface.png){: w="200"}
_Interface_

> The code is available on Github here : [https://github.com/nicopaulb/Garmin-DeLijn-Bus-Tracker](https://github.com/nicopaulb/Garmin-DeLijn-Bus-Tracker).
{: .prompt-tip }

## Garmin Connect IQ Store

Publishing the application was quite straightforward if you compare it to a lot of others stores and platforms. Simply create an account, upload the application, update the store details and you're done.

> The Garmin application is available on the Connect IQ Store here : [https://apps.garmin.com/fr-FR/apps/1d2b5826-ae2e-4bb9-a6e7-76e3e6b1ef5a
](https://apps.garmin.com/fr-FR/apps/1d2b5826-ae2e-4bb9-a6e7-76e3e6b1ef5a).
{: .prompt-tip }