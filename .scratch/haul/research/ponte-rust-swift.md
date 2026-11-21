# Come parlano il nucleo in Rust e l'interfaccia in Swift

> Ricerca per il ticket `issues/02-ponte-rust-swift.md`. Fonti lette il 2026-11-21, con la
> versione o la data indicate accanto a ognuna. Quello che non è stato possibile verificare è
> segnato come **non verificato**.

## 📖 Due parole prima di cominciare

- **Ponte** (in inglese FFI, foreign function interface): il codice che permette a un programma
  scritto in una lingua (Swift) di chiamare funzioni scritte in un'altra (Rust) dentro lo stesso
  processo, cioè dentro la stessa app in esecuzione.
- **Generatore di binding**: uno strumento che legge il codice Rust e scrive da sé il codice Swift
  del ponte, così nessuno lo scrive a mano.
- **Libreria statica** (`.a`): il codice Rust compilato, che viene incollato dentro l'eseguibile
  dell'app al momento della build. **XCFramework**: il contenitore di Apple che porta una libreria
  già compilata con le sue intestazioni, pronta per Xcode. **Swift Package**: il modo di Apple per
  distribuire e includere moduli in un progetto.
- **Swift Concurrency**: il sistema di Swift per il lavoro asincrono (`async`/`await`, i flussi
  `AsyncStream`, i controlli `Sendable` che impediscono di condividere dati in modo pericoloso fra
  thread).
- **Processo separato**: un secondo programma, lanciato dall'app, che parla con lei con messaggi.
  **XPC** è il meccanismo di Apple per questi messaggi fra processi.

## 🧭 Un fatto che semplifica tutto: macOS 27 è solo Apple silicon

- La pagina di compatibilità di macOS 27 elenca solo Mac con chip Apple (M1 e successivi, e
  MacBook Neo con A18 Pro) e dice che si aggiorna se si ha un Mac con Apple silicon
  ([Apple Support 127255](https://support.apple.com/en-us/127255), letta il 2026-11-21).
- Con il minimo a macOS 27.2, Haul si compila **per una sola architettura**, `arm64`. Non serve
  il binario universale (la fusione di due architetture con `lipo`), che negli esempi di UniFFI e
  di altri progetti occupa buona parte degli script di build.
- Il bersaglio Rust `aarch64-apple-darwin` è di **Tier 1 con strumenti host**, il livello di
  supporto più alto del compilatore Rust
  ([rustc platform support](https://doc.rust-lang.org/rustc/platform-support.html), letta il
  2026-11-21).

## 1. UniFFI (Mozilla)

Che cos'è: un generatore di binding di Mozilla che, partendo da annotazioni nel codice Rust
(macro `#[uniffi::export]`) o da un file di descrizione (UDL), scrive il codice per Swift, Kotlin,
Python e altre lingue.

- **Versione e attività**: ultima release `v0.32.2` del 2026-09-22; il changelog ha già una
  sezione non rilasciata con cambi incompatibili; ultimo commit sul ramo principale il
  2026-11-20 (`472cfc08a7`); circa 5.150 stelle
  ([CHANGELOG.md](https://github.com/mozilla/uniffi-rs/blob/main/CHANGELOG.md), letto il
  2026-11-21). La serie è ancora `0.x`: **ogni versione minore può rompere qualcosa**, e il
  changelog lo segnala in una sezione apposita.
- **Build in Xcode**:
  - Il binario `uniffi-bindgen-swift` genera separatamente i sorgenti `.swift`, le intestazioni
    `.h` e il `modulemap` (il file che dice a Swift come vedere la parte in C), anche nella forma
    adatta a un XCFramework
    ([uniffi-bindgen-swift.md](https://github.com/mozilla/uniffi-rs/blob/main/docs/manual/src/swift/uniffi-bindgen-swift.md),
    ramo principale al 2026-11-20).
  - Per un XCFramework il `modulemap` va rinominato `module.modulemap`
    ([module.md](https://github.com/mozilla/uniffi-rs/blob/main/docs/manual/src/swift/module.md)).
  - La pagina sull'integrazione con Xcode descrive una regola di build per i file `.udl`, la
    libreria statica aggiunta in `Link Binary with Libraries` e l'intestazione importata nel
    bridging header; non parla di Swift Package né di XCFramework
    ([xcode.md](https://github.com/mozilla/uniffi-rs/blob/main/docs/manual/src/swift/xcode.md)).
  - Il percorso pratico è quello di Apple: una libreria statica per piattaforma, impacchettata con
    `xcodebuild -create-xcframework -library ... -headers ...`, poi firmata con `codesign
    --timestamp`
    ([Creating a multiplatform binary framework bundle](https://developer.apple.com/documentation/xcode/creating-a-multi-platform-binary-framework-bundle),
    letta il 2026-11-21).
  - Strumento di contorno: `cargo-swift` crea il pacchetto Swift da un crate UniFFI, ma la sua
    tabella di compatibilità arriva a UniFFI `0.31.1` (`cargo-swift` `v0.11.1` del 2026-05-20), quindi
    **non copre ancora la `0.32`**
    ([README di cargo-swift](https://github.com/antoniusnaumann/cargo-swift), letto il 2026-11-21).
- **Firma**: la libreria statica finisce dentro l'eseguibile dell'app, quindi viene firmata e
  notarizzata con l'app stessa, senza passi in più. Questa è una deduzione dalla natura del
  collegamento statico e dalla guida Apple sulla notarizzazione (tutti gli eseguibili firmati,
  Hardened Runtime, timestamp sicuro:
  [Notarizing macOS software](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution),
  letta il 2026-11-21).
- **Chiamate asincrone**: una funzione Rust `async` diventa una funzione Swift `async`; è il lato
  Swift a far avanzare il lavoro, e non serve un motore asincrono in Rust
  ([futures.md](https://github.com/mozilla/uniffi-rs/blob/main/docs/manual/src/futures.md)). Per
  usare `tokio` (il motore asincrono più diffuso in Rust) si tiene un riferimento al motore e ci
  si appoggia a lui; l'attributo `uniffi::async_runtime` esiste ma il manuale dice che non è
  chiaro se resterà
  ([async-overview.md](https://github.com/mozilla/uniffi-rs/blob/main/docs/manual/src/internals/async-overview.md)).
  ⚠️ **La cancellazione non è supportata**: il manuale suggerisce un metodo `cancel()` scritto da
  sé (pagina `futures` del manuale pubblicato, letta il 2026-11-21).
- **Flussi di eventi** (avanzamento, righe di registro): UniFFI non ha un tipo flusso. Il modo
  documentato è il **tratto esterno** (foreign trait): un'interfaccia definita in Rust e
  implementata in Swift, che Rust chiama a ogni evento; il tratto deve essere `Send + Sync` e i
  parametri passano per valore
  ([foreign_traits.md](https://github.com/mozilla/uniffi-rs/blob/main/docs/manual/src/foreign_traits.md)).
  Sul lato Swift l'implementazione può riversare gli eventi in un `AsyncStream`
  ([AsyncStream](https://developer.apple.com/documentation/swift/asyncstream)). Questo schema è una
  composizione dei due documenti, **non un esempio ufficiale verificato**.
- **Swift 6 e `Sendable`**: supporto **parziale**. La maggior parte del codice generato è
  `Sendable`, ma il codice asincrono no; il difetto è seguito nella issue
  [#2448](https://github.com/mozilla/uniffi-rs/issues/2448), aperta al 2026-11-21
  ([overview.md](https://github.com/mozilla/uniffi-rs/blob/main/docs/manual/src/swift/overview.md)).
- **Errori e tipi condivisi**: un `Result<T, E>` di Rust, con `E` un `enum` che implementa
  `std::error::Error`, diventa in Swift una funzione che lancia un errore (`throws`); si possono
  usare anche oggetti come errori
  ([errors.md](https://github.com/mozilla/uniffi-rs/blob/main/docs/manual/src/types/errors.md)).
  Strutture (`Record`), enum e oggetti passano come tipi Swift veri, generati.
- **Progetti reali**:
  - `matrix-rust-sdk` (il nucleo di Element X): usa UniFFI `0.32.1`, ha 107 funzioni `pub async fn`
    nel solo `client.rs` dei binding, e il suo script XCFramework compila anche per
    `aarch64-apple-darwin`
    ([matrix-rust-sdk](https://github.com/matrix-org/matrix-rust-sdk/tree/main/bindings), letto il
    2026-11-21).
  - `bitwarden/sdk-internal`: UniFFI fissato a `=0.32.0`, con un crate `bitwarden-uniffi` e un
    `Package.swift`
    ([sdk-internal](https://github.com/bitwarden/sdk-internal), letto il 2026-11-21).
  - Un'app **solo macOS**, distribuita fuori dal Mac App Store, costruita con UniFFI: **non
    verificata** (gli esempi trovati sono soprattutto iOS).
- **Costo per chi è da solo**: basso nel codice (nessun ponte scritto a mano), medio nella
  manutenzione: aggiornare UniFFI vuol dire leggere i cambi incompatibili a ogni versione minore,
  e lo script di build (cargo, generatore, XCFramework) è da scrivere e tenere.

## 2. `swift-bridge`

Che cos'è: un generatore di binding pensato solo per Rust e Swift. Si descrive il confine in un
modulo Rust annotato (`#[swift_bridge::bridge]`), e lo strumento genera il lato Swift e il lato C.

- **Versione e attività**: ultima release `0.1.59` del 2026-01-06; ultimo commit il 2026-09-05;
  91 issue aperte; circa 1.150 stelle; un autore principale
  ([swift-bridge](https://github.com/chinedufn/swift-bridge), letto il 2026-11-21). Il numero di
  manutentori attivi **non è verificato** oltre l'autore.
- **Build in Xcode**: il libro ha capitoli per Xcode con cargo, per `swiftc` e per creare un Swift
  Package con una libreria statica (`crate-type = ["staticlib"]`); i pacchetti con binari valgono
  solo sulle piattaforme Apple
  ([libro, capitolo swift-packages](https://github.com/chinedufn/swift-bridge/tree/master/book/src/building),
  letto il 2026-11-21). Firma: come UniFFI, la libreria statica viaggia dentro l'app.
- **Chiamate asincrone**: `async` in entrambe le direzioni, da Swift verso Rust e da Rust verso
  Swift. La funzionalità `async` porta con sé `tokio` con il motore multi-thread
  ([Cargo.toml](https://github.com/chinedufn/swift-bridge/blob/master/Cargo.toml), versione
  `0.1.59`). La compatibilità con Swift 6 per il codice asincrono generato è arrivata con la
  `0.1.59` (commit del 2026-01-06, PR #362). Un tipo flusso pronto per gli eventi **non è
  documentato**: si usano callback o funzioni Swift chiamate da Rust.
- **Errori e tipi condivisi**: `Result<T, E>` diventa `throws`; le funzioni Swift chiamate da
  Rust che restituiscono un `Result` devono usare i typed throws (`throws(E)`); strutture ed enum
  trasparenti, con i campi visibili da entrambe le parti, e tipi opachi
  ([libro, funzioni](https://github.com/chinedufn/swift-bridge/blob/master/book/src/bridge-module/functions/README.md)).
- **Progetti reali su macOS**: **non verificati**.
- **Costo per chi è da solo**: API più vicina a Swift di UniFFI, ma un progetto con un solo
  autore, ancora a `0.1.x`, con un ritmo di release irregolare (agosto 2024, poi dicembre 2025):
  il rischio che si fermi ricade su chi lo usa.

## 3. Libreria C con intestazioni generate (`cbindgen`)

Che cos'è: `cbindgen` legge il codice Rust che espone funzioni in stile C (`extern "C"`) e ne
scrive le intestazioni `.h`. Swift può importare il C, quindi il ponte funziona, ma **tutto quello
che si costruisce sopra il C si scrive a mano**.

- **Versione e attività**: ultima release `0.29.4` del 2026-06-10
  ([cbindgen](https://github.com/mozilla/cbindgen), letto il 2026-11-21). Il README dice che lo
  sviluppo è stato fatto su misura delle esigenze dei manutentori e che qualche caso può non
  essere gestito.
- **Build e firma**: libreria statica più intestazioni, impacchettate in un XCFramework con
  `-library` e `-headers` (guida Apple citata sopra); firma con l'app.
- **Chiamate asincrone e flussi**: nessun supporto. Si scrivono a mano puntatori a funzione
  (callback in C), si gestiscono a mano la memoria delle stringhe e il passaggio fra thread, e si
  avvolgono in `async` e `AsyncStream` dal lato Swift.
- **Errori e tipi**: solo tipi C (numeri, puntatori, strutture semplici). Stringhe, liste ed
  errori si codificano a mano, per esempio come codici numerici o come testo JSON.
- **Progetti reali**: `cbindgen` è usato da Mozilla per Firefox (C++); un'app macOS con nucleo
  Rust e interfaccia Swift passata per `cbindgen`: **non verificata**.
- **Costo per chi è da solo**: il più alto. Ogni funzione nuova chiede codice di ponte in due
  lingue, con errori di memoria possibili e difficili da trovare.

## 4. Il nucleo come programma separato (processo figlio o XPC)

Che cos'è: il nucleo Rust diventa un eseguibile a sé, dentro il pacchetto dell'app. L'app lo
lancia e ci parla con messaggi (per esempio righe JSON sullo standard input e output), oppure
attraverso XPC.

- **Dove si mette e come si firma**: Apple indica `Contents/MacOS/` o `Contents/Helpers/` per
  uno strumento di aiuto, e `Contents/XPCServices/` per un servizio XPC
  ([Placing content in a bundle](https://developer.apple.com/documentation/bundleresources/placing-content-in-a-bundle),
  letta il 2026-11-21). Ogni eseguibile va firmato con Developer ID, con Hardened Runtime e
  timestamp sicuro, prima della notarizzazione (guida sulla notarizzazione citata sopra). Un
  eseguibile in più vuol dire un passo di firma in più nello script di rilascio.
- **Processo figlio**: lato Swift si usa `Process` di Foundation
  ([Process](https://developer.apple.com/documentation/foundation/process)) o il pacchetto
  `swift-subprocess`, costruito sulla Swift Concurrency, arrivato alla `1.0.0` il 2026-08-04 e alla
  `1.0.2` il 2026-10-16 ([swift-subprocess](https://github.com/swiftlang/swift-subprocess), letto
  il 2026-11-21). Gli eventi arrivano naturalmente come righe di testo, cioè come un flusso.
- **XPC**: le API Swift `XPCSession` e `XPCListener` esistono da macOS 14; un servizio XPC si crea
  in Xcode come target a sé
  ([XPCSession](https://developer.apple.com/documentation/xpc/xpcsession),
  [Creating XPC services](https://developer.apple.com/documentation/xpc/creating-xpc-services),
  lette il 2026-11-21). Queste API sono in Swift: un servizio XPC scritto in Rust dovrebbe usare
  la libreria C `libxpc` attraverso un ponte; una libreria Rust matura per XPC **non è
  verificata**.
- **Errori e tipi condivisi**: niente generatore. Il formato dei messaggi (per esempio JSON) si
  definisce in un posto e si duplica nei due linguaggi, o si genera da uno schema; ogni
  differenza fra le due copie si scopre solo quando l'app gira.
- **Vantaggio vero**: un crash del nucleo non chiude l'interfaccia, e il nucleo si prova da riga
  di comando senza Xcode. Haul, però, **lancia già un processo esterno** (`gallery-dl`): il nucleo
  separato aggiungerebbe un secondo processo in mezzo, con un protocollo in più da mantenere.
- **Costo per chi è da solo**: medio-alto. Niente ponte in memoria, ma un protocollo di messaggi
  da progettare, versionare e provare, e un eseguibile in più da firmare.

## 🧮 Il quadro in sintesi

| Strada | Build in Xcode | Asincrono ed eventi | Errori e tipi | Maturità | Costo da soli |
|---|---|---|---|---|---|
| UniFFI | XCFramework con script; `cargo-swift` indietro di una versione | `async` sì, cancellazione no, eventi via tratto esterno; Swift 6 parziale | generati, `throws` | `0.32.2`, Mozilla, uso in matrix e Bitwarden | basso nel codice, medio negli aggiornamenti |
| `swift-bridge` | Swift Package o Xcode con cargo | `async` nei due sensi con `tokio`; Swift 6 per l'async dalla `0.1.59` | generati, typed throws | `0.1.59`, un autore | basso nel codice, rischio di abbandono |
| `cbindgen` | XCFramework con script | tutto a mano | solo tipi C | `0.29.4`, Mozilla | alto |
| Processo separato | un target o un eseguibile in più, da firmare | flusso naturale di righe | protocollo duplicato a mano | strumenti Apple maturi | medio-alto |

## ✅ Raccomandazione

**Strada consigliata, se il nucleo resta in Rust: UniFFI**, con le macro (`#[uniffi::export]`),
una libreria statica solo `arm64`, un XCFramework costruito da uno script del repo con
`uniffi-bindgen-swift` e `xcodebuild -create-xcframework`, e la versione di UniFFI fissata
esattamente nel `Cargo.toml` (come fa Bitwarden con `=0.32.0`).

- Gli eventi (avanzamento, registro) passano per un tratto esterno implementato in Swift che li
  riversa in un `AsyncStream`; la cancellazione di un download è un comando esplicito del nucleo
  (`cancel()`), scelta che si concilia bene con la regola del progetto: la decisione di fermare
  vive nel nucleo, con le prove.
- È la strada con il produttore più solido (Mozilla), con il ritmo di rilascio più regolare e con
  progetti grandi che la usano oggi su piattaforme Apple.

**Alternative scartate e perché**

- `swift-bridge`: API gradevole, ma un solo autore, versione `0.1.x`, release rare; per un progetto
  mantenuto da una persona il rischio di restare con un ponte fermo pesa più dell'ergonomia.
- `cbindgen`: costringe a scrivere a mano proprio la parte più delicata (memoria, thread,
  stringhe, errori); è la scelta giusta per esporre una libreria C a molti linguaggi, non per
  un'app con un'interfaccia sola.
- Processo separato o XPC: il beneficio dell'isolamento è reale, ma Haul isola già il lavoro
  pesante dentro `gallery-dl`, che è un processo a sé; un secondo processo aggiunge un protocollo
  da duplicare e un eseguibile da firmare senza un guadagno chiaro.

**Rischi aperti**

- UniFFI è ancora `0.x`: ogni versione minore può rompere qualcosa; serve leggere il changelog a
  ogni aggiornamento (regola del progetto sulle dipendenze).
- Swift 6 in modalità rigorosa: il codice asincrono generato non è `Sendable` (issue #2448 aperta);
  potrebbero servire eccezioni locali nel lato Swift finché non viene risolta.
- Cancellazione non nativa: va progettata nel nucleo.
- `cargo-swift` non segue ancora la `0.32`: lo script di build si scrive e si tiene in casa.
- Nessuna app solo macOS con UniFFI e distribuzione Developer ID è stata verificata come esempio
  pubblico: la catena build, firma e notarizzazione va provata presto con un prototipo.
- Una build mista Rust e Swift richiede due toolchain (Rust e Xcode) anche nell'integrazione
  continua; il costo in minuti su macOS non è stato stimato qui.

**Nucleo in Rust o in Swift: dati per la decisione del ticket 12 (non la decisione)**

- **A favore di Rust**:
  - Il nucleo è separato dall'interfaccia per costruzione: il confine è il ponte, e le prove del
    nucleo girano con `cargo test` senza Xcode.
  - Un nucleo portabile, se un giorno servisse un'altra interfaccia (per esempio Android, che è la
    piattaforma mobile di riferimento delle regole del progetto); oggi Haul è solo macOS.
  - Bersaglio `aarch64-apple-darwin` di Tier 1.
- **A favore di Swift**:
  - Una lingua sola, una toolchain sola (Xcode), nessun ponte, nessun XCFramework, nessuno script
    di generazione, nessun aggiornamento di UniFFI da seguire.
  - Swift Concurrency usata direttamente, senza i limiti del codice generato (`Sendable`,
    cancellazione).
  - Il lavoro centrale di Haul (lanciare `gallery-dl`, leggerne l'output, scrivere il JSON di
    configurazione, aggiornare l'eseguibile) è coperto da strumenti Apple o del progetto Swift:
    `Process` o `swift-subprocess` `1.0.2`, `Codable` per il JSON, `URLSession` per lo scaricamento.
  - La separazione fra nucleo e interfaccia si ottiene anche in Swift, con un modulo del nucleo in
    un Swift Package e le prove in Swift Testing (Swift 6.0, Xcode 16:
    [Swift Testing](https://developer.apple.com/documentation/testing)); il confine però non è
    imposto dal linguaggio, va tenuto con disciplina.
- **Neutro o da misurare**: prestazioni (il collo di bottiglia probabile è la rete e
  `gallery-dl`, non il nucleo: **non misurato**); facilità per gli agenti AI di scrivere e
  provare ciascuna lingua (**non verificata** con dati).
