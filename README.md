# PROJECT APO — Documentation Développeur
> *"Survivre. Comprendre. Choisir."*

Un jeu de survie post-apocalyptique en 3D vue isométrique (style Project Zomboid),
inspiré de **Fallout** et **The Walking Dead**. Développé sous **Godot 4.6**.

---

## 🌍 Lore — Le Monde

**Année 2031. Sainte-Valloire, France.**

Trois ans après **L'Effondrement** — une série de crises simultanées :
- 🦠 Le virus **SORRENTO** *(Systemic Organic Rapid Reanimation & Endocrine Toxin)*
- 🏛️ L'effondrement des gouvernements sous les émeutes et la guerre civile
- 💥 La panne électromagnétique mondiale suite à une attaque sur les satellites

### Types de morts-vivants
|     Type    |      Vitesse     | PV |             Danger              |
|-------------|------------------|----|---------------------------------|
| **Errant**  | Lente (1.2 m/s)  | 60 | Dangereux en groupe             |
| **Coureur** | Rapide (5.5 m/s) | 40 | Très agressif, surtout la nuit  |
| **Hurleur** |     Moyenne      | 80 | Alerte tous les zombies proches |

### Le protagoniste
**Alex Mercer** — Se réveille dans l'hôpital Saint-Croix avec des fragments de mémoire.
Son bracelet dit qu'une infirmière l'a sauvé. Pourquoi était-il là ? Qui a créé SORRENTO ?

---

## 📖 Structure de l'Histoire

```
Acte 1 : Le Réveil
  ├─ Q1 : Le Réveil (hôpital Saint-Croix)
  ├─ Q2 : La Fuite (sortir de la ville)
  └─ Q3 : Un Toit (premier abri)

Acte 2 : Les Survivants
  ├─ Q4 : Trouver une radio
  ├─ Q5 : La Colonie de Millbrook
  └─ Factions, choix moraux, ressources

Acte 3 : La Vérité
  └─ Q6 : Qui a créé SORRENTO ? (corp. Helix Industries)

Acte 4 : Le Choix Final
  └─ Q7 : Détruire le labo / Diffuser le remède / Survivre seul
```

---

## 🗂️ Structure du Projet

```
project-apo/
├── project.godot               ← Config principale (inputs, autoloads)
├── scenes/
│   ├── Main.tscn               ← Menu principal
│   ├── World.tscn              ← Scène de jeu principale
│   ├── Player.tscn             ← Alex Mercer (placeholder capsule verte)
│   ├── Zombie.tscn             ← Zombie (placeholder capsule rouge)
│   └── ui/
│       └── HUD.tscn            ← Interface complète (stats, quêtes, dialogue)
├── scripts/
│   ├── autoload/
│   │   ├── GameManager.gd      ← État global, temps, sauvegarde
│   │   ├── StoryManager.gd     ← Quêtes, dialogues, flags narratifs
│   │   └── AudioManager.gd     ← Gestion audio (placeholder)
│   ├── Player.gd               ← Contrôleur joueur
│   ├── PlayerCamera.gd         ← Caméra isométrique (Q/E = rotation, scroll = zoom)
│   ├── SurvivalStats.gd        ← HP, Faim, Soif, Fatigue, Stress, Infection
│   ├── Inventory.gd            ← Grille 8×6 + hotbar 5 slots
│   ├── InventoryItem.gd        ← Classe item + factory (30+ items prédéfinis)
│   ├── Zombie.gd               ← IA zombie (FSM : Errance → Détection → Poursuite → Attaque)
│   ├── World.gd                ← Cycle jour/nuit, spawn, propagation sonore
│   ├── WorldGenerator.gd       ← Génération par chunks (zones : hôpital, résidentiel...)
│   ├── Hospital.gd             ← Scène hôpital procédurale (Acte 1)
│   ├── Interactable.gd         ← Base interactables + LootContainer
│   └── ui/
│       ├── HUD.gd              ← Interface de jeu
│       └── MainMenu.gd         ← Menu principal
```

---

## 🎮 Contrôles

|     Touche    |         Action         |
|---------------|------------------------|
| `WASD`        | Se déplacer            |
| `Shift`       | Sprint                 |
| `Ctrl`        | Se faufiler (sneak)    |
| `E`           | Interagir              |
| `Clic gauche` | Attaquer               |
| `I`           | Inventaire             |
| `Q / E`       | Rotation caméra (±45°) |
| `Molette`     | Zoom                   |
| `1-5`         | Sélectionner hotbar    |
| `Espace`      | Avancer le dialogue    |

---

## ⚙️ Systèmes de Survie

### Stats du joueur
- **❤️ Santé** — Tombe si blessé ou à 0 faim/soif. Récupère avec soins.
- **🍖 Faim** — Vide en ~3.5h réelles. Sprint × 2.5 la dépense.
- **💧 Soif** — Plus critique que la faim (×1.6 rapide). Sprint × 3.
- **😴 Fatigue** — Réduit la vitesse et augmente le stress si critique.
- **⚡ Stress** — Monte au contact des ennemis. Diminue naturellement.
- **☣ Infection** — Contractée par morsure. Seuls les **antibiotiques** la réduisent.
  Progression : 0% → 100% = mort certaine.

### Modificateurs
- **Sprint** → Faim ×2.5, Soif ×3, Fatigue ×3
- **Sneak** → Faim ×1.2, Fatigue ×1.5, Bruit ×0.1
- **Blessures** → Vitesse –20% par tranche de 2 blessures
- **Fatigue < 20%** → Vitesse –30%
- **Infection > 50%** → Perte de santé continue

---

## 🧟 Intelligence Artificielle

La FSM (machine à états finis) des zombies :

```
IDLE ──(timer)──► WANDER
  ▲                  │ bruit entendu
  │                ALERT ──► (cherche la source)
  │ trop loin        │ joueur vu
  └─────────────── CHASE ──► ATTACK
							   │ joueur mort / trop loin
							   └──► WANDER
```

- **Détection sonore** : Sprint = sphère bruit r=20m, Marche = r=5m, Sneak = r=1m
- **Détection visuelle** : RayCast3D — bloqué par les murs
- **Hurleur** : émet un son global qui alerte tous les zombies dans r=30m

---

## 🗺️ Génération du Monde

- Système de **chunks** : 16×16 tuiles, tuile = 2m → chunk = 32×32m
- **Zones** : Hôpital (départ), Résidentiel, Commercial, Industriel, Parc, Autoroute, Ruines
- Chunks chargés dynamiquement autour du joueur (distance configurable)
- Couleurs placeholder distinctes par type de terrain

---

## 🔌 Intégration des Assets 3D

Remplacer les placeholders dans :

|       Fichier       |  Placeholder  |         À remplacer par         |
|---------------------|---------------|---------------------------------|
| `Player.tscn`       | Capsule verte | Modèle Alex Mercer `.glb` animé |
| `Zombie.tscn`       | Capsule rouge | Modèle zombie `.glb` animé      |
| `Hospital.gd`       | Boîtes grises | Props hospitaliers `.glb`       |
| `WorldGenerator.gd` | Cubes colorés | TileMap 3D ou MeshLibrary       |

---

## 💾 Sauvegarde

Fichier : `user://save_data.tres`
Contenu : position, stats, inventaire, flags histoire, temps de jeu.
Méthodes : `GameManager.save_game()` / `GameManager.load_game()`

---

## 🗺️ Roadmap

- [ ] Implémenter le système de craft
- [ ] Base-building (barricades, camp)
- [ ] PNJ survivants avec dialogues ramifiés
- [ ] Véhicules (voiture, vélo)
- [ ] Remplacer les placeholders par des assets 3D définitifs
- [ ] Système météo (pluie, brouillard)
- [ ] Fin narrative (4 fins différentes selon les flags moraux)
