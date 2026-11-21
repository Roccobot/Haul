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

**Impostazioni avanzate**:
La parte della finestra delle impostazioni generata dalle opzioni di gallery-dl, accanto alle voci
curate a mano.
_Avoid_: opzioni esperte, configurazione completa
