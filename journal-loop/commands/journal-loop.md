---
description: Tick de journalisation horaire pour le projet courant. À utiliser via /loop 1h /journal-loop.
---

Documente l'avancement du projet **dans lequel cette session est ouverte** (cwd).

## Étapes à chaque tick

1. **Détermine la racine projet** : c'est le `cwd` de la session. Toutes les écritures se font dans ce dossier — jamais ailleurs.

2. **Lis `JOURNAL.md` à la racine du projet** pour repérer la dernière entrée et le contexte connu.
   - S'il n'existe pas encore : crée-le avec un en-tête minimal `# Journal — <nom du projet>` puis continue.

3. **Scanne ce qui a bougé depuis ~1h** :
   - Fichiers du dossier projet : `find . -type f -mtime -1 -not -path './.git/*' -not -path './node_modules/*'` (ou comparaison avec timestamps de la dernière entrée)
   - Si le projet a un git : `git log --since="1 hour ago" --oneline` pour repérer les commits récents
   - Regarde aussi la conversation en cours pour détecter les décisions/essais notables : déploiements, bugs résolus, choix d'architecture, blocages.

4. **Décide** :
   - Si rien de significatif depuis la dernière entrée → ne rien écrire, juste répondre `RAS, prochaine vérif dans 1h` en une ligne.
   - Sinon → appende une nouvelle section datée dans `JOURNAL.md` (**fichier unique à la racine du projet**, pas de variante par date).

## Format de la nouvelle section

```markdown
---

# Journal — <date FR>

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

## Règles d'écriture

- Capture les **décisions et leurs raisons**, pas la paraphrase mécanique des diffs.
- Si la même journée a déjà une entrée et qu'on ajoute du contenu : sous-section `### <heure>h — <focus court>` plutôt qu'une nouvelle entrée datée.
- Pas d'écriture en dehors du `cwd` du projet courant.
- Ne lance pas un nouveau `/loop` depuis ce tick — c'est le hook `SessionStart` (ou un démarrage manuel) qui s'en occupe.
