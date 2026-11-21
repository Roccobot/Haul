# Haul

L'app per macOS che dà un'interfaccia nativa a gallery-dl, il programma a riga di comando che
scarica immagini e gallerie da centinaia di siti.

## Language

**gallery-dl**:
Il programma esterno che fa gli scaricamenti; Haul lo procura, lo aggiorna e lo guida.
_Avoid_: motore, backend, CLI

**Configurazione guidata**:
La sequenza di passi del primo avvio che prepara Haul, partendo da zero o da un `config.json`
importato.
_Avoid_: onboarding, wizard, procedura iniziale

**Raccolta**:
Un indirizzo dato a Haul con tutti i file che ne vengono, scaricati in una cartella sola; è
l'unità della coda.
_Avoid_: task, download (come nome), attività, lavoro

**Radice**:
La cartella scelta dall'utente sotto cui Haul mette ogni raccolta, divisa per sito e proprietario;
è l'unica parte del percorso che l'utente decide.
_Avoid_: cartella base, cartella di download, destinazione

**Proprietario**:
Chi possiede o ospita una raccolta: l'autore o l'utente, oppure il gruppo o la ricerca che la
contiene quando un autore non c'è.
_Avoid_: autore (come termine generale), creatore, account

**Accesso**:
Il permesso che Haul ha di entrare in un sito come l'utente, uno per sito, comunque sia stato
ottenuto.
_Avoid_: login, autenticazione, credenziali (come termine generale), sessione

**Impostazioni avanzate**:
La parte della finestra delle impostazioni generata dalle opzioni di gallery-dl, accanto alle voci
curate a mano.
_Avoid_: opzioni esperte, configurazione completa
