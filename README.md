# Consignes données par GEMINI


# 📝 Mémo Chauffage & Rénovation Énergétique — Ludres

Ce document résume la configuration optimale et automatisée de la maison pour l'hiver, combinant un abonnement **EDF Tempo (9 kVA)**, une pompe à chaleur **Daikin Multisplit 4MXM68**, un **chauffage central au gaz** et un **poêle à granulés**.

---

## 1. Réglages Fixes (Physiques)

| Équipement | Emplacement | Réglage Fixe |
| :--- | :--- | :--- |
| **Vannes Thermostatiques Gaz** | Salon (36 m²) | **Position 2** (env. 16°C - 17°C) |
| **Vannes Thermostatiques Gaz** | Étage (Chambres) | **Position 3** (env. 19°C) |
| **Thermostat d'Ambiance Gaz** | Couloir Étage (près SDB) | **19°C** (Radiateur SDB ouvert à 100%) |
| **Voiture Électrique** | Borne de recharge | Bridée à **8A** (~1,8 kW) |
| **Compteur Linky** | Entrée | **9 kVA** (Pic nocturne max estimé à 6,45 kVA) |

---

## 2. Logique de Fonctionnement Selon les Jours Tempo

### 🔵 Jours Bleus (300 j/an) & ⚪ Jours Blancs (43 j/an)
*L'électricité est l'énergie la moins chère.*
* **En Bas (Salon)** : La Daikin FTXM35A gère le chauffage à **19°C / 20°C** (Option *3D Airflow*). En journée, le poêle à granulés prend le relais ; la clim régule sa consommation à zéro d'elle-même (Inverter). Le gaz reste éteint en bas car la pièce ne descend pas sous les 16°C.
* **En Haut (Chambres)** : Les Daikin FTXM20A maintiennent **17°C / 18°C** (*Mode HEAT + Econo + Silence*). 

### 🔴 Jours Rouges (22 j/an)
*L'électricité est prohibitive en Heures Pleines (6h00 - 22h00).*
* **En Heures Pleines (6h - 22h)** : Home Assistant éteint toutes les Daikin. Le poêle à granulés et le chauffage au gaz prennent automatiquement le relais pour maintenir la maison au chaud sans aucune action manuelle.
* **En Heures Creuses (22h - 6h)** : L'électricité redevient rentable. Home Assistant rallume la Daikin de la chambre pour la nuit. La voiture électrique se recharge.

---

## 3. Automatisations Home Assistant (YAML)

> **Prérequis :** Intégrations `EDF Tempo` (via HACS) et `Daikin Onecta` installées.

### 🛑 Extinction automatique à 06h00 (Heures Pleines Rouges)

```yaml
alias: "Tempo : Extinction Daikin - Jours Rouges"
description: "Éteint les climatisations à 6h du matin uniquement les jours rouges Tempo"
trigger:
  - platform: time
    at: "06:00:00"
condition:
  - condition: state
    entity_id: sensor.edf_tempo_couleur_actuelle
    state: "Rouge"
action:
  - service: climate.turn_off
    target:
      entity_id:
        - climate.daikin_salon
        - climate.daikin_chambre
mode: single
```

### 🚀 Rallumage automatique de la chambre à 22h00 (Heures Creuses Rouges)

```yaml
alias: "Tempo : Rallumage Chambre - Nuits Rouges"
description: "Rallume la Daikin de la chambre à 22h en heures creuses lors des jours rouges"
trigger:
  - platform: time
    at: "22:00:00"
condition:
  - condition: state
    entity_id: sensor.edf_tempo_couleur_actuelle
    state: "Rouge"
action:
  - service: climate.set_hvac_mode
    target:
      entity_id: climate.daikin_chambre
    data:
      hvac_mode: heat
  - service: climate.set_temperature
    target:
      entity_id: climate.daikin_chambre
    data:
      temperature: 18
mode: single
```

