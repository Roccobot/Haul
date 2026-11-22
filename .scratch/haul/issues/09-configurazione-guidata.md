# La configurazione guidata

Type: prototype
Status: resolved
Blocked by: 05, 06, 08

## Question

Quali passi ha la configurazione guidata al primo avvio, e come appaiono? Due strade (`Utente-03`):
partire da zero, con valori pronti e nessun termine tecnico, o importare un `config.json` esistente.
Da mostrare a Rocco in un prototipo: i passi (benvenuto, scaricamento di gallery-dl, cartella,
siti e accessi, riepilogo), che cosa si può saltare, che cosa succede a un'importazione con opzioni
che Haul non conosce, e come si riapre la guida più tardi.

## Answer

Deciso con Rocco il 2026-11-22 sul prototipo di tre varianti (le sigle sono le sue risposte).
Prototipo: `.scratch/haul/prototypes/configurazione-guidata-prototype.html` sul branch usa e getta
`prototype/configurazione-guidata`, mai su `main`, pubblicato anche come artefatto privato
([Haul, configurazione guidata](https://claude.ai/artifact/VNE4eKcqfBEA5qkpSPWHoe)).

- **Forma** (`Forma-02`): la variante A, un assistente in una finestra con la colonna dei passi a
  sinistra, come gli assistenti di macOS, e `Indietro` / `Continua` in basso. Le varianti scartate:
  la pagina unica già compilata (B) e le schede centrate con una domanda per volta (C).
- **Le due strade** (`Utente-03`), scelte nel primo passo:
  - **Parti da zero**: `Benvenuto`, `Dove salvare` (radice `~/Downloads/Haul`, `Cambia...`, con
    l'anteprima dell'albero `Radice/Sito/Proprietario/Raccolta`), `Notifiche`, `Pronto`;
  - **Usa la tua configurazione**: `Benvenuto`, `La tua configurazione` (Haul cerca da solo il
    `config.json` di gallery-dl e lo propone, con `Scegli un altro...`), `Che cosa ho trovato`,
    `Notifiche`, `Pronto`.
- **gallery-dl si prepara in sottofondo** dal primo passo, con l'avanzamento in fondo alla colonna:
  non è un passo da aspettare.
- **Niente passo degli accessi**: per `Quando-01` di
  [L'accesso ai siti che lo chiedono](08-accesso-ai-siti.md) Haul chiede l'accesso al bisogno; il
  passo `Pronto` lo dice.
- **Il resoconto dell'importazione** elenca: la radice tenuta; lo schema delle sottocartelle
  sostituito da quello di Haul (`Import-01`); gli accessi spostati nel Portachiavi, col consiglio di
  togliere le password dal file originale (`Import-02`); il numero di opzioni importate; le opzioni
  che Haul non conosce, che **restano** nella copia di Haul e gallery-dl le usa, senza un comando
  nelle impostazioni (`Sconosciute-01`). Il file originale non si tocca.
- **Notifiche** (`Notifiche-01`): un passo che si può saltare, con l'avviso a fine raccolta acceso
  di serie, così la richiesta di permesso di macOS arriva quando la si aspetta.
- **Pronto**: il riepilogo (radice, notifiche, stato di gallery-dl) e come si comincia (`⌘V` o un
  link trascinato sulla finestra); il pulsante finale è `Apri Haul`.
- **Si rifà** (`Riapri-01`) dal menu `Haul > Configurazione guidata...`, che riparte coi valori
  attuali già compilati e non cancella archivio né accessi.
- **Resta alla costruzione**: i testi definitivi con `/microcopy-it` e `/microcopy-en`, e la resa
  vera in SwiftUI col Liquid Glass di macOS 27.2.
