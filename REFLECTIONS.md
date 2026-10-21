# Reflections — Smart Mobility Le Mans


> **Reflection 1.** Before reading the workflow steps later in this document: the
> static feed will run on a monthly schedule, the real-time feed every 15 minutes.
> Both *could* technically run every 15 minutes. Write your own guess now for why the
> static one doesn't — you'll check it against Stage 5's reasoning.

Les horaires théoriques changent rarement (la SETRAM publie un nouveau fichier tous les quelques mois) ;
Le relancer toutes les 15 minutes téléchargerait et recalculerait exactement les mêmes données pour rien
à l'inverse, les retards changent en permanence, d'où les 15 minutes pour le temps réel.

> **Reflection 2.** Before running anything: if a tool infers each file's column
> types automatically and independently (so one file's `stop_id` column becomes
> numbers, another file's `stop_id` column stays text, just because of what values
> happened to appear in each file), what goes wrong when you try to join those two
> files on `stop_id`? Write your prediction, then compare it to the comment in the
> script below explaining why every GTFS file is read with `dtype=str`.

Si un fichier contient uniquement des identifiants comme 1001, pandas les lit comme des nombres. Dans un autre fichier où se trouve aussi "FLEX-381", toute la colonne reste du texte.
Pour pandas, le nombre 1001 et le texte "1001" sont deux valeurs différentes : la jointure ne trouve donc pas de correspondance.
Le pire, c'est que l'erreur est silencieuse. Pas de plantage, mais des lignes perdues ou des valeurs vides après le merge, et des résultats faux sans qu'on s'en rende compte. Selon les cas, pandas peut aussi lever une erreur.
La solution du script : lire tous les fichiers avec dtype=str (ligne 210 du manuel), pour que tous les identifiants soient du texte
partout et que la jointure fonctionne.

 **Reflection 3.** Before reading further: roughly how many individual stop-time
> update rows would a 12-week semester of 15-minute snapshots produce, if you logged
> every one of them? (Order of magnitude is the point, not exact math.) Now compare
> that to logging one row **per route** per snapshot instead (there are only 27-40
> routes). What real analytical question — something a transport planner might
> actually ask — becomes harder to answer with the per-route version? Keep your
> answer; you'll need it again in the final section.

Le calcul, version par arrêt : on compte 4 relevés par heure × 24 h × 7 jours × 12 semaines ≈ 8 000 relevés. À environ 5 000 lignes par relevé, cela fait environ 40 millions de lignes, beaucoup trop pour un fichier suivi par Git.
Le calcul, version par ligne de bus : 8 000 relevés × environ 30 lignes ≈ 240 000 lignes, soit à peu près 150 fois moins, ce qui reste gérable.
Ce qu'on perd : on ne sait plus à quel arrêt ou sur quel tronçon le retard apparaît. On sait seulement que la ligne 5 est en retard en moyenne, pas que le retard se forme à tel carrefour ou dans tel quartier. Un planificateur ne peut donc pas savoir où intervenir précisément (couloir de bus, feu prioritaire…).


> **Reflection 4.** Is that the right way to encode this KPI? A bus arriving very
> early can strand a waiting passenger just as effectively as a very late one — from
> a passenger's perspective, both extremes might be "bad." Propose an alternative
> color/encoding rule for this same chart, and explain what decision or conclusion a
> viewer would reach *differently* under your rule versus the one below.

Le problème : aujourd'hui, une ligne en avance, même de 3 minutes, s'affiche en vert, donc comme « bonne ». Pourtant, un bus parti en avance fait rater le bus au passager qui arrive à l'heure prévue, exactement comme un bus en retard le fait attendre.
La règle proposée : colorer selon l'écart à l'horaire, dans les deux sens. Par exemple :
vert entre -1 min et +2 min (à l'heure) ;
orange puis rouge plus on s'éloigne, que ce soit en avance ou en retard ;
éventuellement deux couleurs différentes, bleu pour l'avance et rouge pour le retard, avec une intensité qui augmente avec l'écart.
Ce que ça change pour la décision : avec la règle actuelle, on conclut que les lignes vertes vont bien et on ne s'en occupe pas. Avec la nouvelle, on repère aussi les lignes systématiquement en avance. Pour celles-là, la solution n'est pas la même que pour un retard : il faut revoir les horaires ou imposer aux bus d'attendre aux arrêts de régulation, plutôt qu'ajouter des bus ou un couloir réservé.

 **Reflection 5.** Now check your Reflection 2 guess against the real answer: why
> would a 15-minute schedule for the *static* GTFS reference be wasteful, and why
> would a monthly schedule for the *real-time* delay snapshots make the whole
> real-time picture useless? Answer both halves — the mismatch runs in both
> directions.

Notre hypothèse réflexion 1 était juste


> **Reflection 6.** Pick one route currently showing a large delay on your
> dashboard. Compare it against
> [SETRAM's own live departure information](https://www.setram.fr/) or plain common
> sense (is there a known local event, roadwork, rush hour?). Did the number match
> what you'd expect? If not, what's your best guess for the discrepancy — a real
> disruption, a data artifact, or something else? This is the same "does this number
> make sense" habit this whole course keeps returning to; write what you actually
> found, not what you expected to find.

La ligne choisie : la ligne 5, avec 144 s de retard moyen, 712 s au maximum et 56 arrêts sautés, relevée à 11h45.
vérif: Il y a bien des travaux sur cette ligne
analyse:
11h45 n'est pas une heure de pointe, donc le trafic seul explique mal ce retard ;
56 arrêts sautés, c'est le signe typique d'une déviation (travaux, voie fermée) : le bus contourne une partie de son trajet ;

## Reflection 7 — When population misleads as a demand proxy


## Reflection 8 — Supply metric and a commune where it misleads


## Reflection 9 — Aggregating stop-level delay events


## Reflection 10 — Closing: predictions vs. what I found
