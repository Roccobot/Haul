# Come parlano il nucleo in Rust e l'interfaccia in Swift

Type: research
Status: open
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
