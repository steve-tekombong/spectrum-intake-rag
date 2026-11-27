# spectrum-intake-rag

Dépôt de travail de l'équipe **RAG + Intake** (Steve et Charles) pour le projet **Spectrum**, hackathon Adopt AI 2026, défi 2 « Répondre à une campagne en cours ».

Ce README est pour nous deux et pour nos assistants IA (le Claude de Charles le lira en premier). Il explique la mission, qui fait quoi et comment on travaille ensemble. Le code finira dans le dépôt d'équipe [`marceaupatu123/spectrum-adopt-ai-2026`](https://github.com/marceaupatu123/spectrum-adopt-ai-2026) : on garde donc la même arborescence pour que le merge soit un simple copier-coller.

---

## 1. Spectrum en bref

Une fois qu'une campagne de désinformation a été **détectée et caractérisée** (par une équipe en amont, pas par nous), Spectrum aide un service public ou une rédaction à décider **s'il faut répondre, et comment**, sans :

- **amplifier** le faux récit (effet Streisand) ;
- **braquer** l'audience par un ton institutionnel ou offensif (effet boomerang).

Le défi demande une réponse **rapide**, avec des **preuves vérifiables et partageables**. Notre équipe cible le **générateur de contre-narration adapté à chaque plateforme**.

Chaîne complète :

```
Dossier (détection) -> Claims -> Preuves (RAG) + Écoute -> Brouillons par canal -> Scoring -> Front
                       \_____ nous _____/
```

**Règle d'or : l'outil est consultatif, il ne publie jamais rien lui-même.** L'A/B test est une *recommandation* faite à l'agent public, qui le mène lui-même avec les contre-narrations générées. L'analyste garde toujours la décision finale.

Cas de démo (fictif) : exercice **ORION**, dans `demo_case/` du dépôt d'équipe.

## 2. Les équipes

| Équipe | Qui | Rôle |
|---|---|---|
| **RAG + Intake** | **Steve, Charles** | Extraire les affirmations du dossier, trouver les contre-preuves sourcées |
| Écoute sémantique + coordination | Marceau | Profil d'audience (codes, griefs, sources de confiance), contrats, orchestrateur |
| Génération + Scoring | Rayan, Karel | Brouillons de riposte par canal, score (preuves, ton, risque d'amplification) |
| Front | Maeva, MH | Interface de démo, faux input, slides |

**Marceau est le seul à modifier les contrats** (`contracts/`, `fixtures/`, `shared/`, `orchestrator/`). S'il nous manque un champ, on s'arrête et on lui demande (champ voulu, raison, modules touchés).

## 3. Notre mission : Intake + RAG

### Intake (entrée)

- **Entrée** : le dossier déjà caractérisé fourni par la détection (le récit, les messages, la communauté visée, les canaux).
- **Travail** : en extraire les **claims**, c'est-à-dire les affirmations factuelles vérifiables. Une claim = une phrase courte, testable, sans le ton du message d'origine.
- On ne refait pas la détection : on part du dossier tel quel.

### RAG (preuves)

- Pour chaque claim, chercher les **contre-arguments** dans un corpus de livres, articles de presse et sources officielles.
- **Chaque affirmation de la sortie cite une source archivée et horodatée** (URL, lien d'archive, date). Pas de source, pas d'affirmation.
- **Sortie** : un **dossier de preuves vérifiable**, consommé par Génération/Scoring (pour écrire et noter les ripostes) et par le Front (page « sources »).

Les formats exacts d'entrée et de sortie sont définis par les contrats de Marceau (voir section 5). Tant qu'on ne les a pas copiés ici, tout ce qui touche aux noms de champs est une **hypothèse**.

## 4. Qui fait quoi

### Ensemble (Steve + Charles)

1. **Lire les contrats** : `docs/contracts.md`, `docs/modules/intake_rag.md`, `fixtures/intake_rag/` (Steve les copie depuis le dépôt d'équipe).
2. **Étape recherche + réponse** : pour une claim, récupérer les passages pertinents et formuler la contre-preuve avec ses citations.
3. **Test de bout en bout** qui remplace le mock : dossier ORION en entrée, dossier de preuves valide en sortie.

### Steve

- **Intake** : extraction des claims depuis le dossier.
- **Sortie** : construction du dossier de preuves au format du contrat (claims, contre-preuves, citations).

### Charles

- **Corpus de démo** dans `demo_case/preuves/` : sources datées, avec URL d'origine et lien d'archive (Wayback Machine, archive.today…), en lien avec le cas ORION.
- **Indexation / embeddings** : découpage, vectorisation, recherche.

### Bonus (si on a le temps)

- **Analyse de l'audience cible et du canal** : par exemple, une campagne relayée par la presse et touchant les 40-60 ans appelle une réponse sur Facebook. **À caler avec Marceau**, car ça touche à son profilage d'audience et aux contrats.

## 5. Arborescence (à garder telle quelle)

```
modules/intake_rag/core.py     # notre code (fonction déjà typée côté dépôt d'équipe)
demo_case/preuves/             # corpus de démo (Charles)
docs/contracts.md              # formats de données (généré, lecture seule)
docs/modules/intake_rag.md     # notre fiche module (lecture seule)
fixtures/intake_rag/           # exemples d'entrée/sortie (lecture seule)
tests/modules/test_intake_rag.py
```

Les fichiers `docs/`, `fixtures/` et `contracts/` viennent du dépôt d'équipe : **on ne les modifie pas ici**, sinon le merge cassera.

Dans le dépôt d'équipe, notre module tourne en mock par défaut. Pour tester le vrai code (hypothèse sur le nom de la variable, à vérifier dans le Makefile) :

```bash
MOCK_INTAKE_RAG=false python -m pytest tests/modules/test_intake_rag.py
```

## 6. Règles de collaboration

- **Une branche par personne** : `steve/intake`, `charles/corpus`, etc. Jamais de push direct sur `main`.
- **Petites PR**, relues par l'autre. Code + test + doc dans la même PR.
- **Merge fréquent** (toutes les ~2 h) pour ne pas diverger.
- **Marquer les hypothèses** : tout ce qui n'est pas confirmé par un contrat ou par l'équipe porte un commentaire `# HYPOTHÈSE: ...` dans le code, et une ligne dans la PR.
- **Pas de source inventée** : chaque preuve du corpus a une URL réelle, une date et un lien d'archive.
- **Tout en français** (données, prompts, docs).
- Besoin d'un changement de contrat : on demande à Marceau, on ne le fait pas nous-mêmes.

## 7. Pour les assistants IA

Si tu es l'assistant de Steve ou de Charles :

1. Lis ce README, puis les contrats (section 5) avant d'écrire du code.
2. N'écris que dans `modules/intake_rag/`, `demo_case/preuves/` et `tests/`.
3. Ne modifie jamais `contracts/`, `fixtures/`, `shared/`, `orchestrator/`, `docs/contracts.md`.
4. Signale toute hypothèse explicitement, dans le code et dans ta réponse.
5. Respecte la répartition de la section 4 : demande avant de toucher à la partie de l'autre.

## 8. Ressources

- Fiche mission de l'équipe : « Idée du Projet » (PDF) et énoncé « Challenge 2 », dans le dossier Google Drive « Spectrum ».
- Datasets du hackathon : page Ressources du challenge sur Agorize.
