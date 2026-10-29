# HÉLIOM — pas à pas

Web app mono-fichier qui projette les ombres de bâtiments sur un plan GRDF
géoréférencé et dit si un poste de travail sera à l'ombre, et à quelles heures.
Tout le calcul est local dans le navigateur ; `index.html` se suffit à lui-même.

---

## 1. Mettre en ligne (GitHub Pages)

1. Créer un nouveau dépôt GitHub (ex. `heliom`).
2. Y déposer `index.html` à la racine (glisser-déposer dans l'onglet *Code* →
   *Add file* → *Upload files* → *Commit*).
3. *Settings* → *Pages* → *Build and deployment* → *Source : Deploy from a
   branch* → *Branch : main* / dossier `/ (root)` → *Save*.
4. Au bout d'une minute, l'URL s'affiche : `https://<utilisateur>.github.io/heliom/`.

> Le fichier a besoin d'internet au 1er chargement (il va chercher pdf.js et
> tesseract.js sur un CDN). Une fois la page ouverte, le calcul est hors-ligne ;
> seuls un nouveau plan OCR et un rechargement de page redemandent le réseau.

---

## 2. Utilisation

1. **OUVRIR UN PLAN PDF** → le plan s'affiche, le poste est pré-placé au centre.
2. **LIRE LE CARTOUCHE (OCR)** → remplit l'échelle et la position (coordonnée
   Lambert précise si lue, sinon GPS). Toujours **vérifier** les valeurs lues
   (un chiffre mal lu se corrige à la main ; l'échelle est aussi dans le nom du
   fichier : `PLAN_200` = 1:200). Cette étape est indispensable avant OSM : elle
   fixe la position du plan sur la Terre.
3. **CHARGER BÂTIMENTS OSM** → récupère automatiquement les bâtiments voisins et
   leur hauteur depuis OpenStreetMap et les pose en overlay sur le plan. Ceux
   avec une hauteur réelle sont signalés `OSM` ; les autres sont estimés
   (`niveaux×3` ou `défaut`). Si l'overlay est décalé par rapport au plan dessiné,
   **DÉPLACER L'OVERLAY OSM** (glisser) pour le recaler.
4. Au besoin, compléter : **TRACER BÂTIMENT** (clics + double-clic pour fermer),
   **ARBRE** (un clic = cercle 4 m) — utile car OSM ignore les arbres.
5. **PLACER / DÉPLACER LE POSTE** sur le point réel d'intervention.
6. Choisir la **date**, bouger le **curseur d'heure**. Le verdict (haut) et la
   **frise** (bas) se mettent à jour : heures d'ombre de la journée.
7. **EXPORTER LE PROJET** pour réutiliser le calage plus tard (le PDF n'est pas
   inclus : le re-sélectionner après import).

---

## 3. Structure du fichier (pour modifier)

Le `<script>` est découpé en 15 sections numérotées en commentaire. Pour une
modification, chercher le numéro :

| # | Section | Sert à |
|---|---------|--------|
| 1 | Position du soleil | algo solaire (ne pas toucher sauf raison) |
| 2 | Géométrie | point-dans-polygone, ombre concave, distances |
| 3 | État global `S` | toutes les valeurs courantes de l'app |
| 4 | Canvas & transform | zoom / déplacement, vecteur d'ombre |
| 5 | Rendu | dessin plan + ombres + bâtiments + poste |
| 6 | Verdict + marge | SOLEIL / OMBRE / LIMITE + distance au bord |
| 7 | Frise journalière | barre 24 h et heures de bascule |
| 9 | Chargement PDF | rendu du plan |
| 10 | OCR cartouche | lecture échelle + GPS |
| 11 | Souris | tracer / déplacer |
| 13 | UI | boutons, modes, champs |
| 14 | Sauvegarde / import | projet .json |

Les couleurs et le thème sont dans le `:root { … }` du `<style>` en haut.

---

## 4. Modifications courantes (où toucher)

- **Changer une couleur** (soleil/ombre/alerte/avertissement) → variables
  `--soleil`, `--ombre`, `--alerte`, `--avert` dans `:root`.
- **Seuil « LIMITE »** (actuellement 2,5 m, = précision de calage) → constante
  `SEUIL` dans la section 6.
- **Hauteur par défaut / rayon d'arbre** → champ `defH` (UI) ; rayon 4 m codé
  dans `makeTree` (section 11).
- **Pas de la frise** (actuellement 10 min) → boucles `m+=10` dans la section 7.
- **Transparence des ombres** → `COL.ombre` (section 5).
- **Largeur du cartouche pour l'OCR** (si une agence a un cartouche plus/moins
  large) → le `0.235` dans `runOCR` (section 10).

---

## 5. Modifier avec Claude Code

Déposer `index.html` dans le dépôt, puis demander par ex. :
> « Dans index.html, section 7, ajoute le trait de bascule (heure) en texte
> au-dessus de la frise. »

Indiquer le **numéro de section** aide à cibler. Après chaque modif : ouvrir la
page en local (double-clic sur `index.html`) et vérifier **sur un vrai plan**
avant de pousser sur GitHub.

---

## 6. Limites connues (à garder en tête)

- **OCR non fiable à 100 %.** Un plan dense peut faire déborder le trait dans le
  cartouche et corrompre une valeur. D'où la vérification manuelle systématique.
  La position lue conditionne le placement OSM : si elle est fausse, l'overlay
  tombe au mauvais endroit.
- **Hauteurs OSM incomplètes.** Beaucoup de bâtiments OSM n'ont pas de hauteur
  renseignée → estimation `niveaux×3` ou défaut. La source est affichée par
  bâtiment ; la marge du verdict dit si le résultat est robuste malgré l'estimation.
- **Overlay OSM à recaler.** La coordonnée du cartouche (surtout si seul le GPS
  arrondi est lu) et la position OSM ont chacune quelques mètres d'erreur →
  l'overlay peut être légèrement décalé, d'où le recalage manuel par glisser.
- **OSM ignore les arbres** (et parfois des bâtiments récents) → compléter au
  tracé manuel.
- **Relief non modélisé.** Une colline/montagne qui masque le soleil bas n'est
  pas prise en compte (effet réel en PACA aux heures basses).
- **Fuseau horaire = celui de la machine.** Correct en France.
- **Non testé en navigateur à la livraison.** Les maths (soleil, ombres,
  reprojection Lambert↔GPS) ont été vérifiées isolément ; l'interface + pdf.js +
  OCR + appel OSM doivent être essayés sur un vrai plan avant usage réel. En
  particulier, confirmer que l'overlay OSM tombe bien sur les bâtiments dessinés.

---

## 7. Charger la BD TOPO (meilleures hauteurs)

OSM a des footprints corrects mais des hauteurs souvent absentes. La BD TOPO de
l'IGN donne la hauteur (`HAUTEUR`) de presque tous les bâtiments en France.
L'app sait l'importer via **IMPORTER GEOJSON (BD TOPO)** — mais la BD TOPO ne se
charge pas brute : elle arrive en Shapefile/GeoPackage par département (trop gros
pour un navigateur). Il faut d'abord **découper et convertir en GeoJSON** :

**Option A — QGIS (visuel).**
1. Télécharger la BD TOPO du département sur geoservices.ign.fr, ouvrir la couche
   `BATIMENT` dans QGIS.
2. Zoomer sur la zone du site. *Vecteur → Outils de géotraitement → Couper*
   (ou sélectionner les bâtiments voulus).
3. Clic droit sur la couche → *Exporter → Sauvegarder les entités sous* →
   format **GeoJSON**, SCR **EPSG:4326 (WGS84)**.
4. Dans HÉLIOM : OCR du cartouche, puis **IMPORTER GEOJSON**.

**Option B — ogr2ogr (une ligne, GDAL).**
```
ogr2ogr -f GeoJSON sortie.geojson BATIMENT.shp -t_srs EPSG:4326 \
        -spat <lon_min> <lat_min> <lon_max> <lat_max>
```
(`-spat` = emprise à garder ; `-t_srs EPSG:4326` = reprojection en WGS84.)

L'app détecte seule le système de coordonnées du fichier (WGS84, Lambert-93 ou
Lambert II étendu) et reprojette ; l'attribut `HAUTEUR` est utilisé comme hauteur.

**Alternative sans fichiers — le service en ligne WFS.** `data.geopf.fr` peut
renvoyer la BD TOPO par emprise, comme OSM, sans rien télécharger. Même mécanique
que `fetchOSM` (section 10bis) en changeant l'URL et le nom de couche. Point à
valider avant de coder : le **CORS** (le navigateur est-il autorisé à appeler
data.geopf.fr). Si oui, c'est l'option la plus fluide ; si non, rester sur
l'import GeoJSON ci-dessus.
