# Mes photos de famille — prototype

Application pour **trier, dédoublonner et étiqueter** des photos d'actes généalogiques et de documents familiaux.
Pensée pour une personne âgée (gros boutons, fort contraste, pas de jargon) et utilisable aussi par des enseignants
qui annotent des devoirs scannés. **100 % local** : aucune photo ne quitte l'ordinateur.

## Pour la faire tester (rien à installer)

1. Copier le fichier `index.html` sur l'ordinateur (clé USB, e-mail…), par exemple sur le Bureau.
2. Double-cliquer dessus → il s'ouvre dans le navigateur. **Utiliser Google Chrome ou Microsoft Edge** (la voix ne marche pas dans Firefox).
3. Sur l'accueil : « 🧪 Essayer avec des photos d'exemple » crée de faux actes, dont des doubles, pour découvrir l'appli.

Le travail est enregistré automatiquement dans le navigateur (IndexedDB), lié à ce navigateur et à cet emplacement de fichier :
toujours rouvrir **le même fichier avec le même navigateur**. Le bouton « 💾 Sauvegarder mon travail » exporte un fichier JSON
(étiquettes, descriptions, dictionnaire) et un tableau CSV pour Excel/LibreOffice.

## Étiquettes inscrites dans les photos

Au premier lancement, l'application demande **les initiales** de la personne et **ce qu'elle veut faire des étiquettes** (modifiable dans *Réglages*) :

- **Les inscrire dans mes photos** (Chrome / Edge) : à l'ajout, Chrome demande l'autorisation de modifier le dossier. Les étiquettes sont ensuite
  inscrites automatiquement quelques secondes après chaque changement et en passant à la photo suivante.
- **Ne jamais toucher à mes photos** : les fichiers restent intacts, les étiquettes restent dans l'application, la liste se télécharge en tableau
  (une colonne par zone, description, baliseurs, date d'origine).

| Format | Où vont les étiquettes |
|---|---|
| JPEG | XMP + IPTC dans le fichier (EXIF conservé tel quel) |
| PNG, WebP | XMP dans le fichier |
| GIF, BMP, AVIF… | fichier compagnon `photo.ext.xmp` à côté (l'image n'est pas touchée) |

Contenu inscrit : chaque étiquette comme mot clé (`dc:subject`, IPTC Keywords → visibles dans l'Explorateur Windows, vérifié),
les catégories `Zone|Étiquette` (`lr:hierarchicalSubject` → digiKam, Lightroom), la description (`dc:description`, affichée en « Titre » par Windows),
`Baliseur : XX` pour chaque personne qui a étiqueté (et `photoshop:CaptionWriter`), et **`Date d'origine : AAAA-MM-JJ HH:MM`** :
toute écriture change la date du fichier, alors la date d'avant la première inscription est gardée dans la photo (`mpf:OriginalFileDate`)
et c'est elle que l'application affiche et utilise.

Sécurité : les pixels ne sont jamais recompressés ; avant d'écrire, l'application vérifie que l'image est toujours la même et que les étiquettes
se relisent ; Chrome écrit dans un fichier temporaire puis remplace l'original d'un coup. Les informations posées par d'autres logiciels
(note, EXIF, mots clés ajoutés ailleurs) sont conservées. Un fichier aux informations abîmées n'est pas touché.

À l'ajout, les étiquettes déjà présentes dans les photos sont reprises : celles de l'application retrouvent leur zone, celles d'autres logiciels
arrivent en haut, **encadrées de pointillés** (appui long pour les ranger). L'empreinte des doublons porte sur l'image seule : une copie étiquetée
reste reconnue comme double d'une copie qui ne l'est pas. *Réglages › Relier des photos à leur dossier* rattache les photos ajoutées sans accès.

## Apparence

- **Élégant** (par défaut) : papier ivoire, encre bleu nuit, touches de bronze, titres à empattements (Palatino / Iowan / Georgia,
  polices déjà présentes sur l'ordinateur, rien n'est téléchargé), cartes douces, icônes vectorielles au trait intégrées au fichier.
- **Contraste élevé** (thème de secours) : le premier style du prototype, traits épais et couleurs franches. Se choisit dans *Réglages › Apparence*.
- Les étiquettes gardent un numéro de couleur : chaque thème l'affiche avec sa propre teinte.

## Ce que fait le prototype

| Besoin | Réalisation |
|---|---|
| Import depuis SD / disque | Choix d'un dossier entier (sous-dossiers compris), de quelques photos, ou glisser-déposer d'un dossier sur la fenêtre. Les originaux ne sont jamais modifiés : l'appli garde une copie réduite (2200 px, lisible pour un acte) + une vignette. |
| Doublons exacts | Empreinte **SHA-256** de chaque fichier. Les copies identiques partagent la même image stockée. Bouton « garder une seule copie de chaque » en un clic. |
| Quasi-doublons | Empreinte visuelle 32×32 en niveaux de gris, comparée par corrélation (≥ 95 %), ou même poids (±2 %) + dates proches (±10 min) + allure voisine. Proportions très différentes = ignoré. |
| Comparaison | Deux images côte à côte, différences surlignées, « ★ Meilleure qualité » indiquée. Un clic (ou ← / →) garde une photo, l'autre va dans la corbeille ; ses étiquettes et sa description sont reportées. « Ce ne sont pas des doubles » est mémorisé. Tout est annulable (↶ Annuler, Ctrl+Z). |
| Voix | Web Speech API en `fr-FR`, **uniquement sur l'ordinateur** (`processLocally = true`, Chrome récent) avec les étiquettes et le dictionnaire comme mots favoris ; l'appli propose d'installer la voix française une fois si besoin. **Aucun envoi en ligne** : si le navigateur ne sait pas reconnaître la voix localement, la voix est simplement indisponible (le clavier reste possible). |
| Mains libres | Un clic sur « Parler » (ou la **barre d'espace**) : le micro reste ouvert et on enchaîne « mariage Dupond Rennes, suivante… ». Coupé par « stop », un clic, la barre d'espace, en quittant l'écran ou après 3 min sans parole. |
| Description à la voix | « description » fait passer en **dictée libre** (le champ s'encadre, le bouton affiche « Dictée de la description… ») : tout ce qui est dit s'ajoute au texte. Ponctuation à la voix (« virgule », « point », « point d'interrogation », « à la ligne », parenthèses, guillemets), majuscules automatiques, dictionnaire personnel appliqué. Dits **seuls**, « étiquettes » ramène aux étiquettes, « annuler » retire la dernière phrase, « suivante » / « stop » restent des commandes (dans une phrase, ce sont des mots normaux). Changer de photo ramène aux étiquettes. |
| Compréhension | Les phrases sont découpées, les petits mots ignorés, les étiquettes reconnues même mal orthographiées (distance d'édition + « clé sonore » française : Dupont ≈ Dupond ≈ Dupon). Commandes, n'importe où dans la phrase et exécutées dans l'ordre : « suivante », « précédente », « enlève Rennes », « annuler », « stop » ; « description … » en début de phrase (passage en dictée). |
| Dictionnaire personnel | Pour chaque mot non reconnu : « ＋ l'ajouter » dans une zone, ou « ✎ Mal compris, corriger » → le couple *entendu → correct* est retenu et appliqué aux prochaines dictées. Gérable dans Réglages. |
| Badges autour de l'image | 3 zones (haut / gauche / droite, renommables) qui défilent. Clic = coller/décoller ; glisser vers la photo = coller ; **appui long** = l'étiquette se soulève : la lâcher sur une autre pour les **échanger**, sur une autre zone pour l'y **déplacer**, ou sur place pour ouvrir le panneau (nom, couleur, zone, ordre, suppression). « ＋ Nouveau » dans chaque zone. Le compteur sur chaque badge indique son usage. |
| Enlever une étiquette | Geste : **jeter** l'étiquette collée vers le bord (elle devient rouge « lâchez pour enlever »), ou ✕. Annulable. |
| Description libre | Champ texte enregistré automatiquement, ou dictée « description … ». |
| Retrouver | Recherche plein texte (nom, dossier, étiquettes, description), filtres « sans étiquette », par étiquettes combinées, corbeille (remettre / vider). |

## Architecture recommandée pour la suite

```
            ┌──────────────── Cœur web commun (HTML/CSS/JS ou TypeScript) ───────────────┐
            │  ui/        écrans : accueil, doublons, étiquetage, galerie, réglages       │
            │  domain/    règles pures : doublons, analyse vocale, dictionnaire, tags     │
            │  ports/     interfaces : Storage, FileSource, Speech, Hasher                │
            └───────────────┬───────────────────────┬───────────────────────┬─────────────┘
                            │                       │                       │
              Navigateur (ce prototype)     PC : Tauri 2                Tablette : Tauri 2 mobile
              IndexedDB, <input file>,      SQLite (plugin sql),        (ou Capacitor) SQLite,
              Web Speech API                accès disque natif,         sélecteur de fichiers natif,
                                            Vosk / whisper.cpp local    reconnaissance native iOS/Android
```

**Choix proposés**

- **Interface : web (HTML/CSS/JS)** — un seul code pour PC et tablette, les *Pointer Events* gèrent souris et doigt de la même façon
  (déjà le cas ici : appui long, glisser, geste de rejet fonctionnent au toucher).
- **Emballage : Tauri 2** plutôt qu'Electron : exécutable léger (~10 Mo contre ~150 Mo), Windows/macOS/Linux **et** Android/iOS depuis
  la v2, accès natif aux fichiers (vraie suppression/déplacement des doublons sur le disque, lecture des cartes SD). Alternative mobile : Capacitor.
  Flutter est excellent aussi, mais imposerait de tout réécrire en Dart ; React Native est moins à l'aise sur PC.
- **Stockage : SQLite** dans la version emballée (tables `photos`, `tags`, `photo_tags`, `dictionary`, `ignored_pairs`, `settings`) —
  robuste, transactionnel, une seule base à sauvegarder. Le prototype utilise IndexedDB avec le même découpage.
- **Voix 100 % locale garantie** : dans Tauri, **Vosk** (modèle français ~50 Mo, temps réel, grammaire restreignable aux étiquettes =
  excellente précision sur les noms rares) ou **whisper.cpp** (plus précis sur la dictée libre, plus lourd). Sur iPad, la reconnaissance
  Apple « sur l'appareil » ; sur Android, `SpeechRecognizer` avec `EXTRA_PREFER_OFFLINE`.
- **Doublons** : SHA-256 pour l'exact ; empreinte visuelle pour le proche. Au-delà de quelques milliers de photos, remplacer la comparaison
  « tous contre tous » par un index (pHash + arbre BK) et faire le calcul dans un *Web Worker* ou côté Rust.

**Modèle de données**

```
Photo   { id, hash(SHA-256), blobId, name, path, size, lastModified, width, height,
          sig(empreinte 32×32), tags[tagId], description, trashed, trashReason, copies[path] }
Tag     { id, label, zone(top|left|right), order, color }
Dict    [ { heard, correct } ]
Meta    settings (taille du texte, noms des zones, mode voix), ignored (paires « pas des doubles »)
```

## Organisation du code (`index.html`)

Un seul fichier volontairement, découpé en 8 sections numérotées :
1. utilitaires (normalisation, clé sonore, distance d'édition, SHA-256, boîtes de dialogue),
2. stockage IndexedDB, 3. état + navigation, 4. import (décodage, réduction, empreintes, photos d'exemple),
5. doublons, 6. étiquettes (zones, glisser-déposer, appui long, geste de rejet), 7. voix + dictionnaire,
8. galerie, sauvegarde, réglages.

Pour passer à Tauri : découper ces sections en modules (`domain/`, `ports/`, `ui/`) et remplacer seulement
`DB`, `importFiles` et `voiceStart` par des implémentations natives.

## Limites connues du prototype

- Les formats HEIC (iPhone) et TIFF ne sont pas lisibles par le navigateur : ils sont signalés à l'import.
- La corbeille est celle de l'application : les fichiers d'origine ne sont jamais effacés ni déplacés (volontaire, pour la sécurité).
- L'accès aux dossiers depuis une page ouverte d'un double-clic (`file://`) reste à confirmer dans Chrome sur l'ordinateur réel ;
  en cas de refus, l'application le dit et propose d'ajouter les photos sans inscription.
- Mots clés EXIF Windows (`XPKeywords`) : lus si rien d'autre n'existe, mais jamais réécrits (le XMP fait foi pour Windows).
- Les données sont liées au navigateur : sauvegarder régulièrement (💾).
- Pas encore de rotation d'image ni de recadrage.
