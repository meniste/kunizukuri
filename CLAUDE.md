# Kunizukuri 国造り — notes pour reprendre le projet

City builder 2.5D (isométrique) dans le Japon féodal, joué sur iPhone en application web installée (GitHub Pages, hors ligne).
Page de travail sur claude.ai : https://claude.ai/artifact/CLhV2JWH68gFYdVze4YpMX (même jeu, sans la couche hors ligne).

## Fichiers
- `index.html` : tout le jeu (HTML + CSS + JS dans un seul fichier, canvas 2D, aucune dépendance sauf Google Fonts).
- `sw.js` : service worker hors ligne. `const VERSION = 'kunizukuri-vN'`.
- `manifest.webmanifest`, icônes `icon-*.png`, `apple-touch-icon.png`.

## Publier une mise à jour
1. Modifier `index.html` et tester (Chromium/Playwright ; vérifier aussi les rendus qui pourraient différer sur Safari).
2. Incrémenter le numéro partout : `VERSION` dans `sw.js` **et** le libellé `· version N` dans `index.html` (script en bas de page, visible dans le menu 国).
3. Commit sur `main` (GitHub Pages publie `main` / racine). Le téléphone récupère la version à la prochaine ouverture avec réseau et se recharge seul.
4. Republier aussi la page claude.ai (sans le bloc service worker / manifest).
Version actuelle : **6**.

## État du jeu
- Carte procédurale 96×96 (graine), relief jusqu'à 50 niveaux, rivières qui suivent le courant, mer en bordure, 5 essences de forêt, 18 filons dans des parois de montagne (mines en façade).
- Brouillard + explorateurs ; 4 angles de vue (boutons, Q/E, glisser à deux doigts, clic droit) ; jours automatiques ×1/×2/×3 avec pause, bouton « jour suivant » avec lever de soleil ; fond papier.
- Ères : I (rizière, maison, bûcheron, défricher case par case, canal) → II à 12 hab. (chemins 3 niveaux terre/pierre/pavé avec marches rondins/ishigaki/torii, ponts sur plusieurs cases d'une rive à l'autre, mines 3 niveaux, maison améliorée) → III à 40 hab. (temple 5×7, poste de garde 3×3, château 9×9 avec jardin, chantiers sur plusieurs jours).
- Territoire : terres de départ + cercles des postes de garde / château en service.
- Maisons (6 modèles), forêts et camp de bûcherons dessinés une fois puis recopiés (cache de sprites) pour les performances.
- Sauvegarde : `localStorage` clé `kunizukuri5`.

## Pistes suivantes évoquées
Forge (outils et armes en métal), armée qui agrandit et doit garder le territoire, maison de santé, remparts ; dessiner le terrain par morceaux mis en cache pour fluidifier les déplacements sur téléphone.
