---
title: "Artifact MMO : manette et clavier/souris, de l'idée brute au projet finalisé en vibe coding"
published: 2026-04-21
description: Retour d'expérience sur mon projet Artifacts Gamepad Controller, entre implémentation manuelle et finalisation en vibe coding.
image: "./game.png"
tags: [Artifact MMO, Gamepad, React, Next.js, Electron, API]
category: Projects
draft: false
---

Artifact MMO est un MMORPG sandbox un peu à part: le jeu a été pensé pour les développeurs, avec un gameplay pilotable via API HTTP.
Concrètement, chaque action de personnage correspond à un endpoint.  

C'est précisément ce qui m'a donné l'idée de ce projet: construire une interface "console-like" pour piloter mon personnage avec une manette et clavier/souris, en branchant des inputs gamepad sur les actions API du jeu.

- Projet: [Waregalias/artifacts-gamepad](https://github.com/Waregalias/artifacts-gamepad)
- Implémentation locale actuelle: `artifacts-gamepad-controller`
- Démo web: [waregalias.github.io/artifacts-gamepad](https://waregalias.github.io/artifacts-gamepad/)

## Pourquoi faire un gamepad controller pour Artifact MMO

Le jeu est excellent pour l'automatisation et les scripts, mais en pratique je voulais aussi une expérience plus immédiate:

- appuyer sur un bouton pour agir
- garder une boucle de jeu fluide
- profiter de l'API sans friction

L'objectif n'était pas de remplacer l'approche "code-first" d'Artifact MMO, mais de la rendre plus ergonomique au quotidien.

## La progression du projet

Le projet s'est construit en deux phases bien distinctes:

1. Démarrage manuel:
- structure de l'app
- premières routes API
- mapping des boutons et des directions
- gestion des erreurs et des états

2. Finalisation en vibe coding:
- accélération des itérations
- consolidation UI/UX
- ajustements rapides sur les bindings et les flux d'action
- passage à une version plus complète (web + desktop avec Electron)

Cette combinaison a été super efficace: base solide d'abord, vitesse d'exécution ensuite.

## Stack technique

Le contrôleur repose sur:

- Next.js + React + TypeScript
- `react-gamepads` pour capter les entrées manette et clavier/souris
- Zustand pour l'état global
- Electron pour le mode desktop

## Le coeur technique: input -> action -> API

L'app fait essentiellement trois choses:

1. Lire l'état de la manette et clavier/souris
2. Mapper les inputs vers des actions métier (fight, rest, move, etc.)
3. Appeler l'API Artifact MMO puis rafraîchir l'état du personnage

### 1) Mapping des boutons du contrôleur

```ts
export const controllerToGamePadSVG: Model = {
  '0': 'buttonDown',
  '1': 'buttonRight',
  '2': 'buttonLeft',
  '3': 'buttonUp',
  '4': 'TriggerFrontLeft',
  '5': 'TriggerFrontRight',
  '6': 'TriggerBackLeft',
  '7': 'TriggerBackRight',
  '8': 'select',
  '9': 'start',
  '12': 'directionUp',
  '13': 'directionDown',
  '14': 'directionLeft',
  '15': 'directionRight',
}
```

Cette table sert de point d'ancrage entre l'index physique des boutons et les actions lisibles côté app.

### 2) Stabiliser les entrées avec un délai court

```ts
const pressedButtons = Object.entries(currentButtonsClicked)
  .filter(([, isPressed]) => isPressed)
  .map(([button]) => button);

if (pressedButtons.length > 0) {
  if (timerRef.current) {
    clearTimeout(timerRef.current);
  }
  timerRef.current = setTimeout(() => {
    gamePadEvent(currentButtonsClicked);
  }, 200);
} else {
  gamePadEvent(currentButtonsClicked);
}
```

Ce mini debounce limite les doubles déclenchements non désirés et garde un comportement propre en jeu.

### 3) Mapping des boutons vers les actions Artifact MMO

```ts
export const ArtifactActionButtonMap = {
  buttonUp: 'rest',
  buttonRight: 'fight',
  buttonLeft: 'gathering',
  buttonDown: 'transition',
} as const;
```

Les directions sont traitées à part pour le déplacement:

```ts
export enum ArtifactActionMoveX {
  directionLeft = -1,
  directionRight = 1,
}

export enum ArtifactActionMoveY {
  directionUp = -1,
  directionDown = 1,
}
```

### 4) Déclenchement des requêtes API

```ts
export const move = async (
  apiKey: string,
  name: string = 'none',
  fx: number = 0,
  fy: number = 0,
  dx: number,
  dy: number
): Promise<ArtifactResponse> => {
  return fetch(`https://api.artifactsmmo.com/my/${name}/action/move`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      'Authorization': `Bearer ${apiKey}`,
    },
    body: `{"x":${fx + dx},"y":${fy + dy}}`,
  })
    .then(res => res.json())
    .then(json => {
      if (json.error) {
        throw new Error(json.error.message, json.error.code);
      }
      return json.data;
    });
}
```

Même principe pour `rest`, `fight`, `gathering` et `transition`.

## Ce que j'en retiens

Ce projet m'a confirmé deux choses:

- Artifact MMO est une excellente base pour créer des interfaces de jeu personnalisées.
- Le duo "démarrage manuel + finalisation en vibe coding" est redoutablement efficace.

Le manuel m'a permis d'ancrer la logique correctement.
Le vibe coding m'a permis d'aller beaucoup plus vite sur la finition, l'ergonomie et les itérations.

Au final, j'ai une app qui reste fidèle à l'esprit API d'Artifact MMO, mais avec une prise en main beaucoup plus naturelle à la manette et clavier/souris.

## Sources

- Documentation Artifact MMO: [docs.artifactsmmo.com](https://docs.artifactsmmo.com/)
- Repository du projet: [github.com/Waregalias/artifacts-gamepad](https://github.com/Waregalias/artifacts-gamepad)
