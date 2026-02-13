# 🏗️ Livrable 2 : Architecture Logicielle du Système
**Projet :** Worldwide Weather Watcher  
**Plateforme :** Seeeduino Lotus (ATmega328P) + Grove Ecosystem  
**Auteurs :** [Ton Nom / Groupe]  

## 1. Vue d'ensemble
Ce document décrit l'architecture logicielle de la station météo. Le système repose sur une **programmation événementielle non-bloquante** pour garantir la fiabilité de l'acquisition GPS et la réactivité de l'interface utilisateur.

L'architecture est divisée en 4 modules principaux :
1.  **Core (Main Loop) :** Orchestration générale.
2.  [cite_start]**FSM (Finite State Machine) :** Gestion des modes de fonctionnement[cite: 4].
3.  **HAL (Hardware Abstraction Layer) :** Pilotage des capteurs Grove et de la carte SD.
4.  **Data Manager :** Gestion du système de fichiers et de la configuration EEPROM.

---

## 2. Machine à États (State Machine)
Le cœur du système est une machine à états finis qui respecte les transitions définies dans le cahier des charges.

### Diagramme des États-Transitions
[cite_start]Ce schéma illustre comment le système navigue entre les modes **Standard**, **Éco**, **Maintenance** et **Configuration** en fonction des interactions boutons [cite: 4-14].
