# I predefiniti di rinomina di NameMangler

> Ricerca per il ticket [La rinomina automatica](../issues/07-rinomina.md) (`Altro-01`). Fonte: i sei
> predefiniti `.nma` di Rocco (plist binari), letti per intero. Sono **esempi delle sostituzioni che
> fa spesso**, non regole da copiare: qui si raccolgono i **generi di intervento** che ricorrono.
> Le regole si applicano in ordine; gli esempi prima/dopo sono costruiti, non presi da file veri.

## 📦 I sei predefiniti

| predefinito | regole | a che cosa serve (dedotto) |
|---|---|---|
| `DL Grabber` | 83 | file dei booru (rule34, konachan, yande.re, danbooru, derpibooru) col nome fatto di tag `Personaggio (Serie) + Personaggio (Serie)`, forse scaricati con Imgbrd-Grabber |
| `DL Img:Social` | 60 | titoli di post da siti social e d'arte (DeviantArt, ArtStation, Newgrounds, Imgur) |
| `DL NSFWalbum (cartelle)` | 119 | nomi di **cartelle** di set fotografici: studio, modelle, titolo |
| `DL Patreon` | 26 | post di Patreon: `id_titolo_numero` e numeri fra parentesi in testa |
| `DL Reddit` | 35 | titoli di post di Reddit, con l'identificativo in testa da togliere |
| `DL Roccobot` | 67 | il predefinito generico: pulizia, maiuscole, volumi, capitoli, parti |

Il conteggio è quello degli elementi di ogni plist. La prima regola di `DL Grabber` è un cambio di
maiuscole (`Task` 2, codice 4, probabilmente Title Case: da verificare); in `DL Reddit` c'è una
numerazione in sequenza (`Task` 110).

## 🟢 Generi generali: candidati a regola fissa di Haul

Fra parentesi, quanti predefiniti su sei usano quel genere.

1. **Apici e virgolette tipografiche → apice dritto** (6, tutti): backtick, acuto, virgolette
   curve e caporali diventano `'`. `Lara’s “new” set` → `Lara's 'new' set`
2. **Trattini lunghi, trattini a larghezza piena e tilde → `-`** (6, tutti; quattro trattano così
   anche `|`). `Tokyo ～ Night` → `Tokyo - Night`
3. **Separatore fra parti sempre ` - `** (5, tutti salvo `Reddit`), lasciando il trattino dentro
   una parola: `(?<=[^ -])(?!\b-\b)(?:\s*\-+\s*)+(?=[^ -])`. `Studio--Anna -Titolo` →
   `Studio - Anna - Titolo`, mentre `Rent-a-Girlfriend` resta com'è.
4. **Spazi multipli ridotti a uno, `_` come spazio** (6, tutti; `Img:Social`, `NSFWalbum` e
   `Roccobot` cambiano ogni `_`, gli altri solo quelli ripetuti). `my__cat   photo` → `my cat photo`
5. **Bordi puliti**: via spazi, punti, virgole, punto e virgola, trattini e `_` in testa e in coda
   (6, tutti, spesso due volte). `- Titolo del set.` → `Titolo del set`
6. **Spazio ideografico (U+3000) → spazio** (4: `Img:Social`, `Patreon`, `Reddit`, `Roccobot`).
7. **Caratteri non Latin-1 tolti** con `[^\x00-\xFF]`: emoji, simboli, alfabeti non latini (4:
   `Img:Social`, `Patreon`, `Reddit`, `Roccobot`). `Sunset 🌅 vibes` → `Sunset vibes`
8. **Caratteri riservati tolti**: `% $ € £ : ? \ / * < > ^ @ # §` e varianti (5, tutti salvo
   `Patreon`). `Who? Me: 100%` → `Who Me 100`
9. **Punteggiatura ripetuta ridotta a un segno** (4: `Img:Social`, `Patreon`, `Reddit`,
   `Roccobot`). `Wow!!! Finally...` → `Wow! Finally.`
10. **Spazi attorno a punteggiatura e parentesi**: niente spazio dopo `(` e prima di `)` `,` `!`,
    uno spazio dopo `,` `;` `!` `?`, maiuscola dopo `.` `!` `?` e ` -` (4: `Img:Social`,
    `NSFWalbum`, `Patreon`, `Roccobot`). `( hello ),yes!ok` → `(hello), yes! Ok`
11. **Parentesi vuote tolte** (4: `Grabber`, `NSFWalbum`, `Patreon`, `Roccobot`).
    `Titolo ( ) [ ]` → `Titolo`
12. **Prima lettera maiuscola** (3: `Img:Social`, `Reddit`, `Roccobot`; `Patreon` dopo ` - `,
    `Grabber` col Title Case). `first try` → `First try`
13. **Tag di Reddit tolti**: `[F]`, `(M)`, `[F23]`, `(f 21)`, `[OC]`, `(gif)` (3: `Img:Social`,
    `Reddit`, `Roccobot`). `Morning coffee [F23] (OC)` → `Morning coffee`
14. **Emoticon tolte**: `;)`, `:D`, `:-)`, `:O`, `<3` (3: gli stessi). `Happy Friday :D <3` →
    `Happy Friday`
15. **Parole di servizio tolte**: `animated`, `commission`, `sketch`, `nsfw`, `(un)censored`,
    `preview`, `link below`, `patreon only` (3: gli stessi). `Dragon (commission)` → `Dragon`
16. **Sigle in maiuscolo**: `3d 4k hq hdr pov oc wip nsfw vs ok lol` e simili, con elenchi quasi
    uguali (3: `Grabber`, `Img:Social`, `Roccobot`). `my 4k pov wip` → `my 4K POV WIP`
17. **Marchi con le maiuscole giuste** (5, tutti salvo `NSFWalbum`, che ha gli studi): `TikTok`
    (3), `OnlyFans`, `YouTube`, `ArtStation`, `DeviantArt`, `RedGifs` (2 ciascuno), `PornHub`,
    `Imgur`, `Patreon`, `Ko-Fi`, `D.Va`. `tiktok from youtube` → `TikTok from YouTube`
18. **Domini tolti**: `.com`, `.net`, `.org`, `.it`, `.co.uk`, `.jp`, `.ru`, `.tv` (3: `Grabber`,
    `NSFWalbum`, `Roccobot`). `hegre.com Anna` → `hegre Anna`
19. **Contrazioni inglesi ricostruite**: `Lets` → `Let's`, `ur` → `your`, `im` → `I'm`, `dont` →
    `don't`, `i` → `I` davanti a un verbo (4: `Grabber`, `Img:Social`, `NSFWalbum`, `Roccobot`).
    `lets go, im ready` → `let's go, I'm ready`
20. **Numeri allineati con lo zero**, quasi sempre a **due** cifre, e `(3)` → `03` (4:
    `Img:Social`, `Patreon`, `Reddit`, `Roccobot`). `Beach day 3` → `Beach day 03`
21. **Estensioni uniformi in minuscolo**: `jpeg` → `jpg` (5, tutti salvo `NSFWalbum`, che lavora
    su cartelle) e `m4v` → `mp4` (4: `Grabber`, `Patreon`, `Reddit`, `Roccobot`). `IMG.JPEG` →
    `IMG.jpg`
22. **Suffissi di copia tolti**: ` copy`, ` copia`, `copia di ` (2: `Img:Social`, `Roccobot`).

## 🟡 Generi specifici: esempi da citare, non da adottare in blocco

- **Franchise e personaggi** (`Grabber`, una trentina di regole): la variante romanizzata diventa
  il nome internazionale (`(Shingeki no Kyojin)` → `(Attack on Titan)`), via gli editori
  (`Nintendo`, `and 3 more`), personaggi uniti sotto una serie sola:
  `Rem (Re:Zero) + Ram (Re:Zero)` → `Rem + Ram (ReZero)`.
- **Minuscole nel Title Case** (`Grabber`, `Img:Social`): da 226 a 238 parole inglesi (articoli,
  preposizioni, verbi, colori, volgarità) e 250 parole giapponesi romanizzate (`Grabber`).
  `The Girl In The Red Dress` → `The Girl in the red Dress`
- **Nomi propri con la maiuscola** (`Img:Social`, `NSFWalbum`): giorni, mesi, continenti e circa
  380 paesi e nazionalità; `NSFWalbum` aggiunge le feste. `summer in japan` → `summer in Japan`
- **Studi NSFW** (`NSFWalbum`, 48 regole): grafia giusta dello studio più ` - `, poi maiuscole e
  `&` fra le modelle. `metart anna and bea - sunny day` → `Met-Art - Anna & Bea - Sunny day`
- **Risoluzioni, conteggi, date, dimensioni** (`NSFWalbum`, una ventina di regole): `x120`,
  `5616x3744px`, `85 pics`, `2019-05-12`, `350 MB`, i mesi, `issue 12`.
  `Sunny day x85 2019-05-12` → `Sunny day`. `Roccobot` scrive `1080 p` → `1080p`.
- **Parti e capitoli** (`NSFWalbum`, `Roccobot`): `part 2`, `pt.2`, `parte 02` → `II`, da 1 a 5;
  `Roccobot` scrive `volume 3` → `Vol.3`, `capitolo 4` → `Ch.4`.
- **Nome di serie di gallery-dl rigirato** (`Img:Social`, `Patreon`):
  `newgrounds_812345_My Title` → `My Title - Newgrounds 812345`,
  `123456789_Titolo_01` → `Titolo 01 - Patreon 123456789`.
- **Gerghi e parole singole**: `rn` → `right now`, `OF` → `OnlyFans`, `B&W`, `Q&A`, via
  `maxresdefault`, `cdnvideo`, `poll winners` (`Img:Social`, `NSFWalbum`, `Roccobot`).
- **Riserva per i nomi vuoti** (`Reddit`): un nome di sole cifre diventa `90` più quattro cifre a
  caso.

## 🔁 Regole trasversali

Le candidate a regola fissa sono quelle presenti in cinque o sei predefiniti e indipendenti dal
sito: apici e trattini (1-2), separatore ` - ` (3), spazi e bordi (4-5), caratteri riservati (8),
marchi (17), estensioni (21). Con quattro predefiniti seguono U+3000 (6), non Latin-1 (7),
punteggiatura (9-10), parentesi vuote (11), contrazioni (19) e numeri allineati (20); con tre,
prima maiuscola, tag di Reddit, emoticon, parole di servizio, sigle e domini (12-16, 18).

## 🗃️ Dati che gallery-dl dà già

- **NSFWAlbum**: `studio`, `models` e `title` sostituiscono i 48 riconoscimenti di studio e le
  regole che separano modelle e titolo; la grafia degli studi resta da uniformare solo se il campo
  non la dà già (da verificare sui dati veri).
- **Reddit**: `title` è già senza identificativo, quindi `^\S{5,}\s+` non serve; `id` e `num`
  danno identificativo e numero. Il flair dovrebbe essere `link_flair_text` (da verificare): se
  c'è, si toglie dal titolo per confronto.
- **Estensione**: il campo `extension` uniforma `jpeg` e `m4v` senza leggere il nome.
- **Patreon, Newgrounds, ArtStation, DeviantArt, Imgur**: le regole rigirano `id`, titolo e
  numero del nome di serie; Haul può comporre dai campi (nomi esatti per sito da verificare).
- **Booru (`Grabber`)**: personaggi e serie potrebbero arrivare già divisi (`tags_character`,
  `tags_copyright`, con l'opzione `tags`): da verificare.
- **Date e risoluzioni**: dove ci sono `date`, `width` e `height`, servono solo le regole che le
  **tolgono** dal titolo.

## ⚠️ Possibili problemi

**Contrasti con le decisioni già prese**

- **Caratteri vietati cancellati**: i predefiniti tolgono `: ? * / \ < >` e quattro fanno di `|`
  un `-`; per Haul i caratteri vietati da Windows diventano `_`.
- **`[^\x00-\xFF]` cancella gli alfabeti non latini**: un titolo tutto giapponese resta vuoto,
  mentre per Haul diventa `nomeCartella ###`; il controllo sull'alfabeto va fatto prima della
  pulizia. Toglie anche lettere latine fuori da Latin-1 (`ł`, `č`, `ő`, `ş`).
- **Numeri a due cifre** (`01`), contro le tre di Haul (`001`).
- **Numero in testa** in `Patreon` e `Img:Social` (`01 - Titolo`), contro titolo e numero.
- **Sito e identificativo sempre nel nome** (`- Patreon 123456789`): per Haul l'identificativo va
  in fondo solo se il nome è già preso, e senza etichetta.
- **Riserva casuale** di `Reddit` (`90` più cifre a caso), contro `nomeCartella ###`.
- **Tonde tolte per intero** (`Img:Social`): Haul toglie i tag fra quadre; le tonde possono
  contenere parte del titolo.
- **Title Case e minuscole forzate**: Haul ha deciso solo che il tutto maiuscolo diventa normale.

**Regole rischiose**

- **Solo inglese**: contrazioni, `i` → `I` e Title Case guastano i titoli italiani (`i gatti` →
  `I gatti` in `NSFWalbum`).
- **Sigle ambigue**: `ko`, `op`, `sd`, `ita`, `jap` diventano maiuscole anche dove sono parole.
- **`OF` → `OnlyFans`** in un titolo tutto maiuscolo (`PROOF OF LOVE`): il maiuscolo va
  normalizzato prima.
- **`NSFWalbum` toglie parole vere**: `may`, `mar`, `jun` come parola, e
  `\s?\S{3,}?[ -]\d{1,3}[ -]\d{2,4}` cancella una parola seguita da due numeri.
- **Domini in `Grabber`**: per la precedenza dell'alternanza solo `.net` ha l'ancoraggio, quindi
  `.co` (col punto) sparisce dentro qualunque nome.
- **Regole morte o ripetute**: in `Img:Social` `^(.+)\s(\d+)\s(.*)$` non ha il segno di regex e
  non scatta; `Grabber` ripete tre volte la stessa fusione.
- **`m4v` → `mp4`** cambia solo il nome, non il contenitore.
- **Nessun predefinito** separa i nomi incollati né taglia a 120 caratteri: sono regole nuove.
