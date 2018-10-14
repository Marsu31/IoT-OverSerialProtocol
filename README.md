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
* **types de messages**
   * **action**
     * verbe (lisibilité). Eviter les codes et abréviations, les nanosecondes gagnées sont elles si primordiales par rapport à la lisibilité ?
     * avec/sans paramètre
     * avec/sans retour. Le retour doit contenir une référence vers l'identifiant de l'action causale.
   * **log**
* **paramètres**
  * listes possibles mais pas les sous-listes (simplicité)
  * non typés. A l'appelant de savoir quoi envoyer.
* **identification du message** et **horodatage**. Du fait de l'asynchronisme il est utile de pouvoir dater et connaître un message. Par exemple pour qu'un retour soit reconnu comme le retour d'un appel donné.
* **identification des intervenants**. L'émetteur et le receveur. Nécessaire en mode multi-points.
* **implémentations (languages)**
  * **C++** pour l'Arduino (ESP/AVR) et le Raspberry.
  * ? **Python** pour le Raspberry
