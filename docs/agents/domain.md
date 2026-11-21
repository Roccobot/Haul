# Domain docs

Come le skill di Matt Pocock leggono la documentazione del dominio di questo repo quando esplorano
il codice. Il repo ha un contesto solo (configurazione del 2026-11-21): anche con il nucleo e
l'interfaccia in due linguaggi, il dominio di Haul è uno.

## Prima di esplorare, si leggono

- **`GLOSSARY.md`** alla radice del repo;
- **`docs/adr/`**: le decisioni (ADR) che toccano l'area in cui si sta per lavorare.

Se questi file non esistono, **si va avanti in silenzio**: non si segnala che mancano e non si
propone di crearli in anticipo. Li crea `/domain-modeling` quando un termine o una decisione si
fissa davvero. Le regole del progetto restano in `AGENTS.md` e `Rules.md`, che vincono su questi
file dove parlano.

## Struttura

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── 0001-...md
│   └── 0002-...md
└── (le cartelle del codice, quando nascono)
```

## Si usa il vocabolario del glossario

Quando un testo nomina un concetto del dominio (il titolo di un ticket, una proposta, un'ipotesi,
il nome di una prova), usa il termine come lo definisce `GLOSSARY.md`, senza sinonimi che il
glossario esclude. Se il concetto nel glossario non c'è, è un segnale: o si sta inventando una
parola che il progetto non usa (si ripensa), o c'è un buco vero (lo si annota per
`/domain-modeling`).

## Si segnalano i contrasti con una decisione

Se una proposta contraddice una decisione già presa in `docs/adr/`, lo si dice apertamente, senza
scavalcarla in silenzio:

> _Contraddice ADR-0007 (...), ma vale la pena riaprirla perché..._
