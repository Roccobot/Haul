# Le opzioni di gallery-dl e il modo di guidarlo

Type: research
Status: resolved
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

Ricerca completa in [opzioni-di-gallery-dl.md](../research/opzioni-di-gallery-dl.md), letta su
gallery-dl 1.32.21 (2026-11-21, commit `ceeb5bc6`).

- **`configuration.rst`**: 245.137 byte, 646 titoli di voce, 654 chiavi (84 generali, 416 per un
  solo sito su 106 categorie), contati con uno script. Formato abbastanza regolare, nessuno schema
  leggibile da macchina.
- **Generazione**: un generatore nel nucleo legge il file della versione inclusa e scrive un
  catalogo JSON; ogni tipo diventa un controllo (interruttore, numero, menu, testo, elenco,
  cartella, formato con anteprima, editor JSON per il resto). Descrizioni avanzate in inglese, voci
  curate con testi scritti a mano. Una prova automatica a ogni versione nuova; le chiavi che il
  catalogo non conosce si conservano.
- **Voci da curare a mano**: dodici gruppi (cartella, sottocartelle, nome dei file, caratteri nei
  nomi, file già presenti, archivio dei file scaricati, accesso, garbo verso i siti, che cosa
  scaricare, data dei file, metadati, `videos` e `include` per sito).
- **Come si guida**: `gallery-dl --config-ignore -c <file di Haul> <URL>`, con cache e cartella di
  base spostate in Haul, `extractor.input: false`, uscita a righe con prefissi propri, codice
  d'uscita come somma di bit, anteprima con `-j`, campi della rinomina con `-K`, annullamento con
  `SIGINT` (i `.part` permettono la ripresa).
- **Rischi**: la documentazione non è un contratto e non sempre coincide col codice; `-u` e `-p`
  rendono visibili le password; condizioni e formati speciali eseguono codice Python, quindi
  importare una configurazione altrui vuol dire fidarsi.
