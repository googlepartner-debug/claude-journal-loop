---
description: Tick de journalisation horaire pour le projet courant. À utiliser via /loop 1h /journal-loop. Délègue le gros du travail à un subagent pour épargner le contexte principal.
---

# Journal-loop — délégation subagent

L'écriture du journal pollue rapidement le contexte principal (lectures de
`JOURNAL.md`, scans de fichiers, etc.). Ce skill se contente de :

1. **Préparer** un mémo court (<400 mots) de ce qui s'est passé depuis le dernier tick : décisions, bugs résolus, fichiers livrés, points bloquants. Tu construis ce mémo à partir de la conversation en cours (que le subagent ne peut pas voir).
2. **Déléguer** l'écriture effective du journal à un subagent `general-purpose`, qui fait toutes les lectures/écritures du fichier sans encombrer le main.

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
- `description`: `Append journal entry`
- `prompt` : un prompt **self-contained** qui contient :
  - Le `cwd` du projet courant (`/Users/dk/Documents/Github/dkkrypte-unlocked` par défaut sauf si tu sais que ça a changé)
  - Le mémo construit à l'étape 1
  - L'instruction d'aller lire `JOURNAL.md`, repérer la dernière entrée, et soit ajouter une **sous-section `### Xh — <focus>`** sous l'entrée du jour si elle existe déjà, soit créer une **nouvelle entrée datée** avec le format ci-dessous
  - Le format à respecter (voir ci-dessous)
  - Les règles d'écriture (voir ci-dessous)
  - **Demande explicitement au subagent de répondre en <50 mots** : juste "entry added: <titre court>" ou "RAS"

## Cas spécial Content Factory

Si `cwd` = `/Users/dk/Documents/Projets/Content Factory`, le subagent doit aussi
scanner les workflows n8n via MCP avant d'écrire — focus Bangle-Up :
- `BUBUYQk31PxBZZO6` — [Workflow A] Sophie Master Generator
- `Ic5QkHdt8SlKHukf` — [Workflow B1] Sophie Set Generator INIT
- `Wv0bsocQuP8B9bHJ` — [Workflow B2] Sophie Set Generator GEN Single Master
- `3z7QrYoS3CGq8PyI` — [Workflow C] Ad Creative Generator
- `P7JFPTrxCPiW2HA2` — [Workflow D] Bracelet Stack Composer

Et la table Airtable `Ad_Visuals` (`tbllBMJXDUID9csZi`) si pertinent. Mentionne
cette consigne explicitement dans le prompt du subagent quand applicable.

## Format de la nouvelle section (à passer au subagent)

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

## Règles d'écriture (à passer au subagent)

- Capture les **décisions et leurs raisons**, pas la paraphrase mécanique des diffs.
- Si la même journée a déjà une entrée et qu'on ajoute du contenu : **sous-section `### <heure>h — <focus court>`** plutôt qu'une nouvelle entrée datée.
- Pas d'écriture en dehors du `cwd` du projet courant.
- Ne lance pas un nouveau `/loop` depuis ce tick — c'est le hook `SessionStart` (ou un démarrage manuel) qui s'en occupe.

## Après le subagent

Affiche à l'user juste le retour du subagent (1 ligne max). Ne re-paraphrase
pas. Si le subagent dit "entry added: 16h tick rotation OpenAI clarifiée",
affiche ça tel quel.
