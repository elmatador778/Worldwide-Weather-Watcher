# Livrable 2 : Architecture Logicielle du Système
**Projet :** Worldwide Weather Watcher  
**Plateforme :** Seeeduino Lotus (ATmega328P) + Grove Ecosystem  
**Auteurs :** ANDRIANARISATA Tsiky; LALINNE Robin; LANDRIER Quentin  

## 1. Vue d'ensemble
Ce document décrit l'architecture logicielle de la station météo. Le système repose sur une **programmation événementielle non-bloquante** pour garantir la fiabilité de l'acquisition GPS et la réactivité de l'interface utilisateur.

L'architecture est divisée en 4 modules principaux :
1.  **Core (Main Loop) :** Orchestration générale.
2.  **FSM (Finite State Machine) :** Gestion des modes de fonctionnement.
3.  **HAL (Hardware Abstraction Layer) :** Pilotage des capteurs Grove et de la carte SD.
4.  **Data Manager :** Gestion du système de fichiers et de la configuration EEPROM.

## 2. Machine à États (State Machine)
Le cœur du système est une machine à états finis qui respecte les transitions définies dans le cahier des charges.

### Diagramme des États-Transitions
Ce schéma illustre comment le système navigue entre les modes **Standard**, **Éco**, **Maintenance** et **Configuration** en fonction des interactions boutons

```mermaid
stateDiagram-v2
    direction LR

    [*] --> Initialisation

    Initialisation --> Mode_Configuration : Bouton rouge maintenu
    Initialisation --> Mode_Standard : Aucun bouton

    state Mode_Configuration {
        LED jaune fixe
        Paramétrage série
        Timeout 30 min
    }

    Mode_Configuration --> Mode_Standard : Inactivité

    state Mode_Standard {
        LED verte
        Log toutes les 10 min
    }

    state Mode_Economique {
        LED bleue
        Intervalle x2
        GPS 1/2
    }

    state Mode_Maintenance {
        LED orange
        SD désactivée
        Lecture USB live
    }

    Mode_Standard --> Mode_Maintenance : Btn rouge > 5s
    Mode_Standard --> Mode_Economique : Btn vert > 5s
    Mode_Economique --> Mode_Standard : Btn rouge > 5s
    Mode_Maintenance --> Mode_Standard : Btn rouge > 5s
