# La rinomina automatica

Type: grilling
Status: resolved
Blocked by: 03

## Question

Quali tipi di download rinomina Haul da solo, e con quali criteri? Da decidere con Rocco: che cosa
vuol dire 'tipo di download' (un sito, un genere di contenuto come immagini, video o fumetti, un
formato di file); per ognuno lo schema del nome (titolo, autore, data, numero progressivo, id
originale); se i criteri sono fissi, scelti da un elenco o scrivibili; se la rinomina vale anche
per file già scaricati. Le possibilità reali le dice la ricerca
[Le opzioni di gallery-dl e il modo di guidarlo](03-opzioni-di-gallery-dl.md).

## Answer

Deciso con Rocco il 2026-11-21 (cinque giri a scelta multipla, le sigle sono le sue risposte).
Nomi di serie di gallery-dl letti sul sorgente della 1.32.21 (`filename_fmt`): quasi tutti numerici
(Pixiv `{id}_p{num}`, X `{tweet_id}_{num}`, Instagram `{media_id}`); NSFWAlbum ha i campi `studio`,
`models` e `title`. I predefiniti di NameMangler di Rocco, **esempi** delle sostituzioni che fa
spesso e non regole da copiare, sono riassunti per genere in
[I predefiniti di rinomina di NameMangler](../research/predefiniti-namemangler.md).

**Principio**: la rinomina è un insieme di criteri **fissi** decisi da Haul (`Regole-01`), senza
impostazioni, che valgono per i soli file nuovi (`Esistenti-01`). I 'tipi di download' sono casi
riconosciuti dal nome (`Tipo-01`), non siti o formati.

1. **Base** (`Base-01`): `Titolo NN.ext`, col numero del file nel post.
2. **Senza titolo** (`Senzatitolo-01`): l'inizio del testo del post (X, Instagram, Tumblr),
   ripulito; senza testo, `nomeCartella ###.ext`.
3. **Alfabeti non latini** (`Alfabeti-01`): un nome del tutto in un altro alfabeto (cirillico,
   greco, arabo, ebraico, cinese, giapponese, coreano) diventa `nomeCartella ###.ext`. Il
   `nomeCartella` è il nome della cartella della raccolta; se è straniero anche quello, il nome del
   proprietario, poi quello del sito (`Cartella-01`); le cartelle restano come sono. Il `###` ha
   tre cifre e continua dal numero più alto già nella cartella (`Numeri-01`).
4. **Pulizia dei titoli** (`Pulizia-01`, `Generali-01`, `Generali-02`): si tolgono flair, emoji,
   emoticon (`:)`, `<3`), caratteri di altri alfabeti, tag fra parentesi quadre (`[F]`, `(OC)`),
   hashtag, menzioni e link, domini (`.com`), sigle come `OC`, `POV`, `4K`, `HD`, `WIP`, risoluzioni
   e conteggi (`4000px`, `120 pics`, `1.2 GB`), parentesi rimaste vuote e punteggiatura ripetuta;
   il tutto maiuscolo diventa normale. Le lettere latine con segni (`ł`, `č`, `ő`) restano
   (`Latino-01`).
5. **Tipografia** (`Generali-01`): apici e virgolette curve diventano `'`, trattini lunghi e tilde
   diventano `-`, le parti del nome si separano con ` - ` (il trattino dentro una parola resta), i
   nomi dei siti prendono le maiuscole giuste (`TikTok`, `YouTube`, `DeviantArt`, `OnlyFans`).
6. **Caratteri vietati** (`Vietati-01`): `\ / : * ? " < > #` e simili si cancellano, `|` diventa
   `-`; vale per file e cartelle, e sostituisce il `_` deciso prima in
   [Cartelle e sottocartelle dei download](06-cartelle.md).
7. **Maiuscole** (`Inglese-01`): restano quelle del titolo; nessuna regola propria dell'inglese
   (Title Case, contrazioni, `i` che diventa `I`).
8. **Lunghezza** (`Lunghezza-02`): oltre 120 caratteri il titolo si taglia all'ultima parola
   intera; numero e identificativo restano.
9. **Omonimi** (`Omonimi-01`, `Omonimi-02`): se il nome è già preso da un altro file, in fondo si
   aggiunge l'identificativo del post, senza parentesi (`OC 01 abc123.jpg`). Così un file nuovo
   non è scambiato per un doppione da saltare.
10. **NSFWAlbum** (`Nsfwalbum-01`, `Nsfwalbum-02`): `Studio - Anna & Bea - Titolo 001.jpg` dai
    campi di gallery-dl, con le parti vuote tolte; la cartella della raccolta ha lo stesso schema,
    e il proprietario è lo studio.
11. **In più** (`Altro-01`): estensioni uniformi e minuscole (`JPEG`, `jpe`, `jfif` diventano
    `jpg`, `m4v` diventa `mp4`); nomi incollati separati (`WhiteFluffyDog` diventa
    `White Fluffy Dog`).

**Resta alla costruzione**: l'ordine in cui le regole si applicano, i campi di testo e di flair per
sito (`link_flair_text` di Reddit è da verificare con `gallery-dl -K`), e una prova per ogni regola
con titoli veri dei siti di riferimento.

## Comments

- 2026-11-21, primo giro con Rocco: i 'tipi' sono casi riconosciuti dal nome (`Tipo-01`, risposta
  libera): nome oltre 120 caratteri → accorciato tenendo una parte significativa e univoca; nome
  caotico senza schema (Reddit) → via le emoji, il minimo indispensabile; nome in un alfabeto
  straniero (cirillico) → `nomeCartella ###.ext`; NSFWAlbum → sempre e solo
  `Nome studio - Modella/e - Titolo del set`; e altri casi da proporre. `Base-01` titolo e numero;
  `Regole-01` criteri fissi decisi da Haul; `Esistenti-01` solo i file nuovi.
- Secondo giro: `Omonimi-01` l'identificativo in fondo, senza parentesi (risposta libera);
  `Alfabeti-01` tutti gli alfabeti non latini, giapponese compreso; `Cartella-01` se il nome della
  raccolta è straniero vale quello del proprietario, poi quello del sito, e le cartelle restano come
  sono; `Numeri-01` tre cifre che continuano dal numero più alto già nella cartella.
- Terzo giro: `Omonimi-02` l'identificativo si aggiunge solo se il nome è già preso;
  `Pulizia-01` dai titoli scritti dagli utenti si tolgono flair, emoji, caratteri non latini, tag
  fra parentesi quadre, hashtag, menzioni e link, punteggiatura ripetuta, e il maiuscolo urlato
  diventa normale (risposta libera, con flair e non latini aggiunti da Rocco);
  `Senzatitolo-01` l'inizio del testo del post, poi `nomeCartella ###`; `Lunghezza-02` titolo
  tagliato all'ultima parola intera, con numero e, se serve, identificativo.
- Quarto giro: `Nsfwalbum-01` `&` fra le modelle, parti vuote tolte; `Nsfwalbum-02` cartella con
  lo stesso schema, e proprietario lo studio; `Altro-01` estensioni uniformi e nomi incollati
  separati, più i predefiniti di rinomina di NameMangler che Rocco passa: il ticket resta aperto
  finché non arrivano.
- Quinto giro, coi predefiniti arrivati (esempi, non regole da copiare): `Generali-01` tipografia,
  separatore ` - ` e nomi dei siti, e le sigle (`OC`, `POV`, `4K`...) tolte del tutto (risposta
  libera); `Generali-02` domini, parentesi vuote, risoluzioni e conteggi, emoticon tolti;
  `Vietati-01` caratteri vietati cancellati e `|` che diventa `-`; `Inglese-01` nessuna regola
  propria dell'inglese; `Latino-01` le lettere latine con segni restano; `Chiusura-01` ticket chiuso.
