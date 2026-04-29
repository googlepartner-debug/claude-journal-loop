# claude-journal-loop

Plugin Claude Code qui journalise automatiquement l'avancement de chaque projet dans un `JOURNAL.md` à la racine du dépôt — sans intervention manuelle.

## Ce que ça fait

À chaque ouverture d'une session Claude Code (n'importe quel projet), un hook `SessionStart` vérifie qu'un cron horaire de journalisation tourne pour le `cwd` courant. S'il n'y en a pas, il démarre `/loop 1h /journal-loop` :

- Toutes les heures, Claude scanne les fichiers modifiés (et `git log --since="1 hour ago"` si dispo).
- S'il a quelque chose à dire, il appende une section datée dans `JOURNAL.md`.
- Sinon il répond `RAS` et attend l'heure suivante.

Un `JOURNAL.md` est créé à la racine du projet au premier tick s'il n'existe pas.

## Installation

```bash
# Ajouter la marketplace
claude plugin marketplace add googlepartner-debug/claude-journal-loop

# Installer le plugin
claude plugin install journal-loop@digitalkeys
```

Une fois installé, ouvre une nouvelle session Claude Code dans n'importe quel projet — le tick horaire s'active tout seul.

## Désinstallation

```bash
claude plugin uninstall journal-loop@digitalkeys
claude plugin marketplace remove digitalkeys
```

## Personnalisation

Le command `/journal-loop` est un simple fichier markdown (`journal-loop/commands/journal-loop.md`). Forke et modifie-le pour adapter le format des entrées, le périmètre du scan, ou ajouter des sources (n8n, Airtable, etc.) spécifiques à ton projet.

## Pour vérifier que ça tourne

Dans une session Claude Code :
- `/cron list` → vérifie qu'il y a bien une entrée `/journal-loop` programmée toutes les heures pour le cwd courant.
- `/hooks` → vérifie que le hook `SessionStart` apparaît dans la liste.
