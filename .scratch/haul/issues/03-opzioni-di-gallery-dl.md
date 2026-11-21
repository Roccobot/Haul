# Le opzioni di gallery-dl e il modo di guidarlo

Type: research
Status: open
Blocked by:

## Question

Com'è fatto `docs/configuration.rst` di gallery-dl (circa 245 KB), e come lo trasforma Haul in una
finestra di impostazioni che resta aggiornata con le versioni nuove? E come guida Haul il programma
mentre scarica? Con fonte primaria (il sorgente di `mikf/gallery-dl`) e la versione letta:
- la struttura del file: sezioni, campi di ogni opzione (tipo, valore di serie, esempio,
  descrizione), quanto è regolare per un generatore, e se esiste uno schema leggibile da macchina;
- le opzioni per estrattore (sito) e quelle generali; quante sono, e quali sono le principali per
  chi scarica gallerie di immagini;
- le opzioni che decidono cartelle e nomi dei file (`base-directory`, `directory`, `filename`,
  `path-restrict` e simili) e la loro sintassi di formato, perché servono ai ticket
  [Cartelle e sottocartelle](06-cartelle.md) e [La rinomina automatica](07-rinomina.md);
- come si autentica gallery-dl (credenziali, `cookies`, `cookies-from-browser`, `oauth:`), per il
  ticket [L'accesso ai siti che lo chiedono](08-accesso-ai-siti.md);
- come si segue un download da un altro programma: uscita in JSON (`--dump-json`, `-j`),
  `--simulate`, livelli di registro, codici d'uscita, avanzamento, interruzione e ripresa,
  archivio dei file già scaricati (`--download-archive`);
- dove legge gallery-dl il `config.json` e come Haul può usare il proprio senza toccare quello
  dell'utente.

Esito atteso: un quadro che dica quali voci curare a mano e come generare le altre.

## Answer
