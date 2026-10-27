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
Version actuelle : **12**.

## État du jeu
- Carte procédurale 96×96 (graine), relief jusqu'à 50 niveaux, rivières qui suivent le courant, mer en bordure, 5 essences de forêt, 18 filons dans des parois de montagne (mines en façade).
- Brouillard : plus d'explorateurs (retirés en v10) ; ce sont les tours de guet qui découvrent la carte (`towerReveal`, rayon de vue `see` 11/15/20 selon le niveau, dès la construction, sans soldats ni route) ; 4 angles de vue (boutons, Q/E, glisser à deux doigts, clic droit) ; jours automatiques ×1/×2/×3 avec pause, bouton « jour suivant » avec lever de soleil ; fond papier.
- Ères : I (rizière, maison, bûcheron, défricher case par case, canal) → II à 12 hab. (chemins 3 niveaux terre/pierre/pavé avec marches rondins/ishigaki/torii, ponts sur plusieurs cases d'une rive à l'autre, mines 3 niveaux, maison améliorée) → III à 40 hab. (temple 5×7, château 9×9 avec jardin, chantiers sur plusieurs jours).
- Ère II aussi (v11) : chaîne du métal mine → 鉄 minerai → forge 1×1 `forge` (2 鉄 + 2 木 → 1 鋼 acier/jour, niv. 2 « Haut fourneau tatara » ×2, sprite clé `K`) → armurerie 2×2 (`EQUIP` : 槍 lance 1鋼+1木, 刀 sabre 2鋼, 鎧 armure 2鋼+1銅 réservée à l'« Atelier d'armurier » niv. 2 ; mode `b.mode` auto/槍/刀/鎧, auto = ce qui manque puis de quoi élever une tour, `armoryPick`) ; poste de garde 3×3 passé en ère II ; tours de guet 3 niveaux (2/3/4 soldats, terres 6/8/11, vue 11/15/20), remparts case par case 3 niveaux (palissade / ishigaki / mur blanc couvert) ; rempart posé sur un chemin = porte.
- Armée : les soldats sont des habitants logés par les postes de garde (8 chacun) et le château (30) (`SOLDIER_CAP`, `soldierCap/soldierUsed`) ; tours, postes de garde et château (`mil:true`) prennent leurs soldats dans ce total (sinon état `soldats`). Chaque soldat reçoit l'équipement de son poste (`tierOf` : tour = son niveau 槍/刀/鎧, poste = 槍, château = 刀 ; champ `arms`, rendu à la démolition et à l'amélioration). Élever une tour exige l'équipement du niveau suivant en réserve. Tours, postes de garde et armurerie (`road:true`) doivent toucher le réseau de chemins.
- Stockage (v12) : chaque ressource a une place maximale `storeCap` (calculée dans `allocate` → `computeStore`) = réserve du village `STORE_BASE` + un peu par maison `HOUSE_STORE` (×2 maison améliorée) + entrepôts 2×2 (champ `store` : `kura_riz` grenier à riz +500 米 et `kura_bois` +400 木 en ère I ; `kura_ishi` +400 石, `kura_kane` minerais +200 鉄/150 銅/80 金, `kura_arme` arsenal +120 鋼/50 槍 刀 鎧 en ère II ; comptés une fois le chantier fini, sans ouvriers ni route). Toute production passe par `gain(k,v)` qui plafonne ; ce qui ne rentre pas est perdu (« 米 plein », toast). Une réserve déjà au-dessus (vieille partie) est gardée mais ne grossit plus. Entrepôts dessinés par `STORES[type]` (kura à murs namako, `blitBig`).
- Territoire : terres de départ + cercles des postes armés en service ; rayon +1 par 6 points de rempart à portée (palissade 1, ishigaki 2, mur couvert 3, porte 2 ; plafond +3 tour, +4 garde, +6 château).
- Enceinte : cases inaccessibles depuis le bord de carte sans franchir rempart ou rivière (douves) → +1 place par maison, +1 米 par rizière.
- Maisons (6 modèles), forêts et camp de bûcherons dessinés une fois puis recopiés (cache de sprites) pour les performances.
- Bâtiments militaires dessinés avec les mêmes outils que les maisons (repère local `HF`, `hbox`, `planks`, `lattice`, `irimoya`…) : tours `TOWERS[0..2]` et remparts/portes en sprites (`blit`, clés `T`, `W<niveau>_<masque>`, `G`), armurerie `drawArmory` (repère `HFat`, dessinée à chaque image). Pierres d'ishigaki : `stoneCourses`.
- Temple et château détaillés (`drawTempleD`, `drawCastleD`, appelés par `drawGrand`) : dessinés une fois par zoom dans une grande image en cache (`blitBig`, clé `B<type><A>`, repli en dessin direct au-delà de 3000 px). Temple : enceinte ocre à lignes blanches, grand hall, pagode à 5 toits, clocher, fontaine, lanternes, brûle-parfum, torii, porte. Château : douves, talus d'ishigaki, murs à meurtrières, 3 tourelles d'angle, porte-tour, pont, palais, jardin à étang, donjon à 5 niveaux (pignons chidori, kara-hafu, shachihoko).
- Sauvegarde : `localStorage` clé `kunizukuri5`.

## Pistes suivantes évoquées
Outils en métal, maison de santé, menaces extérieures (bandits ?) pour donner un rôle actif à l'armée ; dessiner le terrain par morceaux mis en cache pour fluidifier les déplacements sur téléphone.
