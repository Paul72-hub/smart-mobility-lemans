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

> **Reflection 7.** Before writing any code: you're about to use commune
> **population** as a stand-in for real transport **demand**, because real ridership
> data doesn't exist as open data. Name one concrete, realistic way a commune could
> have *low* population but genuinely *high* transport demand (or the reverse) — so
> you go into this exercise already knowing the proxy's failure mode, not just its
> formula.

Peu d'habitants mais beaucoup de demande : une commune qui accueille une grande zone d'emplois, une zone commerciale, un hôpital ou une université. Des milliers de personnes y viennent chaque jour sans y habiter. Exemples au Mans : le circuit des 24 Heures (Mulsanne / Arnage) pendant les courses, ou une zone d'activités.
Beaucoup d'habitants mais moins de demande : une commune résidentielle aisée où presque tout le monde a une voiture, ou une population âgée qui se déplace peu aux heures de pointe.
La conséquence : la population mesure où les gens dorment, pas où ils vont. Le ratio offre/population va donc sous-estimer les besoins des zones d'emploi et d'événements.


> **Reflection 8.** State your chosen supply metric in one sentence. Then, using
> your own actual output (not a hypothetical), name one commune where the ranking
> looks wrong to you, and explain why — using what you actually know about that
> commune, not just the formula.

L'indicateur en une phrase : « le nombre de passages de bus/tram prévus par jour de semaine dans les arrêts d'une commune, pour 1 000 habitants ».
La commune où le classement est faux : Le Mans, seulement 10ᵉ avec 154 passages pour 1 000 habitants, derrière La Chapelle-Saint-Aubin (1ʳᵉ, 1 010) et Coulaines (2ᵉ, 762). C'est absurde : Le Mans concentre les 3 lignes de tram et les plus gros pôles (Gares, Mairie).
Pourquoi : chaque arrêt est rattaché à la commune dont le centre est le plus proche, pas à la commune où il se trouve réellement. Le Mans est très étendue mais n'est représentée que par un seul point, son centre-ville. Les arrêts des quartiers périphériques du Mans sont donc plus proches du centre de La Chapelle-Saint-Aubin ou de Coulaines, et leur sont attribués à tort. Résultat : ces petites communes « volent » des arrêts au Mans (53 arrêts pour 2 300 habitants à La Chapelle, par exemple), ce qui gonfle leur score et fait baisser celui du Mans.

> **Reflection 9.** Go back to Reflection 4's answer about per-route vs. per-stop
> logging. If you wanted "which *stops* are affected by repeated delays" answered
> from real-time data (not the static schedule), you'd hit that same data-volume
> problem again. You don't have to build this — but describe, in a few sentences, one
> concrete approach you'd take to aggregate stop-level events so a semester of
> 15-minute snapshots stays a reasonably-sized, git-tracked file.

Le principe : ne pas stocker chaque relevé brut, mais des compteurs agrégés par arrêt et par créneau.
L'approche : un fichier avec une ligne par arrêt × tranche horaire × jour de la semaine, par exemple « arrêt Gares, mardi, 8h-9h ». À chaque relevé, on met à jour les compteurs de cette ligne au lieu d'en ajouter une nouvelle :
le nombre d'observations ;
le nombre de fois où le retard a dépassé 2 minutes ;
la somme des retards (pour recalculer la moyenne) et le retard maximum.
La taille : environ 1 800 arrêts × 7 jours × environ 18 heures de service ≈ 230 000 lignes au maximum, et ce chiffre ne grossit plus avec le temps, contre 40 millions en brut.
Le compromis : on perd le détail de chaque bus et la date exacte. Mais on peut toujours répondre à la question « quels arrêts sont souvent en retard, et à quelle heure ? ». Variante possible : ne garder que les événements de retard au-delà d'un seuil, par exemple plus de 3 minutes, au lieu de tout.

> **Reflection 10 (closing).** Return to Reflection 1. Which of the business
> question's sub-questions turned out to be answerable today, which needed real
> judgment calls from you, and which genuinely need weeks of accumulated history
> before they're answerable at all? Compare this to what you originally guessed.

Ce qu'on peut faire dès aujourd'hui : savoir où se trouve l'offre (arrêts les plus desservis : Gares, Mairie) et voir les retards du moment (ligne 5).
Ce qui a demandé des choix de votre part :
utiliser la population comme approximation de la demande ;
choisir l'indicateur (passages pour 1 000 habitants) ;
rattacher les arrêts au centre de commune le plus proche, ce qui a faussé le classement du Mans ;
choisir les couleurs du graphique (avance ou retard) ;
décider d'agréger par ligne plutôt que par arrêt.
Ce qui demande des semaines d'historique : savoir quand intervenir (retards récurrents aux heures de pointe, certains jours) et prévoir les retards. Avec un ou deux relevés, c'est impossible.
La comparaison avec la réflexion 1 : votre hypothèse sur la fréquence des pipelines était juste. En revanche, ajoutez la partie « facile ou difficile » qui manquait à votre réflexion 1, et dites ce qui vous a surpris. Par exemple : le classement par commune semblait facile, mais il s'est révélé trompeur à cause du rattachement des arrêts.