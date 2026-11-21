# Le opzioni di gallery-dl e il modo di guidarlo

> Ricerca per il ticket `issues/03-opzioni-di-gallery-dl.md`. Fonti lette il 2026-11-21 dal
> sorgente di `mikf/gallery-dl` al tag **`v1.32.21`** (commit
> `ceeb5bc6a402d3528ec58014ac379e0acfa95f1b`). Ogni file citato è stato letto a quel tag, con
> l'indirizzo `https://github.com/mikf/gallery-dl/blob/v1.32.21/<percorso>`. Quello che non è
> stato possibile verificare è segnato come **non verificato**.

## 📖 Due parole prima di cominciare

- **Opzione**: una voce di configurazione di gallery-dl, per esempio `base-directory` (la cartella
  in cui scarica). Si scrive in un file JSON, il `config.json`.
- **Estrattore**: il pezzo di gallery-dl che sa leggere un sito. Ognuno ha una **categoria** (il
  sito, per esempio `pixiv`) e una **sottocategoria** (il tipo di pagina, per esempio `user`).
- **reStructuredText** (`.rst`): un formato di testo con titoli sottolineati da trattini o segni
  di uguale; è il formato di `docs/configuration.rst`.
- **Stringa di formato**: un modello di testo con segnaposti fra graffe, come `{title}.{extension}`,
  che gallery-dl riempie con i dati di ogni file.
- **stdout** e **stderr**: i due canali di testo in uscita di un programma a riga di comando; il
  primo per i risultati, il secondo per i messaggi e gli errori.
- **Codice d'uscita**: il numero che un programma restituisce quando finisce; `0` vuol dire tutto
  bene.

## 🏷️ La versione letta

- L'ultima versione stabile **non è 1.32.20 ma 1.32.21**, pubblicata oggi, il 2026-11-21 alle
  11:04 ora di Roma (`2026-11-21T10:04:54Z`), secondo l'API delle release di GitHub
  (`https://api.github.com/repos/mikf/gallery-dl/releases/latest`, letta il 2026-11-21). La
  1.32.20 è del 2026-11-14.
- La stessa versione è scritta in `gallery_dl/version.py` (`__version__ = "1.32.21"`).
- Al momento della lettura la release 1.32.21 **non aveva ancora file allegati** (l'elenco
  `assets` dell'API era vuoto): probabilmente sono caricati dopo la pubblicazione, **non
  verificato**. Riguarda il ticket `01-fonte-di-gallery-dl.md`.

## 📄 Com'è fatto `docs/configuration.rst`

Fonte: [`docs/configuration.rst`](https://github.com/mikf/gallery-dl/blob/v1.32.21/docs/configuration.rst),
**245.137 byte**.

**Come ho contato.** Ho scaricato il file e l'ho letto con uno script Python mio (lanciato con
`python3 -I`, senza eseguire codice di gallery-dl): un titolo è una riga a inizio riga seguita da
una riga fatta tutta dello stesso segno, lunga almeno quanto il titolo. I numeri qui sotto sono
l'uscita dello script. Le opzioni da riga di comando le ho contate leggendo `gallery_dl/option.py`
con il modulo `ast` di Python, che analizza la struttura del codice senza eseguirlo.

### Le sezioni

Il file ha un titolo principale, un indice (`Contents`) e **8 sezioni**:

| Sezione | Titoli di voce | Chiavi singole |
|---|---|---|
| `Extractor Options` (generali per ogni sito) | 81 | 84 |
| `Extractor-specific Options` (di un sito) | 408 | 416 |
| `Downloader Options` (lo scaricamento) | 28 | 29 |
| `Output Options` (i messaggi) | 15 | 17 |
| `Postprocessor Options` (le azioni dopo lo scaricamento) | 78 | 78 |
| `Miscellaneous Options` (varie) | 18 | 18 |
| `API Tokens & IDs` (chiavi dei servizi) | 6 | 12 |
| `Custom Types` (i tipi di valore, non opzioni) | 12 | esclusi |
| **Totale** | **646** | **654** |

- **Titoli di voce**: le righe sottolineate da trattini, **646**. Alcuni titoli raccolgono più
  chiavi, come `extractor.*.username & .password`: sono **19** titoli con `&`.
- **Chiavi singole**: i titoli con `&` divisi nelle loro chiavi, esclusi i 12 tipi di
  `Custom Types`: **654**.
- **Blocchi**: in 3 casi due titoli sono impilati sopra lo stesso testo (per esempio
  `extractor.bluesky.post.depth` e `extractor.bluesky.likes.depth`), quindi i blocchi di testo
  sono **643**.

### I campi di ogni voce

Ogni voce è una lista di etichette a inizio riga, con il contenuto rientrato di quattro spazi
sotto. Le etichette contate sui 646 titoli:

| Etichetta | Voci | Che cosa contiene |
|---|---|---|
| `Type` | 643 | il tipo o i tipi di valore |
| `Description` | 636 | la spiegazione |
| `Default` | 538 | il valore di serie |
| `Example` | 151 | uno o più esempi |
| `Note` | 106 | avvertenze |
| `Supported Values` | 28 | i valori ammessi, con spiegazione |
| `Special Values` | 9 | valori con un significato speciale |
| `How To` | 6 | come ottenere una chiave (solo in `API Tokens & IDs`) |

- La sequenza più comune è `Type`, `Default`, `Description`: **362** voci. Le sequenze diverse
  sono **52**.
- **Il valore di serie** (sui 643 blocchi): **494** sono un solo valore letterale fra doppi
  backtick (per esempio ``` ``true`` ```), facile da leggere per un programma; **105** non ce
  l'hanno; **39** sono prosa o tabelle (per esempio `extractor.*.browser`, che vale `"firefox"`
  per sei siti e `null` per gli altri); **5** sono un blocco di codice JSON.
- **Il tipo**: **57** scritture diverse, e **150** blocchi ammettono più di un tipo. Le più
  frequenti: `bool` 288, `string` 257, `list` of `strings` 84, `integer` 37, `Path` 17,
  `Format String` 12, `object` (nome → valore) 11.

### Quanto è regolare per un generatore

È **abbastanza regolare** da essere letto da un programma, con poche eccezioni da gestire a mano:

- Lo stesso autore lo legge con espressioni regolari: lo script
  [`scripts/docs_compare.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/scripts/docs_compare.py)
  cerca i titoli con `^extractor\.\*\.([\w-]+)` per confrontarli con `docs/gallery-dl.conf`.
- Le eccezioni trovate dallo script:
  - **14** righe di ancora (`.. _extractor.*.image-range:` e simili) a inizio riga dentro una voce,
    che sembrano etichette ma non lo sono;
  - **3** etichette scritte `Note:` con i due punti;
  - **2** tipi scritti `´´bool´´` con l'accento acuto al posto del backtick;
  - **1** etichetta in minuscolo, `description`, in `extractor.facebook.author-followups`;
  - **3** coppie di titoli impilati sullo stesso testo;
  - i 39 valori di serie in prosa, che non diventano un valore.
- **Uno schema leggibile da macchina non esiste.** Nell'albero dei file del tag (781 percorsi,
  letti con l'API `git/trees` di GitHub) non c'è nessun file di schema JSON. Le fonti più vicine
  sono due:
  - [`docs/gallery-dl.conf`](https://github.com/mikf/gallery-dl/blob/v1.32.21/docs/gallery-dl.conf):
    un file JSON con le opzioni ai valori di serie, che si legge con un normale lettore JSON
    (usa chiavi `"#"` ripetute per i commenti: il lettore di Python le accetta, quello di Swift
    **non verificato**). Le opzioni generali dell'estrattore sono 79, contro le 84 della
    documentazione: mancano `category-transfer`, `directory`, `filename`, `keywords-global`,
    `username`, `password`, `postprocessor-options` e `write-pages`; in più ha `category-map` e
    `config-map` (documentate fra le varie) e `metadata-parent` (nome alternativo di
    `parent-metadata`);
  - [`docs/options.md`](https://github.com/mikf/gallery-dl/blob/v1.32.21/docs/options.md),
    generato in automatico da `scripts/options.py`, con le sole opzioni da riga di comando.

## 🌐 Opzioni generali e opzioni per sito

- **Generali** (`extractor.*.<nome>`): **84** chiavi. Valgono per ogni sito e si possono
  ridefinire per categoria o sottocategoria.
- **Per sito**: **416** chiavi in **106** categorie. Di queste, **14** sono gruppi di siti scritti
  fra parentesi quadre (`[Danbooru]`, `[mastodon]`, `[booru]` e altri), che valgono per tutti i
  siti di quel tipo. I nomi finali diversi sono **210**, e molti si ripetono da un sito all'altro:
  `metadata` in 21 siti, `videos` in 20, `include` in 18.
- I siti con più opzioni: `twitter` 34, `deviantart` 22, `instagram` 16, `kemono` 14, `reddit` 13.
- **I livelli** (dalla sezione `Extractor Options`): un'opzione si scrive a livello base
  (`extractor.<nome>`), di categoria (`extractor.<sito>.<nome>`) o di sottocategoria
  (`extractor.<sito>.<tipo>.<nome>`), e il livello più specifico vince. ⚠️ Un'opzione scritta
  **in cima al file**, accanto a `extractor`, vale come impostazione globale e **scavalca tutte le
  altre**, anche quelle di un sito. Anche `-o chiave=valore` da riga di comando scrive in cima
  (in `gallery_dl/__init__.py` diventa `config.set((), ...)`).
- **I siti**: [`docs/supportedsites.md`](https://github.com/mikf/gallery-dl/blob/v1.32.21/docs/supportedsites.md)
  ha **358** righe di sito (più 31 righe di intestazione di gruppo). Nella colonna dell'accesso:
  297 vuote, 32 `Supported`, 11 `Cookies`, 10 `OAuth`, 5 `API Key`, 3 `Required`.

### Le principali per chi scarica gallerie di immagini

Un criterio oggettivo: le opzioni che gli autori hanno ritenuto abbastanza importanti da dare
loro un'opzione da riga di comando. In `gallery_dl/option.py` ci sono **124** definizioni di
opzione, **12** nascoste (nomi vecchi), quindi **111** visibili (le stesse 111 di `options.md`,
confrontate una per una); di queste **52** scrivono direttamente una chiave di configurazione.
Per gallerie di immagini contano soprattutto:

- **Dove e come si salva**: `base-directory` (`-d`), `directory`, `filename` (`-f`),
  `path-restrict` (`--restrict-filenames`).
- **Che cosa si salta**: `skip` (`--no-skip`, `-A`, `-T`), `archive` (`--download-archive`).
- **Che cosa si scarica**: `file-range` (`--range`), `date-after` e `date-before`,
  `filesize-min` e `filesize-max`, `file-filter` (`--filter`), `tags-blacklist`.
- **Accesso**: `username` e `password` (`-u`, `-p`), `cookies` (`-C`, `--cookies-from-browser`).
- **Garbo verso i siti**: `sleep`, `sleep-request`, `rate` (`-r`), `retries` (`-R`), `timeout`.
- **Dopo lo scaricamento**: `mtime` (data del file, `--no-mtime`), `postprocessors` con
  `metadata` (`--write-metadata`, `--write-tags`) e `zip` (`--zip`, `--cbz`).

## 📁 Cartelle e nomi dei file

Per i ticket `06-cartelle.md` e `07-rinomina.md`. Fonti: `docs/configuration.rst`,
[`gallery_dl/path.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/path.py),
[`docs/formatting.md`](https://github.com/mikf/gallery-dl/blob/v1.32.21/docs/formatting.md).

### Le opzioni

- **`base-directory`**: la cartella di partenza. Di serie `./gallery-dl/`, cioè **relativa alla
  cartella da cui si lancia il programma** (`path.py`: se manca vale `.` + `gallery-dl`). Haul
  deve quindi scriverla sempre, con un percorso assoluto. Accetta `~` e le variabili d'ambiente
  (`util.expand_path`). Può essere anche un oggetto di condizioni (vedi sotto).
- **`directory`**: la lista delle sottocartelle, una stringa di formato per livello, per esempio
  `["{category}", "{user[name]}"]`. Ogni sito ha la sua di serie (`directory_fmt` nel codice
  dell'estrattore). `-D` la svuota e mette tutto in una cartella sola.
- **`filename`**: il nome del file, una stringa di formato; ogni sito ha il suo di serie.
  `-f /O` vuol dire il nome originale (`{filename}.{extension}`). La chiave `extension` c'è
  sempre, anche quando il sito non la dà.
- **`path-restrict`**: i caratteri da sostituire nei nomi. Di serie `auto`, che su macOS vuol
  dire **solo `/`** (`path.py`: `"/"` se non è Windows). Valori speciali: `unix`, `windows`,
  `windows+` (sostituti Unicode dall'aspetto simile), `ascii`, `ascii+`, oppure un oggetto
  carattere → sostituto.
- **`path-replace`**: il sostituto, di serie `_`.
- **`path-remove`**: caratteri da togliere, di serie i caratteri di controllo ASCII.
- **`path-strip`**: caratteri da togliere in fondo a ogni pezzo; su macOS di serie nessuno.
- **`path-convert`**: conversioni applicate a ogni pezzo del percorso (per esempio `g` per un
  nome senza spazi né accenti).
- **`extension-map`**: estensioni da cambiare, di serie `jpeg`, `jpe`, `jfif`, `jif`, `jfi`
  diventano `jpg`.
- **`skip`**: che cosa fare con un file che c'è già. `true` salta, `false` sovrascrive,
  `enumerate` aggiunge un numero (`file.1.ext`), `abort:N` e `terminate:N` fermano dopo N salti
  di fila, `exit:N` chiude il programma.
- `path-extended` vale solo per Windows.

### Le condizioni

`filename`, `directory` e `base-directory` accettano anche un **oggetto**: ogni chiave è una
condizione (un'espressione Python, per esempio `extension == 'mp4'`), ogni valore il formato da
usare se è vera; le condizioni si provano in ordine e la chiave vuota `""` è il caso di riserva.
È il modo nativo per regole come i video in una cartella e le immagini in un'altra.

### La sintassi delle stringhe di formato

Da `docs/formatting.md`: segue `str.format()` di Python, con aggiunte. Un segnaposto è
`{nome!conversione:specificatore}`.

- **Nomi**: semplici (`{title}`), con indice (`{title[6]}`), con taglio (`{title[3:8]}`), con
  alternative (`{empty|title}`, usa il primo non vuoto), con accesso a un campo
  (`{user[name]}`).
- **Conversioni** (una lettera dopo `!`): **24** nella tabella, per esempio `l` minuscolo,
  `u` maiuscolo, `g` versione semplificata per i nomi di file, `W` spazi ripuliti, `j` JSON.
- **Specificatori** (dopo `:`): quelli di Python, come `{num:>03}` per lo zero davanti o
  `{date:%Y%m%d}` per le date, più **16** di gallery-dl, per esempio `?<inizio>/<fine>/`
  (aggiunge testo solo se il valore c'è), `L<lunghezza>/<sostituto>/` (sostituisce se troppo
  lungo), `R<vecchio>/<nuovo>/` (sostituisce testo), `J<separatore>/` (unisce una lista),
  `O<fuso>/` (cambia fuso orario).
- **Campi globali**: `_env` (variabili d'ambiente), `_now` (data e ora correnti), `_nul`, `_lit`.
- **Tipi speciali**: una stringa che comincia con `\f` più una lettera cambia linguaggio:
  `E` espressione Python, `F` f-string, `J` modello Jinja, `T`, `TF`, `TJ` da file, `M` funzione
  di un modulo Python. ⚠️ Questi eseguono codice.
- **I campi disponibili dipendono dal sito**: `gallery-dl -K <URL>` li elenca con valori di
  esempio, divisi fra campi per le cartelle e campi per i nomi dei file. È il modo per costruire
  un'anteprima dal vivo della rinomina.

## 🔑 Come si autentica gallery-dl

Per il ticket `08-accesso-ai-siti.md`. Fonti: `docs/configuration.rst`,
[`README.rst`](https://github.com/mikf/gallery-dl/blob/v1.32.21/README.rst) (§ `Authentication`),
[`gallery_dl/cookies.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/cookies.py),
[`gallery_dl/extractor/oauth.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/extractor/oauth.py).

- **Nome utente e password** (`extractor.<sito>.username` e `.password`, o `-u` e `-p`):
  `configuration.rst` elenca **35** siti. Per 3 (`horne`, `madokami`, `nijie`) l'accesso è
  obbligatorio; per 7 siti di tipo Danbooru ed e621 la password è la chiave API del profilo, non
  la password vera. ⚠️ Il `README.rst` elenca solo 15 siti, con `tsumino` che nella
  documentazione completa non c'è: il README è **indietro**, e fa fede `configuration.rst`.
  - Se la password manca, gallery-dl la chiede a terminale; `extractor.*.input` (di serie `false`
    quando l'ingresso non è un terminale, come quando lo lancia Haul) o `--no-input` lo
    impediscono, e allora si ferma con un errore.
  - `extractor.*.netrc` (`--netrc`) legge le credenziali dal file `~/.netrc`.
- **Cookie** (`extractor.<sito>.cookies`), in tre forme:
  - il percorso di un file `cookies.txt` nel formato Netscape;
  - un oggetto nome → valore;
  - una lista che indica un browser: nome, profilo, portachiavi, contenitore di Firefox,
    dominio. Equivale a `--cookies-from-browser BROWSER[/DOMINIO][+PORTACHIAVI][:PROFILO][::CONTENITORE]`.
    Browser supportati (`cookies.py`): `brave`, `chrome`, `chromium`, `edge`, `opera`,
    `thorium`, `vivaldi`; `firefox`, `librewolf`, `zen`, `floorp`; `safari`, `orion`.
  - Su macOS, per i browser Chromium la chiave dei cookie si legge dal Portachiavi con il comando
    `security find-generic-password`, quindi macOS può chiedere il permesso all'utente. Safari
    si legge da `~/Library/Cookies/Cookies.binarycookies` o dal contenitore di Safari; che serva
    l'accesso completo al disco è **non verificato** nel sorgente.
  - `cookies-update` (di serie `true`) riscrive il file `cookies.txt` con i cookie aggiornati;
    `cookies-select` ruota fra più fonti.
- **OAuth** (un'autorizzazione data dal sito, senza consegnare la password): si lancia
  `gallery-dl oauth:<sito>`. Gli estrattori sono **7**: `flickr`, `smugmug`, `tumblr`,
  `deviantart`, `reddit`, `mastodon` (con l'istanza, `oauth:mastodon:<server>`) e `pixiv`
  (obbligatorio per pixiv secondo il README). Il flusso (`oauth.py`): apre il browser, aspetta la
  risposta su un piccolo server locale alla porta **6414** (opzione `port`), poi mostra i token
  su stdout e li salva nella cache.
- **La cache**: `cache.file`, di serie `~/.cache/gallery-dl/cache.sqlite3` su macOS (vale il
  ramo non Windows), conserva sessioni, cookie e token fra un avvio e l'altro. ⚠️ È **condivisa**
  con il gallery-dl dell'utente, se non si cambia.
- **Chiavi dei servizi** (`API Tokens & IDs`): per 6 siti (DeviantArt, Flickr, MangaDex, Reddit,
  SmugMug, Tumblr) si può usare una chiave propria al posto di quella comune, con le istruzioni
  in `How To`.

## 📡 Seguire un download da un altro programma

Fonti: [`gallery_dl/__init__.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/__init__.py),
[`gallery_dl/job.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/job.py),
[`gallery_dl/output.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/output.py),
[`gallery_dl/exception.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/exception.py),
[`gallery_dl/downloader/http.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/downloader/http.py),
[`gallery_dl/postprocessor/metadata.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/postprocessor/metadata.py),
[`gallery_dl/actions.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/actions.py).

### Uscita in JSON: `-j` e `-J`

- `-j` (`--dump-json`) avvia un `DataJob`, che **non scarica niente**: raccoglie i dati e li
  stampa. Ogni messaggio è una lista `[tipo, ...]`: `2` cartella (con i dati del post), `3` file
  (con indirizzo e dati), `6` collegamento a un altro estrattore (`extractor/message.py`).
- Di serie stampa tutto alla fine, come un'unica lista JSON. Con `output.jsonl: true` stampa una
  riga JSON per messaggio, man mano (formato JSON Lines).
- I campi che cominciano con `_` sono tolti, salvo `output.private: true`.
- ⚠️ Il `DataJob` restituisce **sempre `0`**: un errore finisce nella lista come
  `{"error": ..., "message": ...}`, non nel codice d'uscita.
- `-J` (`--resolve-json`) fa lo stesso seguendo i collegamenti ad altri estrattori.
- Serve per un'anteprima (quanti file, che nomi), non per seguire lo scaricamento.

### `--simulate`

`-s` avvia un `SimulationJob`: fa tutto tranne scaricare, e per ogni file stampa il nome che
avrebbe, come se lo saltasse. ⚠️ Se l'archivio è attivo e `archive-mode` lo prevede, **scrive
anche nell'archivio** (`job.py`, `SimulationJob.handle_url`), quindi va usato senza archivio.

### Registro e livelli

- Il registro va su **stderr** con il formato `[{name}][{levelname}] {message}` (per esempio
  `[twitter][error] ...`), con i livelli `debug`, `info`, `warning`, `error` scritti in minuscolo.
- `-q` mostra solo gli errori gravi, `-w` avvisi ed errori, `-v` anche il debug. `output.log`
  cambia il formato, `output.logfile` (`--write-log`) scrive anche su file.
- `output.errorfile` (`-e`) scrive in un file gli indirizzi che hanno dato errore.

### I file scaricati e l'avanzamento

- `output.mode` decide che cosa si stampa su **stdout** per ogni file. Quando stdout non è un
  terminale (come quando lo legge Haul) vale `pipe`: una riga con il percorso per ogni file
  scaricato, e `# ` più il percorso per ogni file saltato.
- `output.mode` può essere anche un **oggetto** con cinque formati propri: `start`, `success`,
  `skip` (con `{}` per il percorso) e `progress`, `progress-total` (con `{0}` byte scaricati,
  `{1}` byte al secondo, `{2}` byte totali, `{3}` percentuale). Così Haul può dare a ogni riga un
  prefisso suo e riconoscerla senza ambiguità.
  - ⚠️ L'avanzamento va su **stderr**, insieme al registro, e i byte arrivano già arrotondati in
    forma leggibile (`1.23M`, funzione `util.format_value`), non come numero intero; solo la
    percentuale è un intero.
  - ⚠️ `downloader.*.progress` (di serie `3.0` secondi) non è un intervallo: superata quella
    soglia, l'avanzamento si stampa **a ogni blocco** ricevuto (`chunk-size`, di serie 32 KB).
    Con file grandi sono molte righe, e Haul deve diradarle da sé.
- **Un flusso di eventi più ricco**: il postprocessore `metadata` con `filename: "-"` scrive su
  stdout, per gli eventi scelti in `event`: `prepare`, `file`, `after` (file al suo posto),
  `skip`, `error`, `post`, `finalize` e altri. In modalità `jsonl` scrive i dati del file come
  una riga JSON; in modalità `print` scrive una stringa di formato (`content-format`), per esempio
  un prefisso fisso più `{_path}`. È lo stesso meccanismo che usa `--print` (`option.py`,
  `PrintAction`). Come combinare più eventi distinguibili in un solo flusso va provato in un
  prototipo (**non verificato** nella pratica).
- **Il buffer**: quando stdout non è un terminale, Python accumula il testo prima di mandarlo.
  `output.stdout` e `output.stderr` accettano `{"line_buffering": true}`, che fa uscire ogni riga
  subito (`output.configure_standard_streams`).
- Con più indirizzi, `output.progress` stampa `[{current}/{total}] {url}`.

### I codici d'uscita

Non sono documentati: li ho ricavati dal codice. Sono **bit sommati con l'OR**, quindi più errori
diversi danno un numero composto (per esempio `4 | 16 = 20`):

| Codice | Origine nel codice | Significato |
|---|---|---|
| `0` | | tutto bene |
| `1` | `GalleryDLException`, errori inattesi, interruzione | errore generico |
| `4` | `ExtractionError`, `DownloadJob` | estrazione o download di un file fallito |
| `8` | `ChallengeError` | il sito ha mostrato una verifica anti-robot |
| `16` | `AuthenticationError`, `AuthorizationError` | accesso negato o richiesto |
| `32` | `InputError` (formato, filtro, file di ingresso) | errore nei dati dati in ingresso |
| `64` | `NoExtractorError` in `__init__.py` | indirizzo non supportato |
| `128` | `OSError` in `job.py` | errore di rete o di disco |

### Interruzione e ripresa

- **Interrompere**: il segnale `SIGINT` (lo stesso di Ctrl+C) diventa `KeyboardInterrupt` e il
  programma esce con codice `1` e il messaggio `KeyboardInterrupt` su stderr (`__init__.py`).
  Il segnale `SIGTERM` non ha un gestore, quindi chiude il processo all'istante.
- `signals-actions` permette di legare un segnale a un'azione, per esempio
  `"SIGINT": "flag download = stop"`, che ferma solo il file in corso (`actions.py`,
  `action_flag`); `signals-ignore` ignora un segnale.
- **Riprendere**: con `downloader.*.part: true` (di serie) il file si scrive come `.part` e si
  rinomina alla fine. Un `.part` rimasto è ripreso dal punto in cui si era fermato, con
  l'intestazione HTTP `Range` (`http.py`), se il server la supporta. Con `part: false` il file
  incompleto è cancellato.
- Al nuovo avvio i file già presenti sono saltati (`skip: true`), quindi rilanciare lo stesso
  indirizzo continua dove si era rimasti.

### L'archivio dei file già scaricati

- `extractor.*.archive` (`--download-archive FILE`): un database **SQLite**, non un file di
  testo, con gli identificativi dei file scaricati; quelli già presenti sono saltati anche se il
  file è stato spostato o rinominato. Accetta i segnaposti di formato nel percorso (per esempio
  un archivio per sito) e anche un indirizzo PostgreSQL.
- `archive-format` cambia l'identificativo, `archive-mode` decide quando scrivere: `file` (subito,
  di serie) o `memory` (solo a lavoro finito senza errori).

## ⚙️ Dove legge il `config.json`, e come Haul usa il suo

Fonti: [`gallery_dl/config.py`](https://github.com/mikf/gallery-dl/blob/v1.32.21/gallery_dl/config.py),
`gallery_dl/__init__.py`, `gallery_dl/option.py`.

- **I percorsi di serie su macOS** (il ramo non Windows di `config.default`), letti tutti in
  ordine e fusi uno sull'altro:
  1. `/etc/gallery-dl.conf`
  2. `${XDG_CONFIG_HOME}/gallery-dl/config.json` se la variabile esiste, altrimenti
     `~/.config/gallery-dl/config.json`
  3. `~/.gallery-dl.conf`
  4. per l'eseguibile autonomo (PyInstaller), anche `gallery-dl.conf` nella cartella
     dell'eseguibile.
- `GDL_CONFIG_TYPE` o `--config-type` cambiano il formato in YAML o TOML.
- **Come isolarsi**: `--config-ignore` salta tutti i file di serie (`config_load` diventa falso),
  e `-c FILE` carica un file indicato, con errore bloccante se manca o è guasto (`strict=True`).
  Con le due insieme gallery-dl legge **solo** il file di Haul.
- Attenzione a quello che il file di Haul non controlla:
  - `subconfigs` dentro un file carica altri file;
  - la **cache** (`cache.file`) e i file dei cookie sono condivisi se non si cambiano;
  - `--config-create` e `--config-open` agiscono sui percorsi di serie, cioè sui file
    dell'utente: Haul non deve usarli.

## 💡 Raccomandazione

### Le voci da curare a mano

Una lista corta, scelta fra le opzioni che hanno anche un'opzione da riga di comando:

1. 📁 **Cartella di destinazione**: `base-directory`, sempre assoluta.
2. 🗂️ **Sottocartelle**: `directory`, con modelli pronti (per sito, per sito e autore, tutto in
   una cartella) e un campo libero.
3. 🏷️ **Nome dei file**: `filename`, con modelli pronti, il nome originale e l'anteprima fatta con
   `-K`.
4. 🔤 **Caratteri nei nomi**: `path-restrict` (proposta: `windows` o `windows+` per avere nomi
   validi anche su dischi esterni e altri sistemi) e `path-replace`.
5. ♻️ **File già presenti**: `skip` (salta, sovrascrivi, numera).
6. 🗃️ **Ricorda i file scaricati**: `archive`, un database in `~/Library/Application Support`
   di Haul.
7. 🔑 **Accesso per sito**: `username` e `password`, `cookies` dal browser, OAuth.
8. 🐢 **Garbo verso i siti**: `sleep`, `sleep-request`, `downloader.*.rate`, `retries`,
   `timeout`.
9. 🎯 **Che cosa scaricare**: `date-after`, `filesize-min` e `filesize-max`, `file-range`.
10. 🕰️ **Data dei file**: `downloader.*.mtime`.
11. 📝 **Metadati accanto ai file**: il postprocessore `metadata` (sì o no).
12. 🎬 Per sito, solo le voci ricorrenti più chiare: `videos` (20 siti), `include` (18 siti).

### Come generare le altre

- Un generatore, nel nucleo, legge `docs/configuration.rst` **della versione di gallery-dl che
  Haul porta con sé** e scrive un catalogo JSON incluso nell'app: chiave, sezione, sito, tipi,
  valore di serie, valori ammessi, descrizione. Si lancia a ogni aggiornamento di gallery-dl, non
  a ogni avvio dell'app.
- Le regole di lettura: titoli sottolineati da trattini, etichette a inizio riga, le 14 ancore
  ignorate, le eccezioni elencate sopra normalizzate (`Note:`, `description`, `´´bool´´`, titoli
  impilati, titoli con `&` divisi in chiavi).
- **Dal tipo al controllo**: `bool` → interruttore; `integer` e `float` → campo numerico;
  `string` con `Supported Values` → menu; `string` → campo di testo; `list` of `strings` → elenco
  di voci; `Path` → scelta di file o cartella; `Format String` → campo con anteprima; tutto il
  resto (oggetti, condizioni, più tipi) → piccolo editor JSON con la descrizione accanto.
- **Il valore di serie**: il letterale della documentazione, verificato contro `gallery-dl.conf`
  dove c'è; quando è in prosa, si mostra il testo e si lascia vuoto (vale il valore di serie di
  gallery-dl).
- La descrizione resta **in inglese**, com'è nel sorgente, per l'area avanzata: tradurre 654
  chiavi a ogni versione non è sostenibile. Le voci curate hanno testi italiani scritti a mano.
- **Una prova automatica** a ogni aggiornamento: il generatore deve leggere tutti i 646 titoli
  senza voci scartate, ogni voce curata deve esistere ancora nel catalogo nuovo, e il confronto
  fra catalogo vecchio e nuovo elenca le chiavi aggiunte, tolte o cambiate di tipo.
- Le chiavi che l'utente ha già e che il catalogo non conosce si **conservano**, mai cancellate.

### Come Haul lancia e legge gallery-dl

- Un processo separato per ogni lavoro:
  `gallery-dl --config-ignore -c <file di Haul> <URL>`.
- Il file di Haul contiene, oltre alle scelte dell'utente:
  - `extractor.input: false` (mai domande a terminale);
  - `cache.file` nella cartella di Haul, per non toccare la cache dell'utente;
  - `output.stdout` e `output.stderr` con `line_buffering: true`;
  - `output.mode` come oggetto, con un prefisso proprio per `start`, `success`, `skip`,
    `progress` e `progress-total`;
  - `output.log` con un prefisso e campi separati da tabulazione, per esempio
    `HAUL-LOG\t{levelname}\t{name}\t{message}`;
  - eventualmente il postprocessore `metadata` su stdout per gli eventi `after`, `skip` ed
    `error`, da provare in un prototipo.
- Haul legge stdout e stderr riga per riga, riconosce i prefissi, dirada l'avanzamento, e alla
  fine legge il codice d'uscita come somma di bit.
- **Anteprima**: `-j` con `output.jsonl: true` per sapere quanti file e quali nomi, prima di
  scaricare; `-K` per i campi disponibili nella rinomina.
- **Annullare**: `SIGINT`, poi `SIGKILL` (chiusura forzata) se non esce entro qualche secondo; i
  `.part` restano e il lavoro riparte da lì.

### Rischi aperti

- ⚠️ **Il formato della documentazione non è un contratto**: le eccezioni di oggi sono poche, ma
  una versione nuova può introdurne altre. Va gestito con la prova automatica, non con la fiducia.
- ⚠️ **La documentazione non è sempre allineata al codice**: il README elenca 15 siti con
  password contro i 35 di `configuration.rst`, e `gallery-dl.conf` non ha tutte le chiavi
  documentate.
- ⚠️ **Password in chiaro**: `-u` e `-p` sono visibili a chi elenca i processi, e un file di
  configurazione con la password è un file con un segreto. Le regole del progetto vogliono i
  segreti fuori dai file in chiaro: la scelta (Portachiavi di macOS più un file temporaneo con
  permessi ristretti, oppure solo cookie e OAuth) è del ticket `08-accesso-ai-siti.md`.
- ⚠️ **Codice eseguibile nella configurazione**: condizioni, filtri e formati `\fE`, `\fM`, il
  postprocessore `exec` eseguono codice Python. Importare la configurazione di un altro vuol dire
  fidarsi di lui; l'area avanzata dovrebbe segnalarlo.
- ⚠️ **L'avanzamento è approssimato** (byte arrotondati) e **frequente** (a ogni blocco).
- ⚠️ **I cookie dal browser su macOS** possono far comparire richieste di permesso del
  Portachiavi; per Safari, l'accesso completo al disco è **non verificato**.
- ⚠️ **L'eseguibile per macOS**: in `gallery_dl/update.py` il canale stabile ha binari solo per
  Windows e Linux, mentre `gallery-dl_macos` esiste solo nel canale di sviluppo
  (`gdl-org/builds`). La fonte dell'eseguibile è del ticket `01-fonte-di-gallery-dl.md`.
- ⚠️ **Nomi che cambiano**: `config-map` rinomina categorie vecchie (per esempio `kemonoparty` in
  `kemono`), e 12 opzioni da riga di comando sono nascoste perché sostituite. Il catalogo deve
  sapere migrare una chiave vecchia.
