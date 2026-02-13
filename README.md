# Worldwide Weather Watcher — Prototype V1

Livrable 2 — Architecture Logicielle et Fonctionnelle

Système embarqué de surveillance météorologique autonome.

- Microcontrôleur : Seeeduino Lotus (ATmega328P)
- Capteurs : Grove + GPS + Carte SD
- Architecture : Firmware événementiel non bloquant
- Date : 13 Février 2026

---

# 1. Philosophie Logicielle

Le firmware repose sur une architecture non bloquante.

Aucune utilisation de `delay()` afin de permettre :

- Lecture GPS continue (NMEA 9600 bauds)
- Détection d'appuis longs (>5s)
- Gestion simultanée des LEDs d’état
- Écriture sécurisée sur carte SD

Le système est piloté par une Machine à États Finis (FSM).

---

# 2. Machine à États (Modes système)

```mermaid
stateDiagram-v2
    direction TB

    [*] --> INITIALISATION

    INITIALISATION : Vérification matériel\nLED orange
    STANDARD : Acquisition normale LED verte
    CONFIGURATION : Paramétrage série LED jaune
    MAINTENANCE : Diagnostic USB LED orange clignotante
    ECO : Économie énergie LED bleue
    ERREUR : Défaut critique  LED rouge

    INITIALISATION --> STANDARD : OK matériel
    INITIALISATION --> ERREUR : Défaut

    STANDARD --> MAINTENANCE : Btn rouge 5s
    STANDARD --> ECO : Btn vert 5s

    ECO --> STANDARD : Btn rouge 5s
    MAINTENANCE --> STANDARD : Retour utilisateur
```

---

# 3. Algorithme Principal

Architecture coopérative temps réel.

```mermaid
flowchart TD

    START([Boot]) --> INIT[Initialisation périphériques]
    INIT --> CHECK{Bouton rouge ?}

    CHECK -- Oui --> CONFIG[Mode Configuration]
    CHECK -- Non --> STANDARD[Mode Standard]

    STANDARD --> LOOP((Boucle principale))
    CONFIG --> LOOP

    LOOP --> GPS[Lecture GPS prioritaire]
    GPS --> LED[Gestion LEDs]
    LED --> BTN[Lecture boutons]

    BTN --> MODE{Changement mode ?}
    MODE -- Oui --> SWITCH[Mise à jour FSM]
    MODE -- Non --> EXEC

    EXEC{Mode actif}

    EXEC -- Standard --> TIMER1{10 min ?}
    TIMER1 -- Oui --> SDWRITE[Écriture SD]

    EXEC -- Eco --> TIMER2{20 min ?}
    TIMER2 -- Oui --> SDWRITE2[Écriture SD réduite]

    EXEC -- Maintenance --> SERIAL[Sortie USB live]

    SDWRITE --> LOOP
    SDWRITE2 --> LOOP
```


# 4.Diagramme d’activité

```mermaid
flowchart TD
  %% =========================
  %% DÉMARRAGE + CHOIX DU MODE
  %% =========================
  A([Début]) --> B[Initialisation matériel]
  B --> C{Bouton rouge au démarrage ?}

  C -- oui --> D[Mode CONFIGURATION]
  D --> D1[LED Jaune]
  D1 --> D2[Interface série UART]
  D2 --> D3[Modifier les paramètres EEPROM]
  D3 --> D4{30 min sans activité ?}
  D4 -- oui --> D5[Retour en mode STANDARD]
  D4 -- non --> D2

  C -- non --> E[Mode STANDARD]
  E --> E1[LED Verte]

  %% Fusion des 2 chemins avant la boucle principale
  D5 --> M((Fusion))
  E1 --> M

  %% =========================
  %% BOUCLE PRINCIPALE
  %% =========================
  M --> F{Station allumée ?}
  F -- non --> Z([Fin d'activité])
  F -- oui --> G[Lire capteurs]
  G --> H[Horodatage RTC]
  H --> I[Vérifier cohérence données]
  I --> J[Enregistrer sur carte SD]
  J --> K[Attendre LOG_INTERVAL]

  %% =========================
  %% ÉVÈNEMENTS / TRANSITIONS
  %% =========================
  K --> V{Vert appuyé 5s ?}
  V -- oui --> ECO[Mode ECONOMIQUE]
  ECO --> ECO1[LED Bleue]
  ECO1 --> ECO2[GPS : 1 mesure sur 2]
  ECO2 --> ECO3[LOG_INTERVAL x2]
  ECO3 --> ECO4[Rouge 5s = STANDARD]
  ECO4 --> RJOIN((Fusion et retour))

  V -- non --> R{Rouge appuyé 5s ?}
  R -- oui --> MAINT[Mode MAINTENANCE]
  MAINT --> MA1[LED Orange]
  MA1 --> MA2[Affiche données série]
  MA2 --> MA3[Pas d'écriture SD]
  MA3 --> MA4[Rouge 5s = Retour mode précédent]
  MA4 --> RJOIN

  R -- non --> ERR{Erreur capteur / RTC / SD ?}
  ERR -- oui --> ERRA[LED clignotante erreur]
  ERRA --> RJOIN
  ERR -- non --> RJOIN

  %% Retour à la boucle
  RJOIN --> F
```

---

# 5. Gestion des Fichiers SD

## Format des logs

```
AAMMJJ_R.LOG
```

Exemple :

```
260213_0.LOG
```

### Rotation automatique

- écriture dans `_0.LOG`
- dépassement taille → renommage `_1.LOG`
- nouveau fichier créé

---

# 6. Diagnostic LED (Codes erreurs)

```mermaid
gantt
    title Séquences LED (visualisation normalisée)
    dateFormat X
    axisFormat %s

    section RTC HS
    Rouge :crit, 0, 1
    Bleu  :active, 1, 1

    section GPS HS
    Rouge :crit, 0, 1
    Jaune :active, 1, 1

    section Capteur HS
    Rouge :crit, 0, 1
    Vert  :active, 1, 1

    section SD Pleine
    Rouge :crit, 0, 1
    Blanc :active, 1, 1

    section Accès SD
    Rouge court :crit, 0, 1
    Blanc long  :active, 1, 2
```

---

# 7. Paramètres Configurables

| Paramètre | Valeur |
|---|---|
| MIN_TEMP_AIR | -10°C |
| MAX_TEMP_AIR | 60°C |
| PRESSURE_MIN | 850 hPa |
| PRESSURE_MAX | 1080 hPa |
| LOG_INTERVAL | 10 min |

---

# 8. Architecture Firmware

```mermaid
flowchart LR

    APP[Application FSM]
    DRIVERS[Drivers Capteurs]
    HAL[HAL Microcontrôleur]
    HW[Hardware ATmega328P]

    APP --> DRIVERS
    DRIVERS --> HAL
    HAL --> HW
```

---

# 9. Objectifs Techniques

- Firmware non bloquant
- Acquisition multi-capteurs
- Gestion énergétique
- Diagnostic terrain
- Intégrité des données

---

# Licence

Projet éducatif — utilisation libre pour apprentissage.
