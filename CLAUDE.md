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
Version actuelle : **9**.

## État du jeu
- Carte procédurale 96×96 (graine), relief jusqu'à 50 niveaux, rivières qui suivent le courant, mer en bordure, 5 essences de forêt, 18 filons dans des parois de montagne (mines en façade).
- Brouillard + explorateurs ; 4 angles de vue (boutons, Q/E, glisser à deux doigts, clic droit) ; jours automatiques ×1/×2/×3 avec pause, bouton « jour suivant » avec lever de soleil ; fond papier.
- Ères : I (rizière, maison, bûcheron, défricher case par case, canal) → II à 12 hab. (chemins 3 niveaux terre/pierre/pavé avec marches rondins/ishigaki/torii, ponts sur plusieurs cases d'une rive à l'autre, mines 3 niveaux, maison améliorée) → III à 40 hab. (temple 5×7, poste de garde 3×3, château 9×9 avec jardin, chantiers sur plusieurs jours).
- Ère II aussi : armurerie 2×2 (2 鉄 + 1 木 → 1 刀 par jour, niveau 2 « Grande forge » ×2), tours de guet 3 niveaux (2/3/4 soldats, rayon 6/8/11), remparts case par case 3 niveaux (palissade / ishigaki / mur blanc couvert) ; rempart posé sur un chemin = porte.
- Armée : tours, postes de garde et château (`mil:true`) ont besoin de soldats (ouvriers) armés d'un 刀 chacun (pris une fois, rendu à la démolition, champ `arms`). Tours, postes de garde et armurerie (`road:true`) doivent toucher le réseau de chemins.
- Territoire : terres de départ + cercles des postes armés en service ; rayon +1 par 6 points de rempart à portée (palissade 1, ishigaki 2, mur couvert 3, porte 2 ; plafond +3 tour, +4 garde, +6 château).
- Enceinte : cases inaccessibles depuis le bord de carte sans franchir rempart ou rivière (douves) → +1 place par maison, +1 米 par rizière.
- Maisons (6 modèles), forêts et camp de bûcherons dessinés une fois puis recopiés (cache de sprites) pour les performances.
- Bâtiments militaires dessinés avec les mêmes outils que les maisons (repère local `HF`, `hbox`, `planks`, `lattice`, `irimoya`…) : tours `TOWERS[0..2]` et remparts/portes en sprites (`blit`, clés `T`, `W<niveau>_<masque>`, `G`), armurerie `drawArmory` (repère `HFat`, dessinée à chaque image). Pierres d'ishigaki : `stoneCourses`.
- Temple et château détaillés (`drawTempleD`, `drawCastleD`, appelés par `drawGrand`) : dessinés une fois par zoom dans une grande image en cache (`blitBig`, clé `B<type><A>`, repli en dessin direct au-delà de 3000 px). Temple : enceinte ocre à lignes blanches, grand hall, pagode à 5 toits, clocher, fontaine, lanternes, brûle-parfum, torii, porte. Château : douves, talus d'ishigaki, murs à meurtrières, 3 tourelles d'angle, porte-tour, pont, palais, jardin à étang, donjon à 5 niveaux (pignons chidori, kara-hafu, shachihoko).
- Sauvegarde : `localStorage` clé `kunizukuri5`.

## Pistes suivantes évoquées
Outils en métal, maison de santé, menaces extérieures (bandits ?) pour donner un rôle actif à l'armée ; dessiner le terrain par morceaux mis en cache pour fluidifier les déplacements sur téléphone.
