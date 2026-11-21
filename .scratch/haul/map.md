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

## Not yet specified

- **La finestra principale**: com'è fatta mentre scarica, la coda, la cronologia dei download, le
  anteprime; dipende da [Come si avvia e si segue un download](issues/05-come-si-scarica.md).
- **Notifiche, icona nel Dock, barra dei menu**: quanto Haul resta visibile mentre lavora.
- **Video e formati speciali**: se e come Haul procura `ffmpeg` o `yt-dlp`, che gallery-dl usa per
  alcuni siti; dipende dalla fonte dell'eseguibile.
- **Errori e registro**: come si mostrano a chi non conosce gallery-dl un sito che cambia, un
  accesso scaduto, un limite di richieste.
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
