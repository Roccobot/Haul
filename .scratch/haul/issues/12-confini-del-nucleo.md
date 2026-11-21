# I confini fra nucleo e interfaccia

Type: grilling
Status: open
Blocked by: 01, 02, 03

## Question

Il nucleo resta in Rust? E che cosa vive nel nucleo e che cosa nell'interfaccia? La regola
inderogabile chiede le decisioni nel nucleo, con le prove, e l'interfaccia che le esegue. Da
decidere con Rocco, coi risultati delle tre ricerche: il linguaggio del nucleo (Rust come proposto,
o Swift se il ponte costa più di quanto rende); le responsabilità del nucleo (modello delle
impostazioni e generazione del `config.json`, coda dei download, gestione del processo di
gallery-dl e lettura del suo avanzamento, aggiornamento dell'eseguibile, regole di cartelle e
rinomina); dove vive il codice nel repo; come si prova il nucleo senza interfaccia. In più si
conferma la fonte di gallery-dl proposta dalla ricerca (`Fonte-03`: Python incluso e gallery-dl da
PyPI) e si decide se un primo prototipo tecnico (build UniFFI, Python incluso, firma e
notarizzazione vere) precede la specifica, visto che tre ricerche lo chiedono.

## Answer
