# second-brain

Plugin Claude Code qui journalise automatiquement l'avancement de chaque projet **et** en maintient un wiki dérivé interlié — sans intervention manuelle.

Inspiré du pattern « LLM Wiki » d'Andrej Karpathy : https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Modèle en 3 couches

```
<cwd>/
  JOURNAL.md          source brute, append daté, immuable
  wiki/
    index.md          carte du projet : liens vers toutes les pages sujet
    <sujet>.md        pages thématiques interliées via [[nom-de-page]]
    _schema.md        règles de maintenance du wiki (bootstrap auto)
```

- `JOURNAL.md` = ce qui s'est passé, quand. Chronologique.
- `wiki/` = état courant consolidé, dérivé du journal. C'est ce qu'on relit pour s'orienter — plus facile à requêter qu'un log linéaire. La connaissance se compose dans le temps au lieu d'être re-dérivée à chaque fois.

## Ce que ça fait

À chaque ouverture d'une session Claude Code (n'importe quel projet), un hook `SessionStart` vérifie qu'un cron horaire tourne pour le `cwd` courant. S'il n'y en a pas, il démarre `/loop 1h /second-brain`. À chaque tick, le main construit un mémo court de ce qui a bougé, puis délègue à un subagent :

1. **A** — append d'une entrée datée dans `JOURNAL.md` (source brute, jamais condensée ; archivage mensuel au-delà de ~2 mois).
2. **B** — mise à jour du wiki dérivé : pages sujet (état courant), `[[liens]]`, `index.md`, lint léger chaque tick + lint complet périodique.
3. **C** — *(hebdomadaire, gâté)* publication des nouvelles **fonctionnalités visibles utilisateur** (≤3 par projet) dans un changelog commun privé.

Rien à dire depuis le dernier tick → `RAS`, pas de subagent.

## Changelog commun

Une fois par semaine et par projet, le tick dérive ≤3 features user-visibles du `JOURNAL.md` (exclut fixes infra, refactos, debug, ops) et les pousse dans un repo privé partagé `changelog` : `CHANGELOG.md` (humain) + `changelog.json` (machine-lisible, interrogeable par une app via l'API GitHub). Section `🆕 Récentes` (12 dernières) puis `📦 Archives`. La cadence hebdo est gâtée par `last_sync` par projet — pas de cron séparé, ça ride le tick horaire existant.

## Installation

```bash
claude plugin marketplace add googlepartner-debug/claude-journal-loop
claude plugin install second-brain@digitalkeys
```

Ouvre une nouvelle session Claude Code dans n'importe quel projet — le tick horaire s'active tout seul.

## Désinstallation

```bash
claude plugin uninstall second-brain@digitalkeys
claude plugin marketplace remove digitalkeys
```

## Configuration par projet (optionnelle)

Place un `.second-brain.json` à la racine d'un projet pour ajouter des consignes de scan spécifiques (sources n8n, tables Airtable, focus métier) sans toucher au plugin :

```json
{
  "extra_scan": "Scanner ces workflows n8n via MCP avant d'écrire : <IDs>. Vérifier la table Airtable <ID> si pertinent.",
  "focus": "<nom du focus>"
}
```

## Personnalisation

Le command `/second-brain` est un simple fichier markdown (`second-brain/commands/second-brain.md`). Forke et modifie-le pour adapter le format des entrées, le périmètre du scan ou la structure du wiki.

## Pour vérifier que ça tourne

- `/cron list` → une entrée `/second-brain` programmée toutes les heures pour le cwd courant.
- `/hooks` → le hook `SessionStart` apparaît dans la liste.
