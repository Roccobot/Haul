# Da dove prende Haul l'eseguibile di gallery-dl

Type: research
Status: resolved
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

Ricerca completa in [fonte-di-gallery-dl.md](../research/fonte-di-gallery-dl.md) (2026-11-21).

- **Strada consigliata (`Fonte-03`)**: Haul include un interprete Python di `python-build-standalone`
  per `arm64`, firmato e autenticato con l'app; gallery-dl, `requests` e `yt-dlp` si scaricano da
  PyPI come wheel pure (solo codice Python) in `Application Support`, e Haul lancia `python -m
  gallery_dl` come processo separato. Dalla rete non arriva nessun eseguibile: niente quarantena,
  niente autenticazione a ogni versione. Integrità con lo SHA-256 di PyPI e la firma GPG del
  manutentore; la versione nuova si legge da `https://pypi.org/pypi/gallery-dl/json`, e la
  precedente resta finché la nuova non supera una prova.
- **Premesse cambiate**: lo sviluppo di gallery-dl è passato su Codeberg (2026-04, dopo una
  segnalazione DMCA); le stabili hanno di nuovo eseguibili, ma solo per Windows e Linux; macOS 27
  gira solo su Mac con chip Apple, quindi basta `arm64`.
- **Scartate**: la build giornaliera di `gdl-org/builds` (sviluppo, firma ad hoc, nessuna firma del
  manutentore); una build PyInstaller fatta da Haul a ogni versione, tenuta di riserva.
- **Da provare in un prototipo**: i permessi dell'interprete sotto il runtime protetto, i tempi di
  avvio. **Da confermare**: l'impronta della chiave GPG del manutentore; le FAQ della FSF sulla
  licenza. `ffmpeg` resta a carico dell'utente, con una guida.
- La scelta della fonte la conferma Rocco nel ticket
  [I confini fra nucleo e interfaccia](12-confini-del-nucleo.md).
