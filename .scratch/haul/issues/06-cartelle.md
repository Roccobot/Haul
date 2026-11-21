# Cartelle e sottocartelle dei download

Type: grilling
Status: resolved
Blocked by: 03

## Question

Dove salva Haul, e con quale albero di sottocartelle? La richiesta di Rocco: la cartella di
download e tutte le sottocartelle le stabilisce l'app. Da decidere: la cartella di serie (per
esempio `~/Downloads/Haul`) e come si cambia; lo schema delle sottocartelle (per sito, per autore o
galleria, per data); se lo schema si può cambiare e quanto; che cosa succede a un download ripetuto
(archivio dei file già scaricati); i caratteri non validi nei nomi su macOS. Le possibilità reali le
dice la ricerca [Le opzioni di gallery-dl e il modo di guidarlo](03-opzioni-di-gallery-dl.md).
Un vincolo già deciso in [Come si avvia e si segue un download](05-come-si-scarica.md): una
raccolta corrisponde a una cartella sola; gli esempi si prendono dai siti di Rocco elencati là.

## Answer

Deciso con Rocco il 2026-11-21 (tre giri a scelta multipla, le sigle sono le sue risposte). Schemi
di serie di gallery-dl letti sul sorgente della 1.32.21 (`directory_fmt` degli estrattori): Reddit
`{category}/{subreddit}`, Pixiv `{category}/{user[id]} {user[account]}`, Kemono
`{category}/{service}/{user}`, Bunkr (lolisafe) `{category}/{album_name} ({album_id})`, DeviantArt
`{category}/{username}` con le cartelle della galleria sotto. Sono diversi da sito a sito, quindi
lo schema è di Haul.

- **Radice** (`Radice-01`): `~/Downloads/Haul` di serie, proposta nella configurazione guidata e
  cambiabile lì e nelle impostazioni. È **l'unica cosa che si cambia** (`Cambio-01`): niente
  modelli né formato libero per le sottocartelle, e `base-directory` e `directory` restano fuori
  dalle impostazioni avanzate (ticket [La finestra delle impostazioni](11-finestra-delle-impostazioni.md)).
- **Albero fisso** (`Schema-01`, `Nome-01`): `Radice/Sito/Proprietario/Raccolta`, sempre uguale;
  **dentro** la cartella della raccolta resta l'albero che il sito prevede (sottocartelle comprese,
  per esempio le cartelle di una galleria DeviantArt). Haul scrive per ogni sito la propria lista
  `directory`, che prende i campi giusti di quel sito per i primi tre livelli e accoda quelli che
  gallery-dl aggiunge oltre. Una raccolta corrisponde così a una cartella sola, come vuole
  [Come si avvia e si segue un download](05-come-si-scarica.md).
- **Proprietario** (`Proprietario-01`): l'autore o l'utente; dove manca, chi ospita la raccolta
  (il subreddit, il tag di una ricerca); dove non c'è nessuno, come un album Bunkr, il livello si
  toglie (`Bunkr/Album`).
- **Nome della raccolta**: il nome o l'identificativo che il sito dà alla raccolta (album, galleria,
  cartella). Per un **profilo intero** è la sezione, col nome inglese del sito (`Artworks`,
  `Gallery`, `Posts`, `Favourites`), che non cambia con la lingua del Mac (`Profilo-01`). Un **post
  singolo** va nella cartella della sezione del suo profilo (`Singolo-01`), così un profilo
  scaricato dopo lo trova e lo salta.
- **File già presenti** (`Doppioni-01`): si saltano (`skip: true`, insieme all'archivio di
  `Archivio-01`), e alla fine della raccolta Haul dice quanti file c'erano già.
- **Caratteri** (`Caratteri-01`, corretto da `Vietati-01` in
  [La rinomina automatica](07-rinomina.md)): i caratteri vietati da Windows, per nomi validi anche
  su dischi exFAT e su un NAS, si **cancellano**, e `|` diventa `-`; fino al 2026-11-21 la scelta
  era sostituirli con `_`.
- **Lunghezza** (`Lunghezza-01`): ogni nome di cartella preso da un titolo si taglia a 80
  caratteri, sotto il limite di 255 byte di macOS anche con kanji e accenti.
- **Disco assente** (`Disco-01`): se la radice è su un disco non collegato, la coda si ferma e Haul
  lo dice; riparte quando il disco torna o quando si sceglie un'altra radice.
- **`config.json` importato** (`Import-01`): si tiene la sua radice, lo schema delle sottocartelle
  diventa quello di Haul, e la configurazione guidata lo dice.
- **Resta alla costruzione**: la tabella dei campi per sito (quale campo è il proprietario, quale la
  raccolta, quali sezioni) per i siti di riferimento del ticket 05, verificata con `gallery-dl -K`
  su un indirizzo vero di ciascuno.

## Comments

- 2026-11-21, primo giro con Rocco: `Radice-01` `~/Downloads/Haul` di serie, da cambiare nella
  configurazione guidata; `Schema-01` Sito/Autore o utente/Raccolta, uno schema di Haul uguale per
  tutti i siti; `Cambio-01` si cambia solo la radice; `Doppioni-01` un file già presente si salta,
  e alla fine Haul dice quanti file della raccolta c'erano già.
