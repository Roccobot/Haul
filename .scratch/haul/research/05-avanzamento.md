# L'avanzamento di un download in gallery-dl

> Ricerca per il ticket `issues/05-come-si-scarica.md`. Fonti lette il 2026-11-21 dal sorgente di
> `mikf/gallery-dl` al tag **`v1.32.21`** (commit `ceeb5bc6a402d3528ec58014ac379e0acfa95f1b`, lo
> stesso di `opzioni-di-gallery-dl.md`), dal mirror GitHub; lo sviluppo vive su
> [Codeberg](https://codeberg.org/mikf/gallery-dl). Ogni riga citata è all'indirizzo
> `https://github.com/mikf/gallery-dl/blob/v1.32.21/<percorso>#L<riga>`.
> ⚠️ **Nessuna prova è stata eseguita**: la prova di un download reale con l'uscita personalizzata
> è stata negata dai permessi della sessione. Tutto quello che segue è letto dal codice, e le
> righe segnate **da provare** vanno confermate con un giro vero prima di costruirci sopra.

## 📖 Due parole prima di cominciare

- **kwdict**: il dizionario dei metadati di un file (titolo, numero, estensione...), che riempie
  le stringhe di formato.
- **Evento** (o *hook*): un momento del download in cui gallery-dl chiama i postprocessori, per
  esempio `prepare` (prima), `after` (dopo), `skip` (saltato).
- **`\r`**: il ritorno a capo senza nuova riga, che riporta il cursore all'inizio della riga per
  riscriverla; è il trucco delle barre di avanzamento nel terminale.

## 1️⃣ Il totale dei file di un URL

- 🔢 **`count` e `num` esistono, ma contano un gruppo, non l'URL.**
  - `GalleryExtractor.items()` (`gallery_dl/extractor/common.py#L886-L935`) mette in `count` il
    numero dichiarato dal sito o, se manca, `len(imgs)` quando la lista ha una lunghezza; `num` è
    il numero progressivo (`enum = "num"`, riga 877). Vale per gallerie e capitoli
    (`ChapterExtractor` eredita da `GalleryExtractor`, riga 956).
  - Negli estrattori di profili e flussi `count` è per **post**: per esempio Twitter
    (`extractor/twitter.py#L140`, `tdata["count"] = len(files)`) conta i media di un tweet, non
    quelli dell'utente.
  - Delle 257 voci `.py` di `extractor/`, 98 scrivono un campo `count` e 110 un `num` o un
    `enumerate(..., 1)` (conteggio con `grep -l` sul tag). Il significato cambia da sito a sito:
    **non è un totale affidabile per l'URL**.
  - Quando un URL rimanda ad altri (un profilo che apre gallerie: `Message.Queue`,
    `job.py#L263-L272`), ogni figlio è un job a sé, col suo `count`: il totale dell'URL di
    partenza non esiste da nessuna parte.
- 🐢 **Un passaggio preliminare costa quanto l'estrazione intera.**
  - `-g` (`UrlJob`, `job.py#L988`), `-j` (`DataJob`, `job.py#L1055`) e `-s` (`SimulationJob`,
    `job.py#L882-L892`) fanno girare lo stesso estrattore; cambia solo che cosa si fa di ogni
    file (stampare, non scaricare). L'estrattore è un generatore che chiede le pagine al sito man
    mano (`dispatch`, `job.py#L210-L283`): **rifà tutte le richieste al sito e scorre tutta la
    paginazione**, con le stesse pause (`SimulationJob` rispetta anche `sleep`, riga 888).
  - Costa quindi il doppio delle richieste di metadati e raddoppia il rischio di limiti di
    frequenza (429) e blocchi; non scarica i file, che sono la parte pesante in byte.
- ❌ **Un totale certo senza un secondo giro non c'è.** Si possono avere solo: il `count` del
  gruppo corrente, quando l'estrattore lo dà, e il conteggio dei file arrivati finora. 💡 Un
  ripiego: un secondo giro *solo dopo* il primo, ma è inutile; oppure il totale del giro
  precedente dello stesso URL, conservato da Haul, come stima.

## 2️⃣ L'avanzamento in byte di un file

- ⚙️ **Chi lo misura.** Il downloader HTTP chiama `self.out.progress(totale, scaricati,
  byte_al_secondo)` a ogni blocco ricevuto, ma solo quando sono passati più di
  `downloader.*.progress` secondi dall'inizio del file (`downloader/http.py#L403-L428`; valore di
  serie `3.0`, riga 31; `null` lo spegne). Il totale viene dall'intestazione `Content-Length`, e
  manca (`None`) se il server non la manda.
- ⏱️ **Con che frequenza.** Una volta per blocco, senza altro limite: il blocco è
  `downloader.http.chunk-size`, di serie 32.768 byte (`http.py#L29`). A 10 MB/s sono circa 300
  righe al secondo; Haul deve diradarle lui, o alzare `chunk-size`. Con `progress` a `0` si ha
  l'avanzamento anche dei file piccoli (sotto i 3 secondi non compare nulla).
- 🎬 **Per ytdl.** `downloader/ytdl.py#L329-L344` aggiunge un *progress hook* a yt-dlp e passa
  `total_bytes` o, se manca, `total_bytes_estimate` (stima), con `downloaded_bytes` e `speed`. La
  frequenza la decide yt-dlp (**da provare**).
- 📤 **Dove lo scrive.** Dipende da `output.mode` (`output.py#L372-L401`):
  - in `auto`, se stdout **non** è un terminale (il caso di Haul, che legge da una pipe) vale
    `PipeOutput` (riga 418), che **non scrive l'avanzamento affatto**: eredita il `progress` vuoto
    di `NullOutput` (riga 414). Stampa solo il percorso a fine file e `# percorso` per i saltati;
  - in `terminal`/`color` scrive su **stderr** con `\r` davanti e senza a capo
    (`output.py#L447-L454`): `\r 42%   1.23MB  512.00kB/s `;
  - con `output.mode` come **oggetto** (`CustomOutput`, `output.py#L479-L535`) si scelgono i
    formati di `start`, `success`, `skip`, `progress` e `progress-total`
    (`docs/configuration.rst`, sezione `output.mode`). Il formato può finire con `\n`, e allora
    ogni aggiornamento è una riga. Ci si mette `"shorten": false`, perché altrimenti i nomi si
    accorciano sulla larghezza di un terminale che non c'è.
- 🔢 **Che numeri arrivano nel formato personalizzato.** In `progress-total`: `{0}` scaricati,
  `{1}` velocità, `{2}` totale, già resi leggibili da `util.format_value` (`util.py#L165-L173`:
  tre cifre e un suffisso, per esempio `1.23M`, quindi **arrotondati**), e `{3}` la percentuale
  intera, calcolata sui byte esatti. In `progress` (totale ignoto) solo `{0}` e `{1}`. I byte
  esatti non escono.
- 🚰 **Il flusso arriva subito?** `stderr_write` scrive e svuota il buffer se il canale non è a
  buffer di riga (`output.py#L344-L348`); in Python 3.9 e successivi stderr è a buffer di riga
  anche in una pipe, e un `\n` (o un `\r`) lo svuota (documentazione di `io.TextIOWrapper`). Le
  righe di `start` passano da `stdout_write_flush`, che svuota sempre (righe 439, 470 e 520). Per
  sicurezza Haul può lanciare il processo con `PYTHONUNBUFFERED=1`. Attenzione: su stderr passano
  anche i messaggi di log, che si distinguono col prefisso scelto nel formato.

## 3️⃣ Uno alla volta o in parallelo

- 🧵 **Uno alla volta.** `dispatch` scorre i messaggi dell'estrattore in un solo ciclo, e per ogni
  file chiama `handle_url`, che scarica, chiama i postprocessori e solo poi torna al ciclo
  (`job.py#L210-L283` e `#L418-L515`). Anche i figli (`handle_queue`) partono uno dopo l'altro,
  nello stesso processo.
- 🔎 Nel sorgente, fuori da `extractor/`, non c'è `threading`, `ThreadPool` né
  `concurrent.futures` per i download (`grep` sul tag), e fra le opzioni non c'è un numero di
  download paralleli.
- 💡 Il parallelismo, se serve, lo fa Haul: più processi di gallery-dl, uno per URL. Sullo stesso
  sito aumenta il rischio di limiti di frequenza.

## 4️⃣ Una riga per ogni file iniziato, finito o saltato

- 🪝 **Gli eventi**, nell'ordine in cui `DownloadJob.handle_url` li chiama (`job.py#L418-L515`):
  - `prepare`: nome del file deciso, prima del controllo dell'archivio e dell'esistenza;
  - `skip` (da `handle_skip`, `job.py#L646-L660`): già nell'archivio o già su disco;
  - `prepare-after`: dopo i controlli, subito prima di scaricare;
  - `error`: download fallito anche con gli URL di riserva;
  - `file`: scaricato nel file temporaneo, prima di spostarlo al posto finale;
  - `after`: file al suo posto (`pathfmt.finalize()`), dopo `out.success`.
  - Altri: `post`, `post-after`, `child`, `child-after`, `init`, `finalize`,
    `finalize-success`, `finalize-error` (elenco in `option.py#L177-L181`).
- 🖨️ **Il modo più semplice: `--Print [EVENTO:]FORMATO`** (`option.py#L421-L425`). Crea un
  postprocessore `metadata` in modo personalizzato che scrive il formato su stdout all'evento
  scelto (`option.py#L156-L207`), **e scarica lo stesso** (`--print`, minuscolo, invece spegne il
  download e l'uscita normale). Si può ripetere, una volta per evento. Esempio:
  `--Print "prepare-after:HAUL START {_path}"`, `--Print "after:HAUL DONE {_path}"`,
  `--Print "skip:HAUL SKIP {_path}"`, `--Print "error:HAUL ERR {_path}"`.
- 📁 **Il percorso.** `{_path}` è nel kwdict di serie: `metadata-path` vale `_path` se non lo si
  cambia (`job.py#L77-L85`), ed è un proxy che si stampa come il percorso del file
  (`output.py#L118-L131`). In `prepare` il percorso può non essere ancora costruito
  (`build_path` viene dopo, `job.py#L436-L437`): per l'inizio conviene `prepare-after`.
- 📏 **La dimensione.** Nessun campo del kwdict la porta. Tre strade:
  - Haul legge la dimensione dal disco quando riceve la riga `after` col percorso: la più
    semplice e certa;
  - una stringa di formato con espressione Python (`\fE`, `formatter.py#L211-L215`), che può
    usare i moduli attraverso `std` (`util.py#L545-L560` e `#L740-L757`), per esempio
    `std.os.path.getsize(...)` (**da provare**);
  - `metadata-http` mette nel kwdict le intestazioni HTTP, compresa `Content-Length`
    (`http.py#L270-L272`, `util.py#L323-L334`), ma è la dimensione annunciata e non vale per
    ytdl.
- ⚙️ **Con `exec`** (`postprocessor/exec.py#L62-L67`) si può agganciare un comando a qualunque
  evento (`"event": "prepare-after,after,skip"`), ma lancia un processo per file: per scrivere
  una riga `--Print` è più leggero.

## 🧭 Conseguenze per Haul

- ✅ **Percentuale per file: sì**, con `output.mode` personalizzato, `downloader.*.progress` a `0`
  e formati che finiscono con `\n` e un prefisso riconoscibile; vale per HTTP e per ytdl. Quando
  il server non dà la dimensione si mostra solo quanto è arrivato e la velocità, senza
  percentuale. I byte sono arrotondati a tre cifre, la percentuale è esatta. Haul dirada gli
  aggiornamenti (uno ogni 100-250 ms basta all'occhio).
- ✅ **Contatore dei file: sì**, con righe `--Print` su `prepare-after`, `after`, `skip`, `error`:
  file in corso, finiti, saltati e falliti, ognuno col percorso; la dimensione la legge Haul dal
  disco a `after`.
- ⚠️ **Totale dei file dell'URL: non certo.** Si può mostrare:
  - il `count` del gruppo corrente (galleria, capitolo, post) quando l'estrattore lo dà, come
    "file 3 di 12 di questa galleria";
  - per i profili e i flussi un contatore aperto ("23 file finora"), senza barra complessiva;
  - come stima, il totale dell'ultimo giro dello stesso URL, dichiarato come stima.
  Un passaggio preliminare con `-s` o `-j` darebbe il totale, ma rifà tutte le richieste e scorre
  tutte le pagine: si sconsiglia come comportamento di serie.
- 🧵 **Un file alla volta per URL**: la barra di un URL è una sola, e il parallelismo, se lo si
  vuole, è fra URL diversi, con un processo ciascuno.

## 📚 Fonti

- `gallery_dl/output.py`, righe 118-131, 333-348, 372-401, 414-454, 479-535.
- `gallery_dl/downloader/http.py`, righe 29-31, 85-88, 270-272, 345, 396-428.
- `gallery_dl/downloader/ytdl.py`, righe 38, 329-344.
- `gallery_dl/job.py`, righe 77-85, 210-283, 296-310, 418-515, 646-660, 850-860, 882-892, 988,
  1055.
- `gallery_dl/option.py`, righe 156-207, 391-437, 971-990.
- `gallery_dl/postprocessor/exec.py`, righe 62-67; `postprocessor/metadata.py`, righe 20-60.
- `gallery_dl/extractor/common.py`, righe 871-956; `gallery_dl/extractor/twitter.py`, riga 140.
- `gallery_dl/util.py`, righe 165-173, 323-334, 545-560, 740-757; `gallery_dl/formatter.py`,
  righe 211-222, 600-604.
- `docs/configuration.rst`, sezioni `downloader.*.progress`, `output.mode`, `output.progress`.
- Documentazione di Python, `io.TextIOWrapper` (`line_buffering`) e `sys.stderr` (buffer di
  riga dalla 3.9).
