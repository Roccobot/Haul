# Da dove prende Haul l'eseguibile di gallery-dl

> Ricerca per il ticket `issues/01-fonte-di-gallery-dl.md`. Fonti lette il 2026-11-21, con la
> versione o la data indicate accanto a ognuna. Quello che non è stato possibile verificare è
> segnato come **non verificato**. Le note sulla licenza sono una lettura dei testi, non una
> consulenza legale.

## 📖 Due parole prima di cominciare

- **Eseguibile autonomo**: un solo file che contiene gallery-dl, un interprete Python e le
  librerie che servono, e che parte senza installare altro. Lo si costruisce con **PyInstaller**,
  uno strumento che impacchetta un programma Python in un file solo.
- **Interprete Python incluso**: una copia di Python messa dentro l'app; gallery-dl allora resta
  un insieme di file `.py` (testo leggibile), che l'interprete esegue.
- **Wheel** (`.whl`): il pacchetto pronto all'uso con cui si distribuisce una libreria Python.
  Una wheel `py3-none-any` contiene solo codice Python puro, senza parti compilate per una
  macchina precisa.
- **PyPI**: il registro ufficiale dei pacchetti Python.
- **Firma del codice**: un sigillo crittografico sul file eseguibile. **Developer ID** è il
  certificato di Apple per le app distribuite fuori dal Mac App Store; una firma **ad hoc** è un
  sigillo senza identità, che dice solo che il file non è stato alterato.
- **Autenticazione** (in inglese notarization): il controllo automatico di Apple su un file già
  firmato con Developer ID; se va bene, Apple rilascia un biglietto che Gatekeeper trova in rete.
- **Quarantena**: un'etichetta (`com.apple.quarantine`) che macOS mette sui file scaricati da
  certi programmi. **Gatekeeper** controlla i file con quell'etichetta prima del primo avvio.
- **Checksum** (per esempio SHA-256): l'impronta numerica di un file; se un byte cambia, cambia
  l'impronta. **Firma GPG**: un'impronta firmata dall'autore con la sua chiave personale, che
  dimostra chi ha pubblicato il file.

## 🧭 Due fatti che cambiano il quadro del ticket

1. **Lo sviluppo di gallery-dl si è spostato su Codeberg.** L'annuncio, del 2026-04-05, segue una
   segnalazione DMCA (una richiesta di rimozione per violazione del diritto d'autore) e dice che il
   repository GitHub resta solo per l'infrastruttura secondaria: prove automatiche, build
   giornaliere e immagini Docker
   ([issue #9374 su GitHub](https://github.com/mikf/gallery-dl/issues/9374), letta il 2026-11-21).
   Le note di ogni versione su GitHub rimandano a Codeberg
   ([release v1.32.21 su GitHub](https://github.com/mikf/gallery-dl/releases/tag/v1.32.21), del
   2026-11-21).
2. **Le versioni stabili hanno eseguibili, ma solo su Codeberg e non per macOS.** Su GitHub le
   release dalla v1.32.14 alla v1.32.21 non hanno nessun file allegato, mentre fino alla v1.31.10
   (2026-03-19) avevano `gallery-dl.bin`, `gallery-dl.exe` e le loro firme (API di GitHub,
   `repos/mikf/gallery-dl/releases`, letta il 2026-11-21). Su Codeberg la v1.32.21 ha `gallery-dl.bin`
   (Linux), `gallery-dl.exe` e `gallery-dl_x86.exe` (Windows), la wheel, il sorgente `.tar.gz`,
   `SHA256SUMS`, `SHA512SUMS` e le firme
   ([API di Codeberg, release](https://codeberg.org/api/v1/repos/mikf/gallery-dl/releases), letta
   il 2026-11-21). Il README elenca gli eseguibili stabili solo per Windows e Linux
   ([README.rst su Codeberg](https://codeberg.org/mikf/gallery-dl/raw/branch/master/README.rst),
   ramo `master`, letto il 2026-11-21). Lo script di rilascio costruisce i tre eseguibili in
   macchine virtuali Linux e Windows del manutentore, nessuna macOS
   ([scripts/release.sh](https://codeberg.org/mikf/gallery-dl/raw/branch/master/scripts/release.sh),
   righe 59-75, letto il 2026-11-21).

E un terzo fatto, che semplifica le architetture: **macOS 27 gira solo su Mac con chip Apple**.
La pagina di compatibilità elenca solo modelli Apple silicon (M1 e successivi, MacBook Neo con
A18 Pro) e dice che si aggiorna se si ha un Mac con Apple silicon
([Apple Support 127255](https://support.apple.com/en-us/127255), pubblicata il 2026-11-05, letta
il 2026-11-21). Con il minimo a macOS 27.2, a Haul basta `arm64`: il binario universale non serve.

## 🛣️ Le quattro strade

| Sigla | Strada | In breve |
|---|---|---|
| Fonte-01 | Build giornaliera `gallery-dl_macos` di `gdl-org/builds` | pronta, ma di sviluppo e firmata solo ad hoc |
| Fonte-02 | Eseguibile stabile ufficiale per macOS | **non esiste** (vedi sopra) |
| Fonte-03 | Wheel stabile da PyPI, eseguita da un Python incluso in Haul | nessun binario scaricato |
| Fonte-04 | Eseguibile costruito da Haul con PyInstaller sulle stabili, firmato e autenticato | binario di Haul |

### Fonte-01: le build giornaliere di `gdl-org/builds`

- **Come nascono.** Il flusso `executables.yml` del repository GitHub parte ogni giorno a
  mezzanotte (`cron: '0 0 * * *'`), costruisce con PyInstaller su `macOS-latest` con Python
  3.14 e pubblica in `gdl-org/builds` con l'etichetta della data
  ([executables.yml](https://github.com/mikf/gallery-dl/blob/master/.github/workflows/executables.yml),
  letto il 2026-11-21). Lo script usa `--onefile`, cioè un file solo
  ([scripts/pyinstaller.py](https://github.com/mikf/gallery-dl/blob/master/scripts/pyinstaller.py)).
- **Stabilità e frequenza.** È costruita dall'ultimo commit di `master`, non da una versione
  stabile: il README la chiama Nightly Build. In `gdl-org/builds` ci sono 626 release, dalla
  `2024.03.09` alla `2026.11.21`; la più recente è del 2026-11-21 alle 00:18Z (API di GitHub,
  `repos/gdl-org/builds/releases`, letta il 2026-11-21). Una versione nuova si scopre con
  `GET /repos/gdl-org/builds/releases/latest`.
- **Architettura e macOS minimo.** Il file `gallery-dl_macos` del 2026-11-21 è stato scaricato e
  ispezionato senza eseguirlo: `Mach-O 64-bit executable arm64`, solo Apple silicon, con
  `minos 11.0` (macOS 11 come minimo) e SDK 26.2 (`file` e `otool -l`, eseguiti il 2026-11-21).
  ⚠️ Il flusso dichiara `architecture: "x64"`, ma il file è `arm64`: la configurazione e il
  prodotto non coincidono, e un cambio del runner di GitHub potrebbe cambiare l'architettura
  senza preavviso.
- **Firma.** `codesign -dv` dà `Signature=adhoc` e `TeamIdentifier=not set`: firma ad hoc, nessun
  Developer ID, nessuna autenticazione (eseguito il 2026-11-21).
- **Integrità.** Non ci sono firme GPG né file di checksum del manutentore. L'API di GitHub
  riporta per ogni file un campo `digest` SHA-256, calcolato da GitHub; per il file del
  2026-11-21 è `b496268a...3749`, e coincide con lo `shasum -a 256` del file scaricato. Questo
  dimostra che il file non si è rovinato nel viaggio, non chi lo ha prodotto.
- **Dimensione.** 22.573.472 byte, circa 22,6 MB (API di GitHub, release `2026.11.21`).
- **Tempo di avvio.** **Non misurato**: il file non è stato eseguito, perché arriva da una fonte
  che non firma. PyInstaller dichiara che un eseguibile `onefile` all'avvio estrae i suoi file
  in una cartella temporanea, e per questo è un po' più lento della versione a cartella
  ([PyInstaller, operating mode](https://pyinstaller.org/en/stable/operating-mode.html), letta il
  2026-11-21).
- **Dipendenze opzionali.** La build installa `requests[socks]`, `yt-dlp[default]`, `pyyaml` e
  `truststore` prima di PyInstaller, e il gancio `hook-gallery_dl.py` aggiunge `yt_dlp` agli
  import nascosti: yt-dlp è quindi dentro l'eseguibile
  ([hook-gallery_dl.py](https://github.com/mikf/gallery-dl/blob/master/scripts/hook-gallery_dl.py),
  letto il 2026-11-21). `ffmpeg` non c'è.
- **Aggiornatore interno.** gallery-dl ha un comando `-U` che si aggiorna da solo; per macOS
  conosce solo il canale `dev` di `gdl-org/builds` (`BINARIES_DEV`, `"macos": "gallery-dl_macos"`
  in [gallery_dl/update.py](https://codeberg.org/mikf/gallery-dl/raw/branch/master/gallery_dl/update.py),
  letto il 2026-11-21). Haul non deve usarlo, perché sostituirebbe il file alle spalle dell'app.
- **Rischio di continuità.** Le build vivono proprio sull'infrastruttura GitHub che ha ricevuto la
  segnalazione DMCA (issue #9374 citata sopra).

### Fonte-02: un eseguibile stabile ufficiale per macOS

- **Non esiste** il 2026-11-21: né su GitHub né su Codeberg, né nel README, né nello script di
  rilascio (fonti nella sezione sui due fatti). Strada chiusa finché il manutentore non la apre.

### Fonte-03: la wheel stabile da PyPI con un Python incluso

- **Che cosa si scarica.** Solo codice Python puro: `gallery_dl-1.32.21-py3-none-any.whl`,
  pubblicata su PyPI il 2026-11-21 09:56Z, con `requires_python >=3.8` e la sola dipendenza
  obbligatoria `requests>=2.11.0`
  ([API JSON di PyPI per gallery-dl](https://pypi.org/pypi/gallery-dl/json), letta il 2026-11-21).
  Anche `requests` 2.34.2, `charset-normalizer` 3.5.2, `urllib3` 2.8.0, `idna` 3.20, `certifi`
  2026.11.20 e `yt-dlp` 2026.10.26 hanno una wheel `py3-none-any` (API JSON di PyPI di ognuno,
  lette il 2026-11-21). Quindi **nessun file eseguibile** arriva dalla rete: l'unico eseguibile è
  l'interprete, che è dentro Haul e firmato con lui.
- **Stabilità e frequenza.** Sono le versioni stabili. Nel 2026 le release sono state, per mese:
  4 a gennaio, 2 a febbraio, 3 a marzo, 1 ad aprile, 1 a maggio, 4 a giugno, 3 a luglio, 2 ad
  agosto, 4 a settembre, 5 a ottobre, 2 a novembre fino al 21 (API di GitHub, date di
  pubblicazione delle release, contate il 2026-11-21): in media circa una a settimana negli
  ultimi mesi. Una versione nuova si scopre da `https://pypi.org/pypi/gallery-dl/json`, campo
  `info.version`, oppure dall'API delle release di Codeberg.
- **Integrità.** Due livelli, entrambi verificabili da Haul:
  - l'API JSON di PyPI dà lo SHA-256 di ogni file; per la wheel 1.32.21 è `f3f3e62b...8811`, e
    coincide con la riga di `SHA256SUMS` pubblicata su Codeberg dal manutentore (letti il
    2026-11-21);
  - su Codeberg ogni wheel ha una firma GPG `.asc`, e `SHA256SUMS` ha una firma `.sig`. La firma
    di `SHA256SUMS` è una firma OpenPGP binaria con l'impronta della chiave
    `3E09 F590 8333 DD83 DBDC E737 5680 CA38 9D36 5A88` (letta con `xxd` il 2026-11-21); lo
    script di rilascio la produce con `gpg --detach-sign` (`release.sh`, righe 135-145).
    Che quella chiave sia del manutentore è **non verificato** in questa ricerca.
  - PyPI non ha attestazioni di provenienza per questa wheel: l'indirizzo
    `https://pypi.org/integrity/gallery-dl/1.32.21/gallery_dl-1.32.21-py3-none-any.whl/provenance`
    risponde 404 (letto il 2026-11-21).
- **Il Python da includere.** `python-build-standalone`, ora mantenuto da Astral, pubblica
  interpreti pronti; la release `20261110` (2026-11-11) contiene
  `cpython-3.14.10+20261110-aarch64-apple-darwin-install_only_stripped.tar.gz`, di 26.970.375
  byte compressi (circa 27 MB), con il `digest` SHA-256 e un file `SHA256SUMS`
  ([release 20261110](https://github.com/astral-sh/python-build-standalone/releases/tag/20261110),
  letta il 2026-11-21). La destinazione `aarch64-apple-darwin` è Apple silicon
  ([docs/running.rst](https://github.com/astral-sh/python-build-standalone/blob/main/docs/running.rst)),
  ed è compilata con `-mmacosx-version-min=11.0`
  ([cpython-unix/targets.yml](https://github.com/astral-sh/python-build-standalone/blob/main/cpython-unix/targets.yml),
  letto il 2026-11-21). Licenza del progetto: MPL-2.0 (API di GitHub). La dimensione una volta
  estratto è **non verificata**.
- **Firma e Gatekeeper.** L'interprete entra nell'app al momento della build, e si firma con il
  Developer ID di Haul e il runtime protetto (hardened runtime) insieme al resto dell'app, che poi
  si autentica una volta sola. Le wheel scaricate sono file di testo e non sono codice firmabile:
  nessuna quarantena da gestire. Se Python, sotto il runtime protetto, abbia bisogno di qualche
  permesso speciale (entitlement) è **non verificato** e va provato nel prototipo.
- **Limite importante.** Solo i pacchetti Python puri si possono aggiornare scaricandoli. Un
  modulo compilato (`.so`) scaricato dopo l'installazione non porterebbe la firma di Haul, e il
  controllo delle librerie del runtime protetto lo rifiuterebbe. Le dipendenze opzionali
  compilate (per esempio `brotli`) vanno messe nell'app al momento della build, o lasciate fuori.
- **Tempo di avvio.** **Non misurato.** Non c'è l'estrazione in una cartella temporanea della
  Fonte-01, ma il confronto in numeri va fatto nel prototipo.

### Fonte-04: un eseguibile costruito da Haul con PyInstaller

- **Come si fa.** Una macchina di build di Haul (per esempio GitHub Actions su macOS) scarica il
  sorgente stabile, controlla la firma GPG, costruisce con PyInstaller e firma con il Developer ID
  di Haul. PyInstaller firma ad hoc per difetto; con `--codesign-identity` firma con un'identità
  vera e accende il runtime protetto, e per `onefile` la firma dei file interni si può fare solo
  durante la build
  ([PyInstaller, feature notes](https://pyinstaller.org/en/stable/feature-notes.html), letta il
  2026-11-21). PyInstaller produce per difetto un binario per l'architettura della macchina che
  costruisce, quindi `arm64` su un runner Apple silicon (stessa pagina).
- **Autenticazione.** Il servizio di Apple accetta uno ZIP con dentro lo strumento. Apple scrive
  che i biglietti si creano anche per i binari autonomi, ma che oggi non si possono attaccare
  (staple) a loro: Gatekeeper li trova solo in rete
  ([Customizing the notarization workflow](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow),
  letta il 2026-11-21). L'autenticazione chiede Developer ID, runtime protetto e marca temporale
  sicura
  ([Notarizing macOS software before distribution](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution),
  letta il 2026-11-21).
- **Frequenza.** Segue le stabili, ma con un ritardo: ogni versione di gallery-dl (circa una a
  settimana) richiede una build di Haul, una firma, un'autenticazione e una pubblicazione.
- **Integrità.** La più forte delle quattro: Haul controlla che il file scaricato porti la firma
  del suo stesso Team ID, oltre allo SHA-256.
- **Costi.** Minuti di GitHub Actions su macOS a ogni versione di gallery-dl, il segreto del
  certificato sulla macchina di build, e un flusso di rilascio in più da mantenere.
- **Dimensione e avvio.** Simili alla Fonte-01 (stesso PyInstaller `onefile`): circa 22 MB e
  l'estrazione a ogni avvio; **non misurati** per una build di Haul.

## 🛡️ Firma, autenticazione e quarantena per un file scaricato dall'app

- **Su Apple silicon ogni eseguibile deve essere firmato** prima di partire, ma basta una firma
  ad hoc: nessuna identità richiesta
  ([note di rilascio di macOS Big Sur 11.0.1, Universal Apps](https://developer.apple.com/documentation/macos-release-notes/macos-big-sur-11_0_1-universal-apps-release-notes),
  lette il 2026-11-21). La build giornaliera della Fonte-01 ha questa firma minima.
- **La quarantena la mette chi scarica, se lo dichiara.** La chiave `LSFileQuarantineEnabled`
  dell'`Info.plist` indica se i file creati da un'app sono messi in quarantena per difetto
  ([LSFileQuarantineEnabled](https://developer.apple.com/documentation/bundleresources/information-property-list/lsfilequarantineenabled),
  letta il 2026-11-21). Che un'app non sandbox, senza quella chiave, scarichi con `URLSession` un
  file **senza** quarantena è la lettura di questa pagina, ma è **non verificato** sul campo.
  Prova parziale: il file della Fonte-01 scaricato con `curl` dal Terminale ha solo l'attributo
  `com.apple.provenance`, non `com.apple.quarantine` (`xattr`, eseguito il 2026-11-21).
- **Gatekeeper controlla il software scaricato al primo avvio**, e verifica che sia autenticato
  da Apple ([Apple Platform Security, Gatekeeper and runtime protection](https://support.apple.com/guide/security/gatekeeper-and-runtime-protection-sec5599b66df/web),
  pubblicata il 2024-12-19, letta il 2026-11-21). La pagina non parla degli strumenti a riga di
  comando lanciati da un'altra app.
- **Che cosa serve perché parta senza avvisi:**
  - Fonte-01: file senza quarantena e con la firma ad hoc. Funziona solo finché macOS non mette
    o non controlla etichette diverse (l'attributo `com.apple.provenance` è nuovo, e il suo
    effetto è **non verificato**). È la strada più esposta a un cambiamento di macOS.
  - Fonte-03: nessun eseguibile scaricato, quindi nessun avviso possibile per gallery-dl.
  - Fonte-04: file firmato Developer ID e autenticato; parte anche con la quarantena, purché
    Gatekeeper raggiunga in rete il biglietto, perché non si può attaccare al binario.

## 🧩 Dipendenze opzionali: `yt-dlp` e `ffmpeg`

- Il README di gallery-dl le elenca come opzionali: `yt-dlp` (o `youtube-dl`) per i video HLS e
  DASH e l'integrazione `ytdl`, `FFmpeg` per convertire le animazioni Ugoira di Pixiv, `mkvmerge`
  per i tempi esatti dei fotogrammi Ugoira (README.rst, sezione Optional, letto il 2026-11-21).
- **yt-dlp.** Licenza Unlicense (pubblico dominio); versione 2026.10.26
  ([release su GitHub](https://github.com/yt-dlp/yt-dlp/releases/tag/2026.10.26), letta il
  2026-11-21). gallery-dl lo usa come modulo Python (`import yt_dlp`), quindi:
  - Fonte-01 e Fonte-04: è dentro l'eseguibile (come visto nel gancio PyInstaller);
  - Fonte-03: si scarica la sua wheel pura da PyPI accanto a quella di gallery-dl, con lo stesso
    controllo SHA-256. yt-dlp pubblica anche un `yt-dlp_macos` universale per macOS 10.15+ (README
    di yt-dlp), ma a gallery-dl serve il modulo, non l'eseguibile.
- **ffmpeg.** La pagina di download ufficiale non offre file per macOS: rimanda a build statiche
  di terzi su `evermeet.cx` ([ffmpeg.org/download.html](https://ffmpeg.org/download.html), letta
  il 2026-11-21). Architettura, firma e licenza di quelle build sono **non verificate**. Serve
  solo per Ugoira: la scelta più semplice è cercarlo dove l'utente lo ha già (per esempio
  Homebrew) e spiegare in un'impostazione come aggiungerlo, senza includerlo.

## ⚖️ Licenza: GPL-2.0 di gallery-dl e GPL-3.0 di Haul

- **gallery-dl è GPL-2.0 solo**, senza la clausola o versioni successive: l'intestazione dice
  version 2 as published by the Free Software Foundation
  ([gallery_dl/\_\_init\_\_.py](https://codeberg.org/mikf/gallery-dl/raw/branch/master/gallery_dl/__init__.py)),
  e PyPI riporta `GPL-2.0-only` (API JSON, letta il 2026-11-21).
- **Programmi separati.** La GPL-2.0, sezione 2, dice che la semplice aggregazione di un'altra
  opera con il programma su uno stesso supporto non porta l'altra opera sotto la licenza
  ([LICENSE di gallery-dl](https://codeberg.org/mikf/gallery-dl/raw/branch/master/LICENSE), riga
  129). Haul lancia gallery-dl come processo separato e gli parla con argomenti e testo: è il caso
  dei due programmi distinti, quindi Haul resta GPL-3.0. Le FAQ della FSF su aggregazione e
  compatibilità fra GPL-2.0 solo e GPL-3.0 sono **non lette** in questa sessione (il sito
  `gnu.org` ha rifiutato l'accesso automatico): vanno lette prima della release.
- **Che cosa chiede, strada per strada:**
  - **Haul non distribuisce gallery-dl**, lo scarica sul Mac dell'utente da PyPI o da GitHub
    (Fonte-01 e Fonte-03 scaricate al primo avvio): Haul non lo ridistribuisce, e gli obblighi
    della GPL-2.0 sulla distribuzione non scattano. Resta buona pratica mostrare licenza e
    collegamento al sorgente nella finestra Informazioni.
  - **Haul include gallery-dl nell'app** (per esempio la prima wheel dentro il DMG, per funzionare
    senza rete al primo avvio): è distribuzione. La sezione 3 chiede di accompagnare il programma
    con il sorgente completo o con un'offerta scritta valida tre anni; se il programma si offre da
    un posto preciso, offrire il sorgente dallo stesso posto vale come distribuzione del
    sorgente (LICENSE, righe 138-168). In pratica: allegare alla release di Haul il
    `gallery_dl-<versione>.tar.gz` usato, con il testo della licenza.
  - **Fonte-04**: Haul distribuisce un binario che ha costruito, quindi deve pubblicare il
    sorgente corrispondente, compresi gli script di build. Il binario contiene anche il caricatore
    di PyInstaller e l'interprete Python, con licenze proprie (la licenza del caricatore è
    **non verificata** in questa ricerca).
- **Da evitare**: unire il codice di gallery-dl dentro il processo di Haul in modo da farne un
  solo programma (per esempio caricarlo come libreria nel nucleo). Con una GPL-2.0 solo, quella
  combinazione con codice GPL-3.0 è il caso problematico.

## 🧾 Riepilogo per punto del ticket

| Punto | Fonte-01 notturna | Fonte-03 PyPI + Python | Fonte-04 build di Haul |
|---|---|---|---|
| Stabilità | sviluppo, ogni giorno | stabile, circa settimanale | stabile, con ritardo |
| Novità | API `gdl-org/builds` | API JSON di PyPI | API delle release di Haul |
| Architettura | `arm64`, macOS 11+ (osservato) | `arm64`, macOS 11+ | `arm64` (a scelta) |
| Firma | ad hoc, non autenticata | interprete firmato con Haul | Developer ID, autenticata |
| Integrità | `digest` di GitHub | SHA-256 PyPI + GPG del manutentore | Team ID + SHA-256 |
| Dimensione | 22,6 MB | 27 MB compressi una volta + circa 1 MB a versione | circa 22 MB (stima) |
| Avvio | estrazione a ogni avvio, non misurato | non misurato | estrazione a ogni avvio, non misurato |
| yt-dlp | incluso | wheel da PyPI | incluso |
| ffmpeg | no | no | no |
| Licenza | nessuna distribuzione | nessuna, o sorgente allegato | sorgente e script da pubblicare |

## ✅ Raccomandazione

**Strada consigliata: Fonte-03.** Haul include un interprete Python di `python-build-standalone`
per `arm64`, firmato e autenticato con l'app; gallery-dl, `requests` con le sue dipendenze e
`yt-dlp` si scaricano come wheel pure da PyPI in una cartella di Haul in `Application Support`, e
Haul lancia l'interprete come processo separato (`python -m gallery_dl ...`).

- **Perché.** Dalla rete non arriva nessun eseguibile, quindi niente quarantena, niente firma ad
  hoc da sperare che basti, niente autenticazione a ogni versione. Le versioni sono le stabili,
  disponibili lo stesso giorno della release. L'integrità si controlla due volte: lo SHA-256
  dell'API di PyPI e, come secondo livello, la firma GPG del manutentore su `SHA256SUMS` di
  Codeberg. L'aggiornamento di Haul (l'interprete) e quello di gallery-dl (le wheel) restano
  separati.
- **Come si sa che c'è una versione nuova**: `info.version` da `https://pypi.org/pypi/gallery-dl/json`,
  con lo SHA-256 di ogni file nella stessa risposta; si tiene la versione precedente finché la
  nuova non supera una prova (per esempio `--version`), così un aggiornamento rotto si annulla.

**Alternative scartate:**

- **Fonte-01, notturna di `gdl-org/builds`**: versioni di sviluppo non stabili, firma ad hoc senza
  Developer ID, nessuna firma del manutentore, architettura che nel flusso è dichiarata `x64` ma
  esce `arm64`, e dipendenza dall'infrastruttura GitHub colpita dalla segnalazione DMCA. Resta
  utile solo come canale sperimentale opzionale, se un giorno servisse.
- **Fonte-02, eseguibile stabile ufficiale per macOS**: non esiste.
- **Fonte-04, PyInstaller costruito da Haul**: funziona e dà la firma più solida, ma obbliga a una
  build, una firma e un'autenticazione a ogni versione di gallery-dl (circa una a settimana), con
  minuti di GitHub Actions su macOS, il certificato sulla macchina di build e il sorgente da
  pubblicare; in più l'utente riceve le correzioni in ritardo. Da tenere come riserva se la
  Fonte-03 inciampa nel runtime protetto.

**Rischi aperti:**

- ⚠️ **Python sotto il runtime protetto**: quali permessi speciali servano all'interprete firmato
  da Haul è **non verificato**; va provato in un prototipo, con firma e autenticazione vere.
- **Moduli compilati**: un aggiornamento futuro di gallery-dl o di una dipendenza che richieda un
  modulo compilato non si potrebbe scaricare; serve un controllo che accetti solo wheel
  `py3-none-any` e, altrimenti, aspetti un aggiornamento di Haul.
- **Chiave GPG del manutentore**: l'impronta `3E09 F590 ... 9D36 5A88` va confermata da una fonte
  del manutentore prima di metterla nell'app; verificare una firma OpenPGP da Swift o da Rust
  richiede una libreria in più (da scegliere).
- **Tempi di avvio e dimensione dell'interprete estratto**: non misurati; vanno misurati nel
  prototipo, accanto alla Fonte-01 come termine di paragone.
- **Quarantena e `com.apple.provenance`**: l'assenza di quarantena sui file scaricati da Haul è
  dedotta dalla documentazione, non provata; con la Fonte-03 conta poco, perché i file non sono
  eseguibili, ma va osservata.
- **Licenza**: le FAQ della FSF non sono state lette; da leggere prima della prima release, e da
  decidere se la prima wheel viaggia dentro il DMG (allora il sorgente va allegato).
- **ffmpeg**: nessuna fonte ufficiale per macOS; resta a carico dell'utente, con una guida.
- **Spostamenti del progetto**: gallery-dl ha già cambiato casa una volta nel 2026; PyPI è la
  fonte meno legata alla piattaforma che ospita il codice, ma Haul deve gestire con un messaggio
  chiaro il caso in cui la fonte non risponda.
