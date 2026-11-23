# Mappa: la specifica di Haul 1.0

Label: wayfinder:map
Card: [Haul: mappa del progetto](https://trello.com/c/dSVsmAvA)

## Destination

Una specifica pronta per costruire Haul 1.0 (`.scratch/haul/spec.md`, con le decisioni difficili da
rovesciare in `docs/adr/`): l'app per macOS che dà un'interfaccia nativa a gallery-dl, con tutte e
sette le richieste di Rocco decise fino al punto in cui la costruzione parte senza dubbi aperti
(scelte `Meta-01` e `Ambito-01`, 2026-11-21).

## Notes

- **Le richieste di partenza** (Rocco, 2026-11-21, testo in `Rules.md` § '🧭 Che cos'è'):
  configurazione guidata al primo avvio, la finestra delle impostazioni al posto del JSON, cartelle
  e sottocartelle decise dall'app, rinomina automatica per certi tipi di download, piena aderenza
  al design di macOS 27.2 con icona in vero Liquid Glass, gallery-dl scaricato e aggiornato
  dall'app, un'app moderna e accattivante. Tutte e sette entrano nella 1.0 (`Ambito-01`): nella
  finestra delle impostazioni le voci principali sono curate a mano, le altre generate da
  `configuration.rst` in una sezione avanzata.
- **Già deciso nell'intervista di partenza** (2026-11-21):
  - distribuzione diretta, firmata con Developer ID e autenticata da Apple, scaricabile da GitHub,
    fuori dal Mac App Store (`Canale-01`,
    [ADR 0001](../../docs/adr/0001-direct-distribution-outside-app-store.md));
  - gratuita e open source, con licenza GPL-3.0 (`Prezzo-01`, `Licenza-01`);
  - interfaccia in italiano e inglese, che segue la lingua del sistema con l'inglese per le altre
    (`Lingue-01`), testi scritti con `/microcopy-it` e `/microcopy-en`;
  - tema chiaro e tema scuro dal sistema, senza un interruttore proprio (`Tema-01`), e il colore
    d'accento di sistema (`Accento-01`);
  - SemVer da 0.1.0, con la 1.0.0 alla prima versione pubblica, una fonte sola nel progetto Xcode
    letta anche dal nucleo, e il numero di build separato per gli aggiornamenti (`Versione-01`);
  - la configurazione guidata serve chi non conosce gallery-dl e chi lo usa già: due strade al primo
    avvio, partire da zero o importare il proprio `config.json` (`Utente-03`);
  - l'icona: la sessione propone dei bozzetti, da cui parte Rocco (`Icona-02`);
  - minimo macOS 27.2, dalla richiesta di Rocco.
- **La proposta di partenza sull'architettura** (`AGENTS.md`, § '🧭 Il nucleo di `Haul`'): nucleo in
  Rust, interfaccia in SwiftUI e AppKit, gli unici che danno il Liquid Glass vero. La mappa la
  conferma o la cambia nel ticket [I confini fra nucleo e interfaccia](issues/12-confini-del-nucleo.md).
- **Dominio**: il vocabolario è in `GLOSSARY.md`.
- **Skill da consultare**: `grilling` e `domain-modeling` per i ticket di conversazione,
  `prototype` per quelli di forma, `research` per quelli di ricerca. Le conversazioni sono in
  italiano, con le domande a scelta multipla e la proposta consigliata per prima. **Una voce a cui
  Rocco non risponde è approvata come proposta** (`rules/Roccobot.md` § '🧭 Un progetto nuovo
  grande o complesso: si propone `/wayfinder`').
- **Nessuna scelta di design senza Rocco**: forma, icona, comportamento visibile e testi si
  propongono e si chiedono.
- ⚠️ **I risultati delle ricerche vivono in `.scratch/haul/research/`**, accanto alla mappa e
  committati su `main`, e non su un branch `research/<nome>`: così la mappa li cita con un
  collegamento che resta valido. Le decisioni si raccolgono in pochi commit.

## Decisions so far

- La destinazione, il canale di distribuzione, il prezzo e la licenza, le lingue, tema e accento,
  la versione, il pubblico della configurazione guidata e l'icona: decisi da Rocco
  nell'intervista di partenza del 2026-11-21 (Notes, sopra).
- [Da dove prende Haul l'eseguibile di gallery-dl](issues/01-fonte-di-gallery-dl.md): proposta
  `Fonte-03`, un Python incluso e firmato con l'app e gallery-dl scaricato da PyPI come codice
  Python puro, senza eseguibili dalla rete; lo sviluppo di gallery-dl è passato su Codeberg, e
  per macOS non esiste un eseguibile stabile ufficiale. La conferma è di Rocco.
- [Come parlano il nucleo in Rust e l'interfaccia in Swift](issues/02-ponte-rust-swift.md):
  UniFFI, con un XCFramework fatto da uno script del repo; basta `arm64`; da provare presto con
  firma e notarizzazione vere.
- [Le opzioni di gallery-dl e il modo di guidarlo](issues/03-opzioni-di-gallery-dl.md): 654
  chiavi in `configuration.rst` 1.32.21, dodici gruppi da curare a mano, il resto generato in un
  catalogo con una prova a ogni versione; gallery-dl si guida con un `config.json` di Haul isolato
  e un'uscita a righe con prefissi propri.
- [Firma, autenticazione e aggiornamenti di Haul](issues/04-firma-e-aggiornamenti.md): Developer
  ID, notarizzazione e Sparkle 2 da un flusso di Actions avviato da un tag e approvato da Rocco;
  99 USD l'anno ad Apple, il resto gratuito.
- [Come si avvia e si segue un download](issues/05-come-si-scarica.md): la **raccolta** (un
  indirizzo, una cartella) entra in coda da campo, trascinamento, menu Condividi o appunti; due
  alla volta, con `Ferma`, `Riprendi`, `Annulla`; avanzamento per file e totale solo se noto; due
  schede, `Download in corso` e `Raccolte scaricate` (7 giorni di serie); archivio di gallery-dl
  locale, con esporta e unisci fra Mac.
- [Cartelle e sottocartelle dei download](issues/06-cartelle.md): radice `~/Downloads/Haul`, l'unica
  cosa che si cambia; albero fisso `Radice/Sito/Proprietario/Raccolta` scritto da Haul per ogni
  sito, con l'albero del sito dentro la raccolta; profili nella sezione (`Gallery`, `Posts`);
  doppioni saltati e contati; caratteri di Windows cancellati, nomi a 80 caratteri; coda ferma se
  il disco manca.
- [La rinomina automatica](issues/07-rinomina.md): criteri fissi, solo per i file nuovi;
  `Titolo NN`, e `nomeCartella ###` per i nomi senza titolo o in altri alfabeti; pulizia di tag,
  emoji, sigle e conteggi; tipografia e caratteri vietati cancellati; taglio a 120 caratteri;
  identificativo in fondo agli omonimi; schema proprio per NSFWAlbum.
- [L'accesso ai siti che lo chiedono](issues/08-accesso-ai-siti.md): un `Accedi` per sito col modo
  scelto da Haul (OAuth, poi cookie del browser, poi password) e il successivo come piano B; al
  bisogno e nella sezione `Account`, un account per sito; tutto nel Portachiavi; pausa e
  `Accedi di nuovo` quando scade.
- [La configurazione guidata](issues/09-configurazione-guidata.md): assistente con la colonna dei
  passi a sinistra; due strade (da zero, o importando il `config.json` con un resoconto); gallery-dl
  preparato in sottofondo; notifiche saltabili; nessun passo per gli accessi; si rifà dal menu.
- [I bozzetti dell'icona in Liquid Glass](issues/10-icona.md): un gancio da carico che solleva
  un'immagine, infilato in un buco della foto, su piastrella indaco; il disegno finale è di Rocco.

## Not yet specified

- **Icona nel Dock e barra dei menu**: quanto Haul resta visibile mentre lavora (la notifica di
  fine raccolta è decisa).
- **Video e formati speciali**: `yt-dlp` arriva da PyPI con gallery-dl, mentre `ffmpeg` non ha una
  fonte ufficiale per macOS; resta da decidere come Haul guida l'utente a procurarlo.
- **I permessi delle cartelle**: a chi chiede macOS il consenso quando a scrivere è gallery-dl, e
  come lo spiega Haul; da osservare nel primo prototipo.
- **Errori e registro**: come si mostrano a chi non conosce gallery-dl un sito che cambia e un
  limite di richieste (l'accesso scaduto è deciso).
- **Prove automatiche e collaudo**: quali prove per nucleo e interfaccia, dove girano (il repo è
  pubblico, quindi Actions su macOS non si paga), e il documento di feedback del collaudo.
- **Accessibilità**: VoiceOver, tastiera, contrasto, testo più grande.
- **Il `README.md` bilingue** e la pagina da cui si scarica l'app.

## Out of scope

- **Il Mac App Store**: escluso da `Canale-01`, perché la sandbox vieta di scaricare ed eseguire
  gallery-dl dopo l'installazione.
- **Pagamenti e licenze a pagamento**: escluso da `Prezzo-01`.
- **Altre piattaforme** (iOS, iPadOS, Windows, Linux, Android): Haul è un'app per macOS.
- **Un motore di download proprio al posto di gallery-dl**: Haul ne è l'interfaccia.
