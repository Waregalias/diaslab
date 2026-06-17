---
title: "Artifact MMO : un controller gamepad fait maison"
published: 2026-04-21
description: Un petit outil perso pour jouer à Artifact MMO à la manette, construit en vibe coding.
image: "./game.png"
tags: [Artifact MMO, Gamepad, React, Next.js, Electron, API]
category: Projects
draft: false
---

[Artifact MMO](https://artifactsmmo.com/) est un MMO où tout passe par une API HTTP — chaque action du personnage est un appel REST. C'est fun à scripter, mais j'avais envie d'une interface plus directe : brancher une manette et jouer "à la console" plutôt que de tout coder.

- Repo: [Waregalias/artifacts-gamepad](https://github.com/Waregalias/artifacts-gamepad)
- Démo: [waregalias.github.io/artifacts-gamepad](https://waregalias.github.io/artifacts-gamepad/)

## Comment ça marche

L'app fait trois choses : lire la manette, mapper les boutons sur des actions de jeu, appeler l'API.

**Mapping des boutons physiques :**

```ts
export const controllerToGamePadSVG: Model = {
  '0': 'buttonDown',
  '1': 'buttonRight',
  '2': 'buttonLeft',
  '3': 'buttonUp',
  '12': 'directionUp',
  '13': 'directionDown',
  '14': 'directionLeft',
  '15': 'directionRight',
  // ...
}
```

**Actions de jeu :**

```ts
export const ArtifactActionButtonMap = {
  buttonUp: 'rest',
  buttonRight: 'fight',
  buttonLeft: 'gathering',
  buttonDown: 'transition',
} as const;
```

Un debounce de 200ms évite les doubles déclenchements, et les directions du stick gèrent les déplacements sur la map via des coordonnées x/y.

**Appel API :**

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
  }).then(res => res.json())
    .then(json => {
      if (json.error) throw new Error(json.error.message);
      return json.data;
    });
}
```

## Stack

Next.js + React + TypeScript, `react-gamepads` pour les inputs, Zustand pour l'état, Electron pour la version desktop.

Le projet a démarré à la main (structure, routing, gestion des états), puis j'ai accéléré la finition en vibe coding. Ça marche bien pour ce genre de chose : poser les bases soi-même, laisser l'AI finir le boulot répétitif.
