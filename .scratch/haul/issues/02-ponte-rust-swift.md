# Come parlano il nucleo in Rust e l'interfaccia in Swift

Type: research
Status: resolved
Blocked by:

## Question

Se il nucleo è in Rust e l'interfaccia in SwiftUI e AppKit, quali sono a novembre 2026 i modi
raccomandati di collegarli, e quale conviene a Haul? Almeno: UniFFI, `swift-bridge`, una libreria
C con intestazioni generate (`cbindgen`), e il nucleo come programma separato che parla con l'app
(processo figlio, XPC). Per ognuno, con fonte primaria e data:
- integrazione nella build di Xcode (Swift Package, XCFramework, script di build), compilazione
  universale e firma;
- chiamate asincrone e flussi di eventi dal nucleo all'interfaccia (avanzamento di un download,
  righe di registro), con Swift Concurrency;
- gestione degli errori e dei tipi condivisi;
- maturità, manutenzione e progetti reali che lo usano su macOS;
- costo per chi mantiene il progetto da solo.

Esito atteso: una strada consigliata, e se il beneficio del nucleo in Rust regge rispetto a un
nucleo in Swift (un dato per la decisione del ticket
[I confini fra nucleo e interfaccia](12-confini-del-nucleo.md), non la decisione).

## Answer

Ricerca completa in [ponte-rust-swift.md](../research/ponte-rust-swift.md) (2026-11-21).

- **Strada consigliata**: UniFFI di Mozilla, con le macro `#[uniffi::export]` e una libreria
  statica impacchettata in un XCFramework da uno script del repo, versione fissata esatta.
  Avanzamento e registro passano da un'interfaccia Swift che Rust chiama a ogni evento; per fermare
  un download serve un `cancel()` progettato nel nucleo.
- **Basta `arm64`**: macOS 27 gira solo su Mac con chip Apple.
- **Scartate**: `swift-bridge` (un solo autore, ferma alla `0.1.59`), `cbindgen` (tutto a mano), il
  nucleo come programma separato o XPC (un protocollo e un eseguibile in più).
- **Rischi**: UniFFI è ancora `0.x`; il codice asincrono generato non supera ancora i controlli di
  Swift 6 (`Sendable`); nessun esempio pubblico di app solo macOS con Developer ID costruita così,
  quindi build, firma e notarizzazione vanno provate presto.
- **Rust o Swift**: un confronto neutro chiude il rapporto, come dato per il ticket
  [I confini fra nucleo e interfaccia](12-confini-del-nucleo.md).
