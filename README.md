# IoT-OverSerialProtocol
Protocol over serial connection (UART, I²C...)

## Besoin

### Historique

Trop long !? [Aller à la synthèse](#synthese).

#### Robotique
Dans mon projet de robot le MCU de l'Arduino a atteint ses limites : quantité de mémoire et puissance de calcul. Seul un MPU pourra accompagner le robot dans son analyse de l'environnement. Le choix s'est porté pour un Raspberry Pi Zéro (modèle W pour faciliter l'accès à distance pendat le développemtn, modèle sans wifi pour augmenter l'autonomie pour un fonctionnement nominal).
Il y a donc 2 cerveaux dans le projet. Chacun prendra les décisions qui lui son propres. L'arduino gère le déplacement bas niveau (suivi d'un cap, arrêt sur obstacle) et le Raspberry décide de la direction à suivre (analyse des obstacle, évitement).
Les 2 modules doivent communiquer pour échanger des informations telles que leur état, le chemin à suivre...
La communication est bi-directionnelle, asynchrone sans notion de maître/esclave. Le plus simple est de câbler une liaison de type série. Pour le matériel en question sont disponibles l'UART et l'I²C. L'I²C imposant un maître il ne reste plus que l'UART.

#### Domotique
La télécommande domotique contiendra fort probablement un module ESP (8266 ou 32) et un MCU AVR (ATTiny84). Ils doivent aussi communiquer. Ici l'I²C est préférable car l'ESP est le maître, mais aussi parce que le nombre de pins est limité et l'I²C est déjà connecté donc disponible.


### Synthèse
Créer un protocole qui permet facilement de faire communiquer 2 modules électroniques sur une liaison de type Série.

### Cahier des charges

* **au dessus d'une liaison Série**
* **simple (KISS)**
   * **facile à comprendre**
   * **lisible**

## Spécifications techniques

* **asynchrone**. Inhérent au protocole série. Il est possible d'ajouter un système d'accusé de réception mais cela nuirait à la simplicité. A étudier au besoin.
* **1 seule commande à la fois**
* **types de messages**
   * **action/ordre**
     * verbe (lisibilité). Eviter les codes et abréviations, les nanosecondes gagnées sont elles si primordiales par rapport à la lisibilité ?
     * avec/sans paramètre
     * avec/sans retour. Le retour doit contenir une référence vers l'identifiant de l'action causale.
   * **log**
* **paramètres**
  * listes possibles mais pas les sous-listes (simplicité)
  * non typés. A l'appelant de savoir quoi envoyer.
* **identification du message** et **horodatage**. Du fait de l'asynchronisme il est utile de pouvoir dater et connaître un message. Par exemple pour qu'un retour soit reconnu comme le retour d'un appel donné.
* **identification des intervenants**. L'émetteur et le receveur. Nécessaire en mode multi-points.
* **pas de gestion d'erreur de communication**. Le protocole sous-jacent s'en occupe (en partie).
* **implémentations (languages)**
  * **C++** pour l'Arduino (ESP/AVR) et le Raspberry.
  * ? **Python** pour le Raspberry

## Etude en cours

### le protocole

* caractère de début de message : ```#```
* caractère de fin de message : ```\n```
* id émetteur (optionnel en UART)
* ids receveurs (optionnel en UART). Le receveur vérifie s'il fait partie des destinataires (rendre ce check désactivable ?).
* date : ```yyyymmddhhMMss```
* id message. Séquence de ```uint8_t``` ou ```uint16_t``` (à définir : occupation mémoire, utilité d'un grand nombre).
* id du message initial
* séparateur primaire : ```|```. Il sépare les champs du message : date, verbe, paramètres...
* séparateur secondaire : ```^```. Il séprare les éléments d'une liste.
* caractère d'échappement : ```\```. A utiliser si une donnée contient un des caractères utilisés dans la syntaxe du protocole.
* convention de nommage sur setter/getter/retour de setter/retour de getter ?

### implémentation

* utilisation d'objets de type ```Stream``` (cf. https://www.arduino.cc/en/Reference/APIStyleGuide et https://github.com/andrewrapp/xbee-arduino). Dans le constructeur : ```(Stream &serial)```. Attention, Stream fait référence aux liaisons série mais aussi à l'ethernet et au wifi, peut-être une point à noter dans la doc.

### ToDo

* algorigrammes de la lecture sur le port série, UART et I²C différents.
* diagramme de classe
* quel logiciel pour ça ? Dia, yEd, Libre Office.

### Exemples

#### Méthode sans paramètre avec retour

Le module 1 demande aux modules 2 et 3 la température :
```
#1|2^3|20180610110526|123||getTemperature\n
```

Le module 2 donne au module 1 la température :
```
#2|1|20180610110526|84|123|temperatureIs|21\n
```

Le module 3 donne au module 1 la température :
```
#3|1|20180610110526|17|123|temperatureIs|21\n
```

*Noter que chaque module possède sa propre séquence d'identifiant de message. Nul besoin de partager une séquence unique. D'ailleurs, cela impliquerait un maître, du moins pour la génération des nombres.*
