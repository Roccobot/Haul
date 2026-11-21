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
  - le versioni stabili di gallery-dl (`mikf/gallery-dl`, ultima 1.32.20) escono senza eseguibili;
    un `gallery-dl_macos` pronto si trova solo nelle build giornaliere di `gdl-org/builds`, e
    l'alternativa è la versione stabile da PyPI con un Python incluso;
  - `configuration.rst` è lungo circa 245 KB, con centinaia di opzioni: la finestra delle
    impostazioni conviene generarla da lì, curando a mano solo le voci principali.

## 🌿 Ramo e versione

- **Ramo principale `main`.**
- **La versione non esiste ancora**: si sceglie con la prima build, con una fonte sola, e si
  scrive qui e nel nucleo di `AGENTS.md`.
