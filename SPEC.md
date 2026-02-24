# ChessCoach — Cahier des Charges

**Coach d'échecs personnalisé via Telegram**

| | |
|---|---|
| **Projet** | ChessCoach (nom de travail) |
| **Fondateur** | Clément |
| **Version** | 1.0 |
| **Date** | 24 février 2026 |
| **Statut** | Draft — prêt pour review technique |

---

## Table des matières

1. [Vision & Problème](#1-vision--problème)
2. [Concept du produit](#2-concept-du-produit)
3. [Utilisateur cible](#3-utilisateur-cible)
4. [Fonctionnalités détaillées](#4-fonctionnalités-détaillées)
5. [Architecture technique](#5-architecture-technique)
6. [Modèle économique](#6-modèle-économique)
7. [Roadmap](#7-roadmap)
8. [Analyse concurrentielle](#8-analyse-concurrentielle)
9. [Risques & Mitigations](#9-risques--mitigations)
10. [KPIs de lancement](#10-kpis-de-lancement)
11. [Annexes](#11-annexes)

---

## 1. Vision & Problème

### Le constat

Les joueurs d'échecs amateurs (800–1800 Elo) font **les mêmes erreurs d'ouverture en boucle**. Ils perdent des parties sur des schémas qu'ils ont déjà vus, sans jamais corriger le pattern. Les outils existants (Chess.com Game Review, Lichess analysis) proposent une analyse partie par partie, mais aucun ne fait le travail d'un vrai coach :

- **Identifier les erreurs récurrentes** à travers des dizaines de parties
- **Alerter le joueur** quand il refait la même erreur
- **Suivre la progression** dans le temps

### La vision

**ChessCoach** est un coach d'échecs personnel accessible via Telegram. Il se connecte au compte Chess.com du joueur, analyse automatiquement ses parties, détecte les patterns d'erreurs récurrents, et envoie des conseils personnalisés avec des positions interactives — comme un coach qui regarde par-dessus ton épaule après chaque partie.

### Pourquoi Telegram ?

- **0 friction** : pas d'app à télécharger, pas de compte à créer
- **Notifications natives** : le coach vient à toi, pas l'inverse
- **Mini Apps** : échiquier interactif directement dans la conversation
- **2,3 milliards d'utilisateurs** actifs mensuels sur les plateformes de messagerie — Telegram est le choix naturel pour une audience tech-savvy et internationale

---

## 2. Concept du produit

ChessCoach est un **bot Telegram** couplé à une **Mini App** (WebApp intégrée) qui :

1. Se connecte au compte Chess.com de l'utilisateur
2. Analyse automatiquement ses parties (Rapid + Blitz)
3. Identifie les **erreurs récurrentes** d'ouverture (celles qui apparaissent 2+ fois)
4. Envoie des **notifications personnalisées** avec conseil + position interactive
5. Génère un **rapport hebdomadaire** de progression
6. Propose des **drills interactifs** pour corriger les erreurs

**Mot-clé : PATTERNS.** Pas l'analyse d'une partie, mais l'analyse de TES parties sur le temps.

---

## 3. Utilisateur cible

### Persona principal

| | |
|---|---|
| **Nom** | Alex, 28 ans |
| **Elo** | ~1200 Rapid sur Chess.com |
| **Profil** | Joue 3-5 parties par jour, principalement en Blitz/Rapid |
| **Frustration** | « Je perds toujours de la même manière en Italienne mais je ne sais pas quoi faire différemment » |
| **Budget** | Prêt à payer 5-10€/mois pour un outil qui l'aide vraiment à progresser |
| **Habitudes** | Utilise Telegram quotidiennement, consulte son téléphone entre deux parties |

### Segment cible

- **Elo** : 800 – 1800 (joueurs amateurs à intermédiaires)
- **Cadences** : Rapid (10+0, 15+10) et Blitz (3+0, 5+0)
- **Plateformes** : Chess.com (prioritaire), Lichess (Phase 4)
- **Géographie** : Francophone d'abord, puis international (EN)
- **Volume estimé** : ~10M de joueurs actifs mensuels sur Chess.com dans cette tranche

---

## 4. Fonctionnalités détaillées

### 4.1 Onboarding

**Objectif** : connecter le joueur en < 2 minutes et lui montrer de la valeur immédiatement.

**Flux utilisateur :**

```
Utilisateur → /start
Bot → "Bienvenue ! Quel est ton pseudo Chess.com ?"
Utilisateur → "ClementChess42"
Bot → "Je récupère tes parties... ⏳"
       (récupération de 100-300 dernières parties Rapid + Blitz)
Bot → "Analyse terminée ! Voici ton profil :"
       → Rapport initial avec erreurs récurrentes
       → Bouton [Voir mes erreurs] → Mini App
```

**Détails techniques :**

| Élément | Spécification |
|---|---|
| Parties récupérées | 100–300 dernières (Rapid + Blitz) |
| Temps d'analyse | < 60 secondes pour le rapport initial |
| Validation pseudo | Vérification via Chess.com API que le compte existe |
| Profil généré | Ouvertures favorites (Blancs/Noirs), taux de victoire par ouverture, tendances (agressif/positionnel), points forts/faibles |
| Erreurs détectées | Uniquement celles apparaissant **2 fois ou plus** (pattern confirmé) |

**Commandes bot :**

| Commande | Description |
|---|---|
| `/start` | Lancement de l'onboarding |
| `/profile` | Afficher le profil joueur |
| `/errors` | Liste des erreurs récurrentes |
| `/report` | Dernier rapport hebdomadaire |
| `/settings` | Langue, fréquence de notifications, cadence préférée |
| `/help` | Aide et commandes disponibles |

---

### 4.2 Analyse automatique (veille)

**Objectif** : détecter automatiquement les nouvelles parties et alerter si une erreur connue est répétée.

**Fonctionnement :**

1. **Polling régulier** : le backend vérifie les nouvelles parties toutes les 15-30 minutes
2. **Analyse de l'ouverture** : les 15 premiers coups sont analysés
3. **Comparaison** : le coup joué est comparé au meilleur coup (Stockfish) et à la théorie
4. **Détection de récurrence** : si le coup correspond à une erreur déjà cataloguée → alerte

**Notification type :**

```
🔴 Erreur récurrente détectée !

Dans ta partie contre @opponent (Blitz 5+0), tu as joué 
5...Fd7?! dans la Défense Française — c'est la 3ème fois 
ce mois !

Le bon coup est 5...c5 pour contester le centre.

[📋 Voir la position] → Mini App
[🧠 Drill cette ligne] → Mode quiz
```

**Règles de notification :**

- Maximum **3 notifications par jour** (éviter le spam)
- Prioriser les erreurs les plus fréquentes et les plus coûteuses (en centipawns)
- Regrouper si plusieurs erreurs dans la même partie
- Respecter les horaires de l'utilisateur (pas de notif entre 23h et 8h, configurable)

---

### 4.3 Telegram Mini App (échiquier interactif)

**Objectif** : afficher les positions de manière interactive directement dans Telegram.

**Écran principal — Vue erreur :**

```
┌─────────────────────────────┐
│        ♜ ♞ ♝ ♛ ♚ ♝ ♞ ♜    │
│        ♟ ♟ ♟ ♟ ♟ ♟ ♟ ♟    │
│                              │
│           (échiquier)        │
│                              │
│        ♙ ♙ ♙ ♙ ♙ ♙ ♙ ♙    │
│        ♖ ♘ ♗ ♕ ♔ ♗ ♘ ♖    │
├─────────────────────────────┤
│  🔴 Tu as joué : Fd7?!      │
│  🟢 Meilleur coup : c5!     │
│                              │
│  "5...c5 conteste le centre  │
│   et empêche d4-d5. Fd7 est │
│   passif et ne développe     │
│   pas ta position."          │
├─────────────────────────────┤
│  [◀ Prev]  [▶ Next]         │
│  [🧠 Mode Quiz]              │
└─────────────────────────────┘
```

**Fonctionnalités de la Mini App :**

| Fonctionnalité | Description |
|---|---|
| **Échiquier interactif** | Rendu via chessground (Lichess, open source) |
| **Flèches colorées** | Rouge = coup joué (mauvais), Vert = meilleur coup |
| **Navigation** | Coup par coup, avant/arrière, retour à la position clé |
| **Explication textuelle** | En français ou anglais (selon préférence utilisateur) |
| **Mode Quiz** | L'échiquier montre la position, l'utilisateur doit cliquer le bon coup avant de voir la réponse |
| **Animations** | Déplacement fluide des pièces, apparition progressive des flèches |
| **Responsive** | Optimisé pour écran mobile (priorité Telegram mobile) |

**Mode Quiz — Flux :**

1. Position affichée sans flèches
2. Question : « Quel est le meilleur coup pour les Noirs ? »
3. L'utilisateur clique sur une pièce et une case
4. ✅ Correct → animation verte + explication + XP gagné
5. ❌ Incorrect → flèche rouge sur le coup joué, flèche verte sur le bon coup + explication

---

### 4.4 Rapport hebdomadaire

**Objectif** : donner une vue d'ensemble de la semaine et mesurer la progression.

**Envoi** : chaque lundi à 9h (configurable).

**Contenu du rapport :**

```
📊 Rapport hebdomadaire — Semaine du 17 au 23 février

🎮 Parties jouées : 23 (14 Rapid, 9 Blitz)
📈 Elo Rapid : 1387 → 1412 (+25)
📈 Elo Blitz : 1198 → 1205 (+7)

🔴 Erreurs récurrentes cette semaine :
  1. Italienne — 5.d3?! au lieu de 5.c3 (3 fois)
  2. Sicilienne — 6...e5?! au lieu de 6...a6 (2 fois)

✅ Erreurs corrigées :
  • Française — 5...c5 joué correctement 2/2 fois ! 🎉

📊 Score de progression : 67%
   (2 erreurs corrigées sur 3 identifiées la semaine précédente)

🏆 Tu es sur une série de 3 jours sans erreur récurrente !

[📋 Détails complets] → Mini App
[🧠 Drill mes erreurs] → Mode quiz groupé
```

---

### 4.5 Répertoire d'ouvertures personnalisé

**Objectif** : construire un répertoire basé sur ce que le joueur joue DÉJÀ (pas imposer des ouvertures théoriques).

**Fonctionnement :**

1. **Détection** : identifier les 3-5 ouvertures les plus jouées (Blancs et Noirs)
2. **Cartographie** : pour chaque ouverture, lister les lignes principales que le joueur devrait connaître
3. **Gaps** : identifier les variantes où le joueur dévie de la théorie
4. **Drill** : exercice interactif où le bot joue les coups adverses et l'utilisateur doit jouer les bons coups

**Drill interactif — Flux :**

```
Bot → "On travaille la Défense Française, variante d'avance."
Bot → "1.e4" (affiché sur l'échiquier)
Utilisateur joue → 1...e6 ✅
Bot → "2.d4"
Utilisateur joue → 2...d5 ✅
Bot → "3.e5"
Utilisateur joue → 3...c5 ✅ "Parfait ! Tu contestes le centre."
...
Utilisateur joue un mauvais coup → ❌ "Le coup principal ici est Nc6. 
  Ça développe le cavalier vers le centre et prépare Qb6."
```

**Données affichées par ouverture :**

- Nom + code ECO
- Nombre de parties jouées
- Win rate
- Erreurs récurrentes dans cette ouverture
- Lignes à connaître (arbre de variantes)
- Score de maîtrise (% de coups théoriques joués correctement)

---

## 5. Architecture technique

### 5.1 Vue d'ensemble

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│  Utilisateur │────▶│  Telegram    │────▶│  Bot Telegram    │
│  (Mobile)    │◀────│  (Bot API)   │◀────│  (grammY/Node)   │
└──────────────┘     └──────────────┘     └────────┬─────────┘
       │                                           │
       │ Mini App                                  │ API calls
       ▼                                           ▼
┌──────────────┐                          ┌──────────────────┐
│  Mini App    │─────── API REST ────────▶│  Backend API     │
│  (React +    │◀────────────────────────│  (FastAPI/Node)  │
│  chessground)│                          └────────┬─────────┘
└──────────────┘                                   │
                                                   ├──▶ Chess.com API
                                                   ├──▶ Stockfish (WASM)
                                                   ├──▶ PostgreSQL
                                                   └──▶ Redis (cache)
```

### 5.2 Frontend — Telegram Mini App

| Élément | Choix technologique | Justification |
|---|---|---|
| **Framework** | React (Vite) | Écosystème riche, chessground bien intégré |
| **Échiquier** | chessground | Moteur de Lichess, open source, performant, maintenu |
| **Logique échecs** | chess.js | Validation des coups, parsing PGN, détection d'échec/mat |
| **State management** | Zustand ou React Context | Léger, suffisant pour l'app |
| **Styling** | Tailwind CSS | Rapidité de développement, responsive natif |
| **Déploiement** | Cloudflare Pages ou Vercel | CDN global, HTTPS automatique, gratuit |

**Contraintes Telegram Mini App :**

- Viewport variable (selon l'appareil)
- Pas de stockage local fiable → tout sur le backend
- Thème dynamique (s'adapte au thème Telegram de l'utilisateur)
- SDK Telegram Web App pour les interactions natives (bouton retour, haptic feedback, etc.)

### 5.3 Backend — API REST

| Élément | Choix technologique | Justification |
|---|---|---|
| **Framework** | Python / FastAPI | Rapide à développer, async natif, typage fort |
| **ORM** | SQLAlchemy + Alembic | Migrations, relations, requêtes complexes |
| **Base de données** | PostgreSQL (Supabase) | Robuste, JSONB pour les données de parties, managé |
| **Cache** | Redis (Upstash) | Positions déjà analysées, rate limiting, sessions |
| **Task queue** | Celery + Redis (ou ARQ) | Analyse en arrière-plan, polling Chess.com |
| **Auth** | Telegram Web App initData | Authentification native via Telegram |

**Endpoints principaux :**

```
POST   /api/users/onboard          # Onboarding (pseudo Chess.com)
GET    /api/users/:id/profile       # Profil joueur
GET    /api/users/:id/errors        # Erreurs récurrentes
GET    /api/users/:id/report        # Rapport hebdomadaire
GET    /api/games/:id/analysis      # Analyse d'une partie
GET    /api/openings/:id/repertoire # Répertoire d'ouvertures
POST   /api/drill/start             # Démarrer un drill
POST   /api/drill/move              # Soumettre un coup dans un drill
GET    /api/positions/:fen          # Position + évaluation + meilleur coup
```

### 5.4 Bot Telegram

| Élément | Choix technologique | Justification |
|---|---|---|
| **Librairie** | grammY (TypeScript/Node.js) | Moderne, bien typé, middleware puissant |
| **Conversations** | grammY conversations plugin | Gestion de flux multi-étapes (onboarding) |
| **Inline buttons** | Telegram Bot API | Boutons pour ouvrir la Mini App, naviguer |
| **Notifications** | Scheduled tasks (cron) | Rapports hebdo, alertes erreurs |
| **Rate limiting** | Telegram API limits | 30 messages/seconde global, 1 message/seconde par chat |

### 5.5 Moteur d'analyse d'échecs

**Pipeline d'analyse :**

```
Partie (PGN) 
  → Parsing (chess.js / python-chess)
  → Extraction des 15 premiers coups
  → Pour chaque coup :
      → Évaluation Stockfish (profondeur 18-20)
      → Comparaison avec le meilleur coup
      → Si delta > 50cp → marqué comme erreur
  → Clustering par ouverture (code ECO)
  → Comparaison avec erreurs existantes du joueur
  → Si match → erreur récurrente flaggée
```

**Seuils de détection :**

| Catégorie | Perte en centipawns | Icône |
|---|---|---|
| Imprécision | 50–100 cp | ⚠️ |
| Erreur | 100–300 cp | 🔴 |
| Gaffe (blunder) | > 300 cp | 💀 |

**Clustering des erreurs :**

Les erreurs sont regroupées par :
- **Ouverture** (code ECO + nom)
- **Position** (FEN des 4 premières rangées, pour détecter des transpositions)
- **Pattern** (ex : « fou développé passivement », « centre non contesté »)

Une erreur est considérée **récurrente** si elle apparaît **2 fois ou plus** dans les 30 derniers jours.

### 5.6 Intégration Chess.com API

**Endpoints utilisés :**

| Endpoint | Usage |
|---|---|
| `GET /pub/player/{username}` | Vérification du compte |
| `GET /pub/player/{username}/stats` | Elo actuel, stats |
| `GET /pub/player/{username}/games/{YYYY}/{MM}` | Parties du mois (PGN inclus) |
| `GET /pub/player/{username}/games/archives` | Liste des archives disponibles |

**Contraintes :**

- **Rate limit** : non documenté officiellement, ~300 req/min observé
- **Pas de webhooks** : polling nécessaire
- **Données** : PGN complet inclus dans la réponse (pas besoin d'appel supplémentaire)

**Stratégie de cache :**

- Parties déjà récupérées → stockées en BDD, jamais re-fetchées
- Évaluations Stockfish par FEN → cache Redis (TTL 30 jours)
- Profil joueur → refresh toutes les 24h

### 5.7 Hébergement & Infrastructure

| Composant | Service | Coût estimé |
|---|---|---|
| Backend API | Railway ou Hetzner VPS (CX22) | 5–10€/mois |
| Bot Telegram | Même serveur que le backend | inclus |
| PostgreSQL | Supabase (Free → Pro) | 0–25€/mois |
| Redis | Upstash (Free → Pay-as-you-go) | 0–10€/mois |
| Mini App (static) | Cloudflare Pages | Gratuit |
| Stockfish | WASM côté serveur (même VPS) | inclus |
| CI/CD | GitHub Actions | Gratuit |
| Monitoring | Sentry (erreurs) + UptimeRobot | Gratuit |

**Coût total estimé au démarrage : 5–20€/mois**

---

## 6. Modèle économique

### 6.1 Tiers

| | **Free** | **Premium** | **Pro** |
|---|---|---|---|
| **Prix** | 0€ | 7€/mois | 15€/mois |
| Parties analysées | 50 dernières | Illimitées | Illimitées |
| Rapport initial | ✅ Basique | ✅ Détaillé | ✅ Détaillé |
| Erreurs détaillées | 1/semaine | Illimitées | Illimitées |
| Mini App interactive | Consultation seule | ✅ Complète | ✅ Complète |
| Mode Quiz | ❌ | ✅ | ✅ |
| Rapport hebdomadaire | ❌ | ✅ | ✅ |
| Répertoire d'ouvertures | ❌ | ✅ | ✅ |
| Drills interactifs | ❌ | ✅ | ✅ |
| Notifications en temps réel | ❌ | ❌ | ✅ |
| Support Lichess | ❌ | ❌ | ✅ |
| Analyse milieu/finale | ❌ | ❌ | ✅ |

### 6.2 Paiement

- **Processeur** : Stripe (Checkout + Customer Portal)
- **Telegram Stars** : envisager comme canal de paiement alternatif (commission ~30%)
- **Essai gratuit** : 7 jours Premium à l'inscription
- **Annulation** : en un clic via le bot (`/subscription`)

### 6.3 Métriques business

| Métrique | Cible M+3 | Cible M+6 | Cible M+12 |
|---|---|---|---|
| Utilisateurs inscrits | 500 | 2 000 | 10 000 |
| Utilisateurs actifs (WAU) | 150 | 600 | 3 000 |
| Conversion Premium | 8% | 10% | 12% |
| MRR | 840€ | 4 200€ | 25 200€ |
| Churn mensuel | < 15% | < 12% | < 10% |

---

## 7. Roadmap

### Phase 1 — MVP (4-6 semaines)

**Objectif** : prouver la valeur → un joueur donne son pseudo, reçoit ses erreurs récurrentes.

| Tâche | Détail | Priorité |
|---|---|---|
| Bot Telegram — setup | grammY, commandes de base, onboarding | 🔴 Critique |
| Intégration Chess.com API | Récupération parties, parsing PGN | 🔴 Critique |
| Moteur d'analyse v1 | Basé sur la théorie des ouvertures (base ECO), sans Stockfish | 🔴 Critique |
| Détection erreurs récurrentes | Clustering par ouverture + position | 🔴 Critique |
| Rapport texte Telegram | Envoi du rapport dans le chat | 🟡 Important |
| Mini App v1 | Échiquier statique (chessground) + explication | 🟡 Important |
| Base de données | Schema PostgreSQL, modèles utilisateur/parties/erreurs | 🔴 Critique |

**Livrable** : bot fonctionnel, un utilisateur peut s'inscrire et recevoir un rapport d'erreurs.

### Phase 2 — Interactivité (4-6 semaines)

| Tâche | Détail | Priorité |
|---|---|---|
| Mini App v2 | Flèches, navigation, animations, mode quiz | 🔴 Critique |
| Intégration Stockfish | Évaluation précise par position (WASM serveur) | 🔴 Critique |
| Veille automatique | Polling Chess.com, détection nouvelles parties | 🔴 Critique |
| Notifications personnalisées | Alerte quand erreur récurrente re-détectée | 🟡 Important |
| Profil joueur enrichi | Stats détaillées, graphiques de progression | 🟢 Nice-to-have |

**Livrable** : expérience interactive complète, le bot surveille et alerte proactivement.

### Phase 3 — Engagement (4-6 semaines)

| Tâche | Détail | Priorité |
|---|---|---|
| Rapport hebdomadaire | Résumé automatique chaque lundi | 🔴 Critique |
| Mode Quiz/Drill | Exercices interactifs basés sur les erreurs | 🔴 Critique |
| Répertoire d'ouvertures | Arbre personnalisé + drills | 🟡 Important |
| Gamification | Streaks, XP, badges, score de maîtrise | 🟡 Important |
| Graphiques de progression | Courbe Elo, % erreurs corrigées | 🟢 Nice-to-have |

**Livrable** : boucle d'engagement complète, les joueurs reviennent chaque semaine.

### Phase 4 — Monétisation (4 semaines)

| Tâche | Détail | Priorité |
|---|---|---|
| Intégration Stripe | Paiement, abonnements, portail client | 🔴 Critique |
| Tiers Free/Premium/Pro | Gating des fonctionnalités par tier | 🔴 Critique |
| Landing page | Site marketing, SEO, conversion | 🟡 Important |
| Support Lichess | API Lichess (plus permissive que Chess.com) | 🟡 Important |
| Analytics | Mixpanel ou PostHog pour le suivi produit | 🟢 Nice-to-have |

**Livrable** : produit monétisable avec page de conversion.

---

## 8. Analyse concurrentielle

| Concurrent | Forces | Faiblesses | Positionnement |
|---|---|---|---|
| **Chess.com Game Review** | Intégré, puissant, large base | Payant (Diamond), analyse one-shot, pas de suivi des patterns | Outil d'analyse, pas de coaching |
| **Lichess Analysis** | Gratuit, Stockfish, communauté | Pas de personnalisation, pas de suivi temporel | Outil open source |
| **Chessable / ChessMood** | Contenu de qualité, MoveTrainer | Cours génériques, pas basé sur TES parties | Plateforme de cours |
| **DecodeChess** | IA explicative, langage naturel | Pas de suivi dans le temps, pas de notifications | Analyse IA one-shot |
| **ChessCoach (nous)** | Personnel, récurrent, dans Telegram, patterns | Nouveau, pas de marque, scope limité à l'ouverture (v1) | **Coach personnel adaptatif** |

### Différenciation clé

1. **Personnalisation** : analyse basée sur TES parties, TES erreurs, TES ouvertures
2. **Récurrence** : suivi dans le temps, détection de patterns, pas juste une analyse
3. **Accessibilité** : dans Telegram, 0 friction, notifications push
4. **Focus patterns** : ce qui manque à TOUS les concurrents — voir les erreurs à travers 100 parties

---

## 9. Risques & Mitigations

| Risque | Probabilité | Impact | Mitigation |
|---|---|---|---|
| **Chess.com API rate-limitée ou modifiée** | Moyenne | Élevé | Cache agressif, analyse en batch, support Lichess en backup |
| **Stockfish coûteux en compute** | Moyenne | Moyen | WASM côté client quand possible, profondeur limitée (18), cache des évaluations par FEN |
| **Telegram Mini App — limitations UX** | Faible | Moyen | Design mobile-first, tests intensifs sur différents appareils, fallback texte |
| **Faible rétention** | Moyenne | Élevé | Gamification (streaks, XP, badges), challenges hebdo, rapport de progression motivant |
| **Marché de niche trop petit** | Faible | Élevé | Élargir au milieu de partie et aux finales (v2), multi-plateforme |
| **Qualité de l'analyse insuffisante** | Moyenne | Élevé | Stockfish comme source de vérité, feedback utilisateur, itération rapide |
| **Concurrence de Chess.com** | Faible | Élevé | Se différencier sur le suivi dans le temps et l'UX Telegram — Chess.com n'ira pas sur Telegram |

---

## 10. KPIs de lancement

### Objectifs à 30 jours post-lancement beta

| KPI | Objectif | Méthode de mesure |
|---|---|---|
| Utilisateurs beta inscrits | 100 | Compteur BDD |
| Taux d'onboarding complet | > 80% | Funnel : /start → pseudo donné → rapport reçu |
| Utilisateurs actifs semaine 1 | 60% des inscrits | Au moins 1 interaction après onboarding |
| Rétention semaine 4 | > 30% | Utilisateurs actifs S4 / inscrits S1 |
| Taux de conversion Premium | 10% | Abonnés payants / inscrits |
| NPS | > 40 | Survey in-app (après 2 semaines) |
| Erreurs corrigées | > 20% des erreurs identifiées | Erreur non répétée après notification |

### Canaux d'acquisition beta

- Reddit : r/chess, r/chessbeginners
- Discord : serveurs d'échecs francophones
- Twitter/X : communauté échecs
- Forums Chess.com
- Bouche-à-oreille (Telegram = viralité native via partage)

---

## 11. Annexes

### A. Schéma de base de données (simplifié)

```sql
-- Utilisateurs
CREATE TABLE users (
    id              SERIAL PRIMARY KEY,
    telegram_id     BIGINT UNIQUE NOT NULL,
    chess_com_user  VARCHAR(50) NOT NULL,
    language        VARCHAR(5) DEFAULT 'fr',
    tier            VARCHAR(10) DEFAULT 'free',  -- free, premium, pro
    elo_rapid       INT,
    elo_blitz       INT,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    last_sync       TIMESTAMPTZ
);

-- Parties analysées
CREATE TABLE games (
    id              SERIAL PRIMARY KEY,
    user_id         INT REFERENCES users(id),
    chess_com_id    VARCHAR(100) UNIQUE,
    pgn             TEXT NOT NULL,
    time_class      VARCHAR(10),  -- rapid, blitz
    result          VARCHAR(10),  -- win, loss, draw
    user_color      VARCHAR(5),   -- white, black
    eco_code        VARCHAR(5),
    opening_name    VARCHAR(200),
    played_at       TIMESTAMPTZ,
    analyzed        BOOLEAN DEFAULT FALSE
);

-- Erreurs détectées
CREATE TABLE errors (
    id              SERIAL PRIMARY KEY,
    user_id         INT REFERENCES users(id),
    game_id         INT REFERENCES games(id),
    move_number     INT,
    fen             VARCHAR(100),
    move_played     VARCHAR(10),
    best_move       VARCHAR(10),
    eval_loss_cp    INT,           -- centipawns perdus
    category        VARCHAR(20),   -- inaccuracy, mistake, blunder
    opening_eco     VARCHAR(5),
    explanation     TEXT,
    detected_at     TIMESTAMPTZ DEFAULT NOW()
);

-- Patterns d'erreurs récurrentes
CREATE TABLE error_patterns (
    id              SERIAL PRIMARY KEY,
    user_id         INT REFERENCES users(id),
    pattern_fen     VARCHAR(100),  -- FEN simplifié de la position
    opening_eco     VARCHAR(5),
    move_played     VARCHAR(10),
    best_move       VARCHAR(10),
    occurrence_count INT DEFAULT 1,
    last_seen       TIMESTAMPTZ,
    corrected_count INT DEFAULT 0, -- fois où le bon coup a été joué
    status          VARCHAR(20) DEFAULT 'active'  -- active, improving, corrected
);

-- Progression
CREATE TABLE weekly_reports (
    id              SERIAL PRIMARY KEY,
    user_id         INT REFERENCES users(id),
    week_start      DATE,
    games_played    INT,
    elo_rapid_start INT,
    elo_rapid_end   INT,
    elo_blitz_start INT,
    elo_blitz_end   INT,
    errors_repeated INT,
    errors_corrected INT,
    progress_score  FLOAT,        -- % d'amélioration
    report_sent     BOOLEAN DEFAULT FALSE
);
```

### B. Exemple de PGN Chess.com (pour référence)

```pgn
[Event "Live Chess"]
[Site "Chess.com"]
[Date "2026.02.20"]
[Round "-"]
[White "ClementChess42"]
[Black "Opponent123"]
[Result "0-1"]
[ECO "C00"]
[WhiteElo "1387"]
[BlackElo "1425"]
[TimeControl "600"]
[Termination "Opponent123 won by resignation"]

1. e4 e6 2. d4 d5 3. e5 c5 4. c3 Nc6 5. Nf3 Bd7 6. Be2 Nge7 
7. O-O Ng6 8. dxc5 Bxc5 9. b4 Be7 10. Bf4 O-O ...
```

### C. Références techniques

| Ressource | URL |
|---|---|
| Chess.com API | https://www.chess.com/news/view/published-data-api |
| chessground | https://github.com/lichess-org/chessground |
| chess.js | https://github.com/jhlywa/chess.js |
| grammY | https://grammy.dev |
| Telegram Mini Apps | https://core.telegram.org/bots/webapps |
| Stockfish WASM | https://github.com/nicfab/stockfish.wasm |
| FastAPI | https://fastapi.tiangolo.com |
| Stripe Billing | https://stripe.com/billing |

---

*Document rédigé le 24 février 2026. Version 1.0 — à itérer avec l'équipe technique.*
