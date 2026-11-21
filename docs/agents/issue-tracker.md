# Issue tracker: file Markdown

Le mappe, le specifiche e i ticket delle skill di Matt Pocock (`/wayfinder` e quelle che chiama)
vivono come file Markdown in `.scratch/` di questo repo (scelta `Tracker-01`, `rules/Roccobot.md`
§ '🧭 Un progetto nuovo grande o complesso: si propone `/wayfinder`'). Rocco non legge né scrive
questi file: si lavora in chat, e la sessione scrive qui risposte e decisioni. I lavori restano
sulla bacheca Trello `Roccobot`, nella lista `Haul`, e la card del lavoro cita la mappa.

## Convenzioni

- Un lavoro per cartella: `.scratch/<lavoro>/`
- La specifica è `.scratch/<lavoro>/spec.md`
- I ticket sono un file ciascuno, `.scratch/<lavoro>/issues/<NN>-<nome>.md`, numerati da `01`, mai
  un file solo con tutti i ticket
- Lo stato è una riga `Status:` in testa a ogni ticket
- Commenti e storia della conversazione si aggiungono in fondo al file, sotto un titolo
  `## Comments`

## Quando una skill dice 'pubblica nel tracker'

Si crea un file nuovo in `.scratch/<lavoro>/`, creando la cartella se manca.

## Quando una skill dice 'leggi il ticket'

Si legge il file al percorso indicato; di solito la sessione ha già il percorso o il numero.

## Wayfinding operations

Usate da `/wayfinder`. La **mappa** è un file, con un file **figlio** per ogni ticket.

- **Mappa**: `.scratch/<lavoro>/map.md` (il corpo con Notes, Decisions so far e Fog).
- **Ticket figlio**: `.scratch/<lavoro>/issues/NN-<nome>.md`, numerato da `01`, con la domanda nel
  corpo. La riga `Type:` dice il tipo (`research`, `prototype`, `grilling`, `task`); la riga
  `Status:` dice `claimed` o `resolved`.
- **Blocchi**: una riga `Blocked by: NN, NN` in testa. Un ticket è sbloccato quando ogni file che
  elenca è `resolved`.
- **Frontiera**: i file di `.scratch/<lavoro>/issues/` aperti, sbloccati e non presi; vince il
  numero più basso.
- **Presa**: `Status: claimed`, salvata prima di ogni altro lavoro.
- **Soluzione**: la risposta sotto un titolo `## Answer`, poi `Status: resolved`, poi una riga nel
  Decisions so far di `map.md`, col succo e il collegamento al ticket.
