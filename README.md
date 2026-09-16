# Supports de cours

| Séance | Document | Pour qui | Pages |
|--------|----------|----------|-------|
| 1 | [`seance-01-etudiant.pdf`](seance-01-etudiant.pdf) | à distribuer aux élèves | 8 |
| 1 | [`seance-01-prof.pdf`](seance-01-prof.pdf) | vous — déroulé minuté, notes de parole, croquis au tableau, corrigés | 10 |
| 2 | [`seance-02-etudiant.pdf`](seance-02-etudiant.pdf) | à distribuer aux élèves | 12 |
| 2 | [`seance-02-prof.pdf`](seance-02-prof.pdf) | vous — déroulé minuté, démonstrations, croquis au tableau, corrigés | 16 |
| 2 | [`exercices-seance-02.md`](exercices-seance-02.md) | à distribuer aux élèves — deux exercices, sans corrigé | — |

**Séance 1** — 2 heures : culture DevOps (module 1) et Git en équipe (module 2),
soit la partie « Comprendre » du cours embarqué dans l'application. Elle prépare
l'exigence **D-01** du projet.

**Séance 2** — 2 heures : le serveur de production (module 3) et Docker
(module 4), début de la partie « Construire la chaîne ». Elle prépare les
exigences **D-06** et **D-02**, et amène les équipes à mi-chemin du jalon **J2**
de la semaine 4. Le guide enseignant repose sur quatre démonstrations en direct
(journaux d'authentification, groupe `docker`, `nmap` depuis l'extérieur,
conteneur détruit) : un VPS jetable et un vidéoprojecteur sont indispensables.

Le document élève de la séance 2 ne contient **que le cours** : ni objectifs, ni
minutage, ni exercices. Les objectifs pédagogiques et le déroulé minuté sont dans
le guide enseignant seul — pensez à les énoncer à voix haute en ouverture.

Les exercices de la séance 2 ne sont **pas** dans le support : ils vivent dans
leur propre fiche, [`exercices-seance-02.md`](exercices-seance-02.md), à remettre
avec le PDF élève. Elle en contient deux — durcir le serveur (**D-06**) et
conteneuriser l'application (**D-02**) —, indépendants l'un de l'autre, et
**sans corrigé** : les réponses attendues et les pièges restent dans le guide
enseignant.

## Régénérer les PDF

Les sources sont dans [`source/`](source/) : un HTML par document, une feuille
de style commune, les images en PNG et les croquis de tableau en SVG.

```bash
API=../../api/.venv/bin/python      # environnement qui porte weasyprint
$API -c "
from weasyprint import HTML, CSS
docs = (('etudiant', 'seance-01-etudiant.pdf'),
        ('prof', 'seance-01-prof.pdf'),
        ('seance-02-etudiant', 'seance-02-etudiant.pdf'),
        ('seance-02-prof', 'seance-02-prof.pdf'))
for n, s in docs:
    HTML(filename='source/%s.html' % n, base_url='source').write_pdf(
        s, stylesheets=[CSS(filename='source/style.css')])"
```

> Les deux documents de la séance 1 s'appellent `etudiant.html` et `prof.html` ;
> à partir de la séance 2, les sources portent le numéro de séance.

Modifier un croquis : éditez le SVG dans `source/svg/`, puis reconvertissez-le
en PNG (`cairosvg`, largeur 1600 px, fond blanc) dans `source/img/`.

```bash
$API -c "
import cairosvg
cairosvg.svg2png(url='source/svg/06-cache-couches.svg',
                 write_to='source/img/06-cache-couches.png',
                 output_width=1600, background_color='white')"
```

⚠️ `cairosvg` place mal un `<tspan>` à l'intérieur d'un `<text>` centré, et ne
rend pas le glyphe `≈` : dans les croquis, une ligne = un `<text>`, et on écrit
« environ » en toutes lettres.
