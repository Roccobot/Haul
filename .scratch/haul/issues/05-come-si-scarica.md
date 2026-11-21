# Come si avvia e si segue un download

Type: grilling
Status: resolved
Blocked by:

## Question

Qual è il gesto principale di Haul, e che cosa vede chi scarica? Da decidere con Rocco:
- da dove arriva un indirizzo: campo nella finestra, incolla, trascinamento di un link, un'estensione
  di Safari o il menu Condividi, gli appunti letti all'apertura;
- uno per volta o una coda; scaricamenti in parallelo; pausa, ripresa, annullamento;
- che cosa si vede durante il download (avanzamento, anteprime, numero di file) e alla fine
  (mostra nel Finder, notifica);
- la cronologia: se esiste e che cosa ricorda;
- quali siti usa Rocco davvero, come esempi per tutto il resto della mappa.

## Answer

Deciso con Rocco il 2026-11-21 (intervista a scelta multipla, le sigle sono le sue risposte).
Fatti dal sorgente di gallery-dl 1.32.21: [05-avanzamento.md](../research/05-avanzamento.md).

- **Raccolta** (`GLOSSARY.md`): un indirizzo dato a Haul con tutti i file che ne vengono, in una
  cartella sola; è l'unità della coda. Il nome è di Rocco; la cartella la decide il ticket
  [Cartelle e sottocartelle dei download](06-cartelle.md).
- **Ingresso** (`Ingresso-01`, `-02`, `-03`, `-04`, tutte): un campo nella finestra con `⌘V` che
  incolla e avvia, anche più indirizzi uno per riga; il trascinamento di un link da qualunque app,
  sulla finestra o sull'icona nel Dock; Haul nel menu Condividi di sistema (un'estensione a parte);
  l'indirizzo negli appunti proposto quando si torna su Haul, se gallery-dl lo riconosce.
- **Coda** (`Coda-01`): ogni raccolta entra in coda, ne partono al massimo due insieme, numero
  regolabile. Dentro una raccolta gallery-dl scarica un file alla volta, e non si cambia.
- **Comandi** (`Controlli-01`): `Ferma` interrompe gallery-dl (`SIGINT`), `Riprendi` lo rilancia
  e salta quello che c'è già, riprendendo i `.part`; `Annulla` toglie la raccolta dalla coda.
- **Avanzamento**, in stile download manager (risposta libera di Rocco e `Totale-01`): la riga della
  raccolta, con annidato il file in corso e la sua percentuale; i file finiti si contano man mano;
  'N di M' e la percentuale della raccolta compaiono solo quando il totale è noto (il campo `count`
  di una galleria), altrimenti resta il contatore. Nessun giro di conteggio preliminare, che rifà
  tutte le richieste al sito. Il ritmo dell'avanzamento si regola con `downloader.*.progress`;
  i contatori di finiti, saltati e in errore vengono da `--Print` sugli eventi `after`, `skip`,
  `error`.
- **Alla fine** (`Fine-01`, `Fine-02`, suono): una notifica di sistema col numero dei file, solo se
  Haul non è in primo piano, che apre la cartella; `Mostra nel Finder` sulla riga; un suono a fine
  coda, spento di serie e attivabile.
- **Due schede** (correzione di Rocco al riepilogo): `Download in corso` e `Raccolte scaricate`
  (etichette da rifinire con `/microcopy-it` e `/microcopy-en`). Una raccolta finita passa nella
  seconda scheda (indirizzo, data, numero di file, cartella, esito) e ci resta 7 giorni di serie,
  da 1 a 30 a scelta nelle impostazioni (`Durata-01`), poi è tolta; `Scarica di nuovo` la rilancia
  e prende solo i file nuovi (`Rilancio-01`).
- **Archivio dei file già scaricati** (`Archivio-01`): è l'archivio SQLite di gallery-dl
  (`--download-archive`), sul Mac; Haul lo usa e non tiene una cronologia propria dei file.
  `Esporta` salva impostazioni e archivio in un file, `Importa` su un altro Mac unisce i due
  archivi senza perdere voci. L'archivio in iCloud Drive è scartato: iCloud copia il file intero,
  e due Mac che scaricano insieme si sovrascrivono.
- **Siti di riferimento per il resto della mappa**: Bunkr, Pixiv, DeviantArt, Reddit, Tumblr,
  NSFWAlbum, ExHentai, Coomer, Kemono, Patreon, RedGifs, Boosty, Facebook, Newgrounds, Instagram,
  X.
