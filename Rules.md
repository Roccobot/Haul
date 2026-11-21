# Rules.md: regole del progetto Haul (repo `Roccobot/Haul`)

> **Cos'è questo file.** Il testo completo delle regole di **Haul**. Vale per **tutti gli
> agenti**: il nucleo, cioè ogni regola in una riga, vive in `AGENTS.md`, e questo file ne dà il
> perché. Nessun agente lo carica da sé: si legge per intero prima di lavorare su una cosa di cui
> parla. Le regole trasversali vivono nell'hub `Roccobot/roccobot.github.io` (`AGENTS.md` e
> `Rules.md`), quelle universali in `rules/Roccobot.md` di `Roccobot/tools`.

## 🧭 Che cos'è

- **Un'app per macOS che rende gallery-dl usabile da chiunque**, anche da chi non l'ha mai
  configurato. Le richieste dell'utente (2026-11-21):
  - una configurazione guidata al primo avvio;
  - i parametri di [`configuration.rst`](https://github.com/mikf/gallery-dl/blob/master/docs/configuration.rst)
    e il JSON da compilare a mano trasformati in una finestra di impostazioni;
  - la cartella di download e tutte le sottocartelle stabilite dall'app;
  - criteri di rinomina applicati da soli a certi tipi di download;
  - piena aderenza al design di macOS 27.2 e successivi, icona in vero Liquid Glass compresa;
  - l'eseguibile di gallery-dl scaricato e aggiornato dall'app;
  - un'app piacevole, moderna e accattivante.
- **Le scelte si prendono nella mappa di `/wayfinder`**, scelta `Mappa-01` dell'utente del
  2026-11-21: la mappa vive come file Markdown in questo repo.
- **Fatti già accertati che la mappa deve considerare** (2026-11-21):
  - il Liquid Glass vero esiste solo in SwiftUI e AppKit: un'interfaccia scritta in Rust lo può
    solo imitare, e le regole chiedono comunque l'interfaccia nativa;
  - lo sviluppo di gallery-dl è passato da GitHub a Codeberg ad aprile 2026, dopo una segnalazione
    DMCA; le versioni stabili (ultima 1.32.21, del 2026-11-21) hanno eseguibili solo per Windows e
    Linux, e un `gallery-dl_macos` si trova solo nelle build giornaliere di `gdl-org/builds`, senza
    una firma vera; la ricerca della mappa propone un Python incluso nell'app e gallery-dl da PyPI
    (ticket 01 in `.scratch/haul/issues/`, corretto il 2026-11-21 dopo la ricerca);
  - `configuration.rst` è lungo circa 245 KB, con centinaia di opzioni: la finestra delle
    impostazioni conviene generarla da lì, curando a mano solo le voci principali.

## 🌿 Ramo e versione

- **Ramo principale `main`.**
- **SemVer da 0.1.0** (scelta `Versione-01` di Rocco, 2026-11-21): versioni 0.x durante la
  costruzione e 1.0.0 alla prima versione pubblica, con una fonte sola nel progetto Xcode, letta
  anche dal nucleo, e il numero di build separato, che è quello che confrontano gli aggiornamenti.
  Il file preciso si sceglie con la prima build, e si scrive qui e nel nucleo di `AGENTS.md`.

## 🗺️ Le scelte della mappa

- **La mappa vive in `.scratch/haul/map.md`**, coi ticket in `.scratch/haul/issues/`; le scelte
  dell'intervista di partenza (canale, prezzo e licenza, lingue, tema e accento, versione) sono
  nelle sue Notes, e quelle difficili da rovesciare in `docs/adr/`.
- **Interfaccia in italiano e inglese**, che segue la lingua del sistema (`Lingue-01`); **tema e
  colore d'accento di sistema** (`Tema-01`, `Accento-01`).
- **Distribuzione diretta fuori dal Mac App Store**, gratuita e open source con licenza GPL-3.0
  (`Canale-01`, `Prezzo-01`, `Licenza-01`).
