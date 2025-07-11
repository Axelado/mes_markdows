
# Dossier de Spécifications

## 1. Introduction

### 1.1 Contexte

Ce projet a pour objectif de développer un bras robotique à 4 degrés de liberté (DOF) muni d'un gripper, entièrement imprimé en 3D, et contrôlé par une carte ESP32. Il est destiné à réaliser des tâches de type pick & place, commandé via une interface web.

### 1.2 Objectifs

-   Conception mécanique d'un bras à 4 DOF + gripper.
    
-   Contrôle des moteurs via ESP32.
    
-   Interface web pour le pilotage.
    
-   Enregistrement et exécution de séquences de mouvements.
    

### 1.3 Portée

Le système sera utilisé à des fins pédagogiques ou de prototypage, en environnement non industriel.

## 2. Description générale

### 2.1 Architecture générale

Le bras sera composé de segments imprimés en 3D, articulés par des servomoteurs SG90, contrôlés par une carte ESP32 via PWM. L'utilisateur interagit avec une page web pour piloter individuellement les moteurs ou rejouer des séquences enregistrées.



### 2.2 Diagramme Bloc
 ![enter image description here](https://raw.githubusercontent.com/Axelado/mes_markdows/12affbb0eee149e201e2eceec5743818167214a5/Bras%20robotique%20diagramme%20bloc.drawio.svg)


## 3. Spécifications fonctionnelles

### 3.1 Commande manuelle

-   L'utilisateur peut contrôler chaque moteur via un slider (0-180°).
    
-   L'action sur le slider déclenche l'envoi d'une requête HTTP à l'ESP32.
    

### 3.2 Enregistrement de positions

-   Jusqu'à 5 positions peuvent être enregistrées.
    
-   Chaque position contient les angles des 5 servos.
    
    

### 3.3 Rejeu de séquence

-   Les positions enregistrées peuvent être rejouées en séquence.
    
-   Un délai fixe est appliqué entre les positions.
    

### 3.4 Interface web

-   Accessible via le réseau Wi-Fi local de l'ESP32.
    
-   Comporte sliders, boutons de sauvegarde, de rejouement et de calibration.
    

### 3.5 Calibration

-   Une position "Home" est définie et peut être appelée au démarrage ou manuellement.
    

## 4. Spécifications non fonctionnelles

### 4.1 Performance

-   Latence d’exécution d’une commande < 500 ms.
    
-   Temps de démarrage du serveur < 5 s.
    

### 4.2 Fiabilité

-   Sécurité mécanique : angles limités pour éviter les blocages.
        

### 4.3 Portabilité

-   Le firmware doit être compatible ESP32 DevKit V1.
-	Le serveur web doit être accessible sur depuis un ordinateur, un smartphone ou une tablette
    

### 4.4 Maintenance

-   Code bien documenté.
    
-   Composants remplaçables facilement.
    

## 5. Contraintes
    
-   Servos SG90 limités à un couple faible : bras léger.
    
-   Interface web accessible uniquement en local.
    

## 6. Livrables

-   Fichiers STL de la pièces de la structure
    
-   Code source ESP32 (Arduino)
    
-   Interface Web (HTML/JS/CSS)
    
-   Documentation d’utilisation
    
-   Vidéo de démonstration
    

## 7. Validation

-   Contrôle indépendant des 5 servos via interface

-   Temps de réponse sliders → mouvement < 500ms
    
-   Enregistrement de 5 positions

-   Temps de commutation entre positions enregistrées < 2s
    
-   Exécution fluide de la séquence pick & place

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTUxNDcyOTQxLC0xMTA1MjI3MDg3LC0yNj
M0OTA4NDMsNTExODM1NTI5LC0xNjE1MzM3NTNdfQ==
-->