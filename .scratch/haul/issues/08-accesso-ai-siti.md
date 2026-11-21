# L'accesso ai siti che lo chiedono

Type: grilling
Status: resolved
Blocked by: 03

## Question

Come entra Haul nei siti che vogliono un accesso, senza che chi non è esperto debba capire cookie e
token? Da decidere: quali modi offrire (cookie letti dal browser, credenziali, accesso OAuth dove
gallery-dl lo prevede); dove si conservano credenziali e token (il Portachiavi di macOS, mai in
chiaro nel `config.json`); che cosa vede l'utente quando un accesso scade; quali browser
supportare. Le possibilità reali le dice la ricerca
[Le opzioni di gallery-dl e il modo di guidarlo](03-opzioni-di-gallery-dl.md).

## Answer

Deciso con Rocco il 2026-11-21 (tre giri a scelta multipla, le sigle sono le sue risposte). I modi
che gallery-dl 1.32.21 offre sono in
[Le opzioni di gallery-dl e il modo di guidarlo](../research/opzioni-di-gallery-dl.md),
§ 'Come si autentica gallery-dl'.

- **Il modo lo decide Haul** (`Modo-01`): per ogni sito un solo comando, `Accedi`, che usa il modo
  migliore senza che si debba sapere che cosa sono cookie e token. L'ordine è OAuth, dove gallery-dl
  lo prevede (Pixiv, DeviantArt, Reddit, Tumblr, Flickr, SmugMug, Mastodon); poi i cookie del
  browser; poi nome utente e password, dove il sito li accetta.
- **Piano B** (`PianoB-01`): se il modo scelto smette di funzionare, Haul prova da solo il
  successivo nello stesso ordine e dice quale sta usando; se nessuno funziona, vale `Scaduto-01`.
- **Quando** (`Quando-01`): al bisogno, quando un indirizzo lo richiede (la raccolta aspetta e Haul
  chiede l'accesso), più una sezione `Account` nelle impostazioni con tutti i siti
  ([La finestra delle impostazioni](11-finestra-delle-impostazioni.md)). Un account per sito
  (`Account-01`): per cambiarlo si esce e si rientra.
- **Browser** (`Browser-01`): tutti quelli che gallery-dl legge; Haul guarda quelli installati e usa
  quello in cui si è già entrati nel sito, e se sono più d'uno chiede una volta.
- **Permessi di macOS** (`Permessi-01`): prima della richiesta di sistema una schermata spiega
  perché arriva (il Portachiavi per i browser Chromium; per Safari probabilmente l'accesso completo
  al disco), e per l'accesso completo al disco apre la pagina giusta delle Impostazioni di Sistema.
- **Custodia** (`Custodia-01`): password e token nel Portachiavi di macOS, mai in chiaro su disco;
  a ogni avvio Haul li passa a gallery-dl in un file temporaneo leggibile solo dall'utente, e poi
  lo cancella. I cookie del browser non si copiano: gallery-dl li legge ogni volta. La cache di
  gallery-dl è quella di Haul, separata da un gallery-dl installato a mano.
- **Accesso scaduto** (`Scaduto-01`): la raccolta va in pausa col messaggio `Accedi di nuovo a
  <sito>` e un pulsante; dopo l'accesso riprende da dove era, con una notifica se Haul non è in
  primo piano.
- **`config.json` importato** (`Import-02`): password, cookie e token vanno nel Portachiavi e
  spariscono dalla copia di Haul; il file originale non si tocca, e Haul suggerisce di cancellare le
  password da lì.
- **Resta alla costruzione**: verificare quale permesso chiede macOS per i cookie di Safari (nel
  sorgente non è accertato), e quale modo è il migliore per ciascuno dei siti di riferimento del
  ticket [Come si avvia e si segue un download](05-come-si-scarica.md).

## Comments

- 2026-11-21, primo giro con Rocco: `Modo-01` il modo migliore per ogni sito, deciso da Haul, con la
  domanda di un piano B quando smette di funzionare (da decidere nel giro dopo); `Quando-01` al
  bisogno, più una sezione Account nelle impostazioni; `Custodia-01` Portachiavi di macOS, passati a
  gallery-dl in un file temporaneo e mai in chiaro; `Scaduto-01` la raccolta va in pausa con
  'Accedi di nuovo' e riprende dopo l'accesso.
- Secondo e terzo giro: `PianoB-01` il modo successivo, provato da solo; `Browser-01` tutti, usando
  quello in cui si è già entrati; `Permessi-01` una spiegazione prima del consenso di macOS;
  `Import-02` credenziali spostate nel Portachiavi; `Account-01` un account per sito;
  `Chiusura-02` ticket chiuso.
