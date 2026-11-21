# Firma, autenticazione e aggiornamenti di Haul

Type: research
Status: resolved
Blocked by:

## Question

Che cosa serve, a novembre 2026, per distribuire Haul fuori dal Mac App Store (`Canale-01`) e
aggiornarlo? Con fonte primaria e data:
- Apple Developer Program: costo, che cosa serve per firmare con Developer ID e autenticare
  (notarizzazione), e se si può fare da GitHub Actions senza esporre i segreti (i segreti vivono
  solo lato server, nei secret del repo);
- Hardened Runtime ed entitlement necessari a un'app che lancia un eseguibile scaricato (il
  programma gallery-dl) e scrive in cartelle scelte dall'utente;
- l'aggiornamento di Haul stesso: Sparkle 2 (firma EdDSA, appcast su GitHub Pages o Releases) e le
  alternative, con la loro manutenzione;
- costo di Actions su macOS per un repo pubblico e durata tipica di build, firma e autenticazione;
- formato del pacchetto (DMG, ZIP) e come lo installa chi non è esperto.

Esito atteso: la catena consigliata dalla build al file che si scarica, con i passi che può fare
solo Rocco (iscrizione, certificati) separati da quelli automatici.

## Answer

Ricerca completa in [firma-e-aggiornamenti.md](../research/firma-e-aggiornamenti.md) (2026-11-21).

- **Catena consigliata**: un tag di versione avvia un flusso di Actions su `xcode-27` (Xcode 27.2):
  build `arm64`, firma Developer ID con Hardened Runtime e marca temporale, DMG firmato,
  notarizzazione con `notarytool` e una chiave API Team, `stapler`, poi `generate_appcast` di
  Sparkle 2 con la chiave EdDSA; appcast su GitHub Pages e DMG nelle Releases. Firma e
  pubblicazione partono solo dopo l'approvazione di Rocco (environment protetto).
- **Entitlement**: nessuna eccezione e niente sandbox; gallery-dl è un processo separato, quindi la
  validazione delle librerie non lo riguarda (deduzione da provare nel primo prototipo).
- **Solo Rocco**: iscrizione all'Apple Developer Program, certificato Developer ID come `.p12`,
  chiave API Team, chiave EdDSA con copie di sicurezza, segreti del repo, attivazione di Pages.
- **Costi**: 99 USD l'anno ad Apple; Actions gratuito sui runner standard per un repo pubblico;
  Sparkle gratuito.
- **Rischi**: l'immagine `xcode-27` è in anteprima; a chi macOS chieda il consenso per le cartelle
  quando scrive gallery-dl non è chiaro; i tempi di build vanno misurati al primo rilascio.
