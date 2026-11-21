# Firma, autenticazione e aggiornamenti di Haul

Type: research
Status: open
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
