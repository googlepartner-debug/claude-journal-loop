---
description: Second-brain — tick de journalisation horaire pour le projet courant. À utiliser via /loop 1h /second-brain. Délègue le gros du travail à un subagent pour épargner le contexte principal. Maintient un wiki dérivé (pattern karpathy) par-dessus JOURNAL.md.
---

# Second-brain — délégation subagent + wiki dérivé

L'écriture du journal pollue rapidement le contexte principal (lectures de
`JOURNAL.md`, scans de fichiers, etc.). Ce skill se contente de :

1. **Préparer** un mémo court (<400 mots) de ce qui s'est passé depuis le dernier tick : décisions, bugs résolus, fichiers livrés, points bloquants. Tu construis ce mémo à partir de la conversation en cours (que le subagent ne peut pas voir).
2. **Déléguer** au subagent `general-purpose` : (a) l'append dans `JOURNAL.md` (source brute), puis (b) la mise à jour du **wiki dérivé** à partir de ce mémo.

## Modèle en 3 couches (pattern karpathy « LLM Wiki »)

Réf : https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

```
<cwd>/
  JOURNAL.md          source brute, append daté, JAMAIS réécrite a posteriori
  wiki/               à la racine du cwd, à côté de JOURNAL.md
    index.md          carte du projet : liens vers toutes les pages sujet
    <sujet>.md        pages thématiques, interliées via [[nom-de-page]]
    _schema.md        règles de maintenance du wiki (bootstrap si absent)
```

- `JOURNAL.md` = ce qui s'est passé, quand. Chronologique, immuable.
- `wiki/` = état courant *consolidé* de la connaissance, dérivé du journal. C'est ce qu'on relit pour s'orienter. La connaissance se compose dans le temps plutôt que d'être re-dérivée à chaque fois.

## Étape 0 — Cadre (toi, dans le main)

- **cwd** : utilise le `cwd` **réel** de la session courante. Ne code aucun chemin par défaut en dur.
- **Borne temporelle** : tu ne peux pas toujours savoir quand était le dernier tick (la conversation peut avoir été compactée). Dis au subagent de lire la **date de la dernière entrée de `JOURNAL.md`** et de ne couvrir que ce qui s'est passé après.

## Étape 1 — Construire le mémo (toi, dans le main)

Regarde la conversation depuis le dernier tick journal. Liste à l'os :
- Quelles **décisions** ont été prises (et pourquoi).
- Quels **bugs** ont été résolus ou identifiés.
- Quels **fichiers** ont été créés/modifiés (chemins exacts).
- Quels **blocages** restent.
- **N'inclus PAS** : la paraphrase de tous les tool calls, les snippets de code longs, les sorties de commandes verbeuses. Juste la substance.

Si rien de notable depuis le dernier tick → ne spawn PAS de subagent, réponds juste `RAS, prochaine vérif dans 1h` en une ligne.

## Étape 2 — Spawn le subagent

Si tu as du contenu à journaliser, appelle le tool `Agent` :

- `subagent_type`: `general-purpose`
- `description`: `Append journal + update wiki`
- `prompt` : un prompt **self-contained** qui contient :
  - Le `cwd` **réel** du projet courant
  - Le mémo construit à l'étape 1
  - Les **deux tâches** ci-dessous (A puis B), le format, et les règles d'écriture
  - **Demande explicitement au subagent de répondre en <50 mots** : `entry added: <titre> | wiki: <pages touchées>` ou `RAS`

### Règle horaire & I/O fichiers (à passer au subagent)

- **Date/heure réelles** : récupère la date et l'heure courantes via Bash `date` (ne les devine pas). C'est la source du `<date FR>` et du `Xh` des entrées.
- **macOS / ~/Documents (TCC)** : Bash est bloqué pour lire/écrire des fichiers sous `~/Documents/`. Si le `cwd` est sous `~/Documents/`, fais TOUTE l'I/O fichier (`JOURNAL.md`, `wiki/`) avec les tools **Read/Write/Edit** (in-process), pas via Bash. `date` reste OK via Bash (il ne touche aucun fichier). Hors `~/Documents/`, Bash file I/O est permis.

### Tâche A — Append dans JOURNAL.md (source brute)

Lire `JOURNAL.md`, repérer la dernière entrée, puis soit :
- ajouter une **sous-section `### Xh — <focus>`** sous l'entrée du jour si elle existe déjà, soit
- créer une **nouvelle entrée datée** au format ci-dessous.

`Xh` et `<date FR>` viennent du `date` réel. Ne JAMAIS réécrire/condenser une entrée passée — append seulement.

**Archivage** : si `JOURNAL.md` couvre plus de ~2 mois, déplacer les mois révolus vers `JOURNAL-archive/<AAAA-MM>.md` et ne garder que le mois courant dans `JOURNAL.md`. Le wiki reste la couche de lecture principale ; le journal n'a pas à tout retenir.

### Tâche B — Mettre à jour le wiki dérivé

1. Si `wiki/_schema.md` est absent → le créer d'abord (voir contenu par défaut plus bas).
2. Lire `wiki/index.md` (le créer si absent) pour connaître les pages existantes.
3. Pour chaque sujet du mémo : **mettre à jour la page `wiki/<sujet>.md`** (ou la créer). On édite l'état courant du sujet, on ne re-loggue pas l'historique — l'historique vit dans `JOURNAL.md`.
4. **Interlier** : référencer les pages liées via `[[nom-de-page]]`. Ajouter toute nouvelle page à `index.md`.
5. **Lint léger (chaque tick)** : si une info du mémo contredit une page existante, corriger vers l'état le plus récent et noter la contradiction résolue en une ligne dans la réponse.
6. **Lint complet (périodique)** : si l'entrée du jour dans `JOURNAL.md` est un multiple de ~6 ticks (ou si tu repères des incohérences évidentes entre pages), faire une passe de cohérence sur tout le `wiki/` : liens morts `[[...]]`, doublons de sujets, pages contradictoires. Réparer, résumer en une ligne.

## Configuration par projet (optionnelle)

Si un fichier `.second-brain.json` existe à la racine du `cwd`, le subagent le lit et applique ses consignes **avant** d'écrire. Schéma libre, ex. :

```json
{
  "extra_scan": "Scanner ces workflows n8n via MCP avant d'écrire : <IDs>. Vérifier la table Airtable <ID> si pertinent.",
  "focus": "Bangle-Up"
}
```

C'est là que vivent les consignes spécifiques à un projet (IDs n8n, tables Airtable, etc.) — **pas en dur dans ce command**, qui doit rester générique et public.

## Format de l'entrée JOURNAL.md (à passer au subagent)

```markdown
---

# Journal — <date FR réelle>

**Focus du jour** : <1 phrase>

## Résumé exécutif

<2-4 phrases : ce qui a bougé et pourquoi ça compte>

## Ce qui a été livré

<bullet list factuelle, regroupée par sous-système si pertinent>

## Points de vigilance

<liste courte, optionnelle>

## Reste à faire

<checklist `- [ ]`>
```

## Contenu par défaut de `wiki/_schema.md` (si absent, le subagent le crée)

```markdown
# Schéma du wiki

Wiki dérivé du `JOURNAL.md` de ce projet (pattern LLM Wiki, karpathy).

## Règles
- Une page par sujet stable (sous-système, composant, décision structurante, intégration externe). Pas de page par date.
- Les pages décrivent l'**état courant**, pas l'historique. L'historique vit dans `JOURNAL.md`.
- Interlier les pages avec `[[nom-de-page]]` (slug = nom de fichier sans `.md`).
- `index.md` liste toutes les pages avec une ligne d'accroche chacune.
- En cas de contradiction avec une source plus récente : corriger vers le plus récent.

## Nommage
- Fichiers en kebab-case : `auth-supabase.md`, `pipeline-ingestion.md`.
- Titre H1 = nom lisible du sujet.
```

## Format d'une page wiki `<sujet>.md`

```markdown
# <Nom du sujet>

<État courant en quelques phrases.>

## Détails
<ce qui est vrai aujourd'hui : archi, décisions actées, contraintes>

## Liens
- [[autre-page]] — pourquoi c'est lié
```

## Règles d'écriture (à passer au subagent)

- Capture les **décisions et leurs raisons**, pas la paraphrase mécanique des diffs.
- `JOURNAL.md` : append seulement. `wiki/` : édite l'état courant.
- Pas d'écriture en dehors du `cwd` du projet courant.
- Ne lance pas un nouveau `/loop` depuis ce tick — c'est le hook `SessionStart` (ou un démarrage manuel) qui s'en occupe.

## Après le subagent

Affiche à l'user juste le retour du subagent (1 ligne max). Ne re-paraphrase
pas. Si le subagent dit `entry added: 16h tick rotation OpenAI | wiki: rotation-cles, integration-openai`,
affiche ça tel quel.
