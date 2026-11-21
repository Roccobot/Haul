# Distribuzione diretta, fuori dal Mac App Store

Haul si distribuisce firmato con Developer ID e autenticato da Apple, scaricabile da GitHub, e non
dal Mac App Store (scelta `Canale-01`, 2026-11-21). Il motivo è l'aggiornamento di gallery-dl:
la sandbox del Mac App Store vieta a un'app di scaricare ed eseguire programmi dopo
l'installazione, quindi lì gallery-dl dovrebbe essere incluso nell'app e aggiornato solo con una
versione nuova di Haul, mentre i siti cambiano e gallery-dl esce con correzioni anche più volte al
mese.

## Considered Options

- **Mac App Store**: vetrina e aggiornamenti di Apple, ma gallery-dl fermo alla versione inclusa;
  scartato.
- **Solo uso personale**: niente firma né autenticazione, ma l'app non si può dare ad altri senza
  gli avvisi di Gatekeeper; scartato.
