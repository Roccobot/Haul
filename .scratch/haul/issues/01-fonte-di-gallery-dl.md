# Da dove prende Haul l'eseguibile di gallery-dl

Type: research
Status: open
Blocked by:

## Question

Da quale fonte Haul scarica e aggiorna gallery-dl, e con quali conseguenze? Le strade note
(2026-11-21): le versioni stabili di `mikf/gallery-dl` escono senza eseguibili; un
`gallery-dl_macos` pronto c'è solo nelle build giornaliere di `gdl-org/builds`; l'alternativa è la
versione stabile da PyPI con un Python incluso (per esempio `python-build-standalone`), oppure un
eseguibile costruito apposta da Haul (PyInstaller o simili) sulle stabili.

Per ogni strada, con la fonte primaria e la data:
- stabilità e frequenza degli aggiornamenti, e come si sa che c'è una versione nuova;
- architetture (Apple Silicon, Intel, universale) e versione minima di macOS;
- firma, autenticazione di Apple e quarantena di Gatekeeper per un eseguibile scaricato dopo
  l'installazione di un'app distribuita con Developer ID: che cosa serve perché parta senza avvisi;
- verifica dell'integrità del file scaricato (checksum, firme);
- dimensione e tempo di avvio;
- dipendenze opzionali che gallery-dl usa per alcuni siti (`ffmpeg`, `yt-dlp`) e come si procurano;
- licenza: che cosa chiede la GPL-2.0 di gallery-dl a un'app GPL-3.0 che lo scarica o lo include.

Esito atteso: una strada consigliata, con le alternative scartate e il perché.

## Answer
