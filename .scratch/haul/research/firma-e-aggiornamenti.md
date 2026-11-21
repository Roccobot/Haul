# Firma, autenticazione e aggiornamenti di Haul

> Ricerca per il ticket `issues/04-firma-e-aggiornamenti.md`. Tutte le fonti sono state lette il
> 2026-11-21. Le pagine della documentazione Apple sono state lette nella loro versione dati
> (`developer.apple.com/tutorials/data/documentation/...json`), che contiene lo stesso testo della
> pagina Web. Quello che non è stato possibile verificare è segnato come **non verificato**.
> Alcuni fatti provengono dai forum sviluppatori di Apple, dove scrivono gli ingegneri del supporto
> tecnico Apple (DTS): sono segnati come **forum Apple**, perché non sono documentazione ufficiale.

## 📖 Due parole prima di cominciare

- **Firma del codice**: un sigillo crittografico che Apple chiede su ogni programma. Dice chi l'ha
  fatto e garantisce che nessuno l'ha modificato dopo. **Developer ID** è il tipo di certificato
  Apple per firmare app distribuite fuori dal Mac App Store.
- **Notarizzazione** (in italiano anche autenticazione): si manda l'app firmata ad Apple, un
  sistema automatico la controlla in pochi minuti e rilascia un **ticket**, un attestato che dice
  che l'app è stata controllata. Non è la revisione del Mac App Store.
- **Graffatura** (in inglese stapling): il ticket è attaccato al file distribuito, così il Mac
  dell'utente lo trova anche senza connessione.
- **Gatekeeper**: la parte di macOS che, alla prima apertura di un programma scaricato, controlla
  firma e ticket e decide se mostrare un avviso o bloccarlo.
- **Quarantena**: un'etichetta nascosta (attributo esteso `com.apple.quarantine`) che i browser e
  le app per utenti mettono sui file scaricati. È questa etichetta che fa scattare Gatekeeper.
- **Hardened Runtime**: un insieme di protezioni che si accende al momento della firma e rende
  l'app più difficile da manomettere mentre gira. È obbligatorio per la notarizzazione.
- **Entitlement**: una dichiarazione scritta dentro la firma che chiede al sistema un permesso o
  un'eccezione alle protezioni (per esempio: posso caricare librerie di altri sviluppatori).
- **Library validation** (validazione delle librerie): una delle protezioni dell'Hardened Runtime.
  Impedisce all'app di caricare **dentro di sé** librerie che non sono firmate da Apple o dallo
  stesso team.
- **App Sandbox**: un recinto che limita file e risorse a cui l'app può accedere. È obbligatorio
  solo sul Mac App Store.
- **Appcast**: il file (un feed RSS) che l'app legge per sapere se esiste una versione nuova.
- **Runner**: la macchina virtuale su cui GitHub Actions esegue un flusso di lavoro.
- **Segreto** (secret): un valore cifrato salvato nelle impostazioni del repo su GitHub, che il
  flusso legge mentre gira e che non compare mai nel codice.

## 1. Apple Developer Program: costo e che cosa serve

### 💶 Costo

- Il programma costa **99 USD all'anno**, in valuta locale dove disponibile, e i prezzi possono
  variare per regione ([Program enrollment](https://developer.apple.com/help/account/membership/program-enrollment),
  letta il 2026-11-21; [What's included](https://developer.apple.com/programs/whats-included/),
  letta il 2026-11-21). Il prezzo esatto in euro per l'Italia è **non verificato**: Apple lo
  mostra durante l'iscrizione.
- L'esenzione dalla quota vale solo per organizzazioni non profit, istituzioni scolastiche
  accreditate ed enti governativi ([What's included](https://developer.apple.com/programs/whats-included/),
  nota 1), quindi non per una persona che pubblica un'app gratuita.
- Se ci si iscrive dall'app Apple Developer, l'iscrizione è un abbonamento annuale che si rinnova
  da sé finché non lo si disdice ([Program enrollment](https://developer.apple.com/help/account/membership/program-enrollment)).

### 🪪 Che cosa serve per iscriversi (come persona)

- Un Apple Account con l'**autenticazione a due fattori** attiva, e la maggiore età.
- Il **nome legale vero**: niente pseudonimi né nomi d'arte, altrimenti l'approvazione ritarda.
- Se si paga con carta di credito come persona, la carta deve essere **propria**; altrimenti Apple
  chiede un documento d'identità con foto.
- Se la conferma non arriva entro 24 ore dall'acquisto, si contatta Apple con l'Enrollment ID.
  (Tutto da [Program enrollment](https://developer.apple.com/help/account/membership/program-enrollment).)

### ✍️ Che cosa serve per firmare e notarizzare

- Un certificato **Developer ID Application** (firma l'app). Esiste anche **Developer ID
  Installer**, che serve solo per i pacchetti `.pkg`, e Haul non lo usa se si distribuisce in DMG.
- Il certificato lo può creare **solo l'Account Holder** (il titolare dell'iscrizione, cioè Rocco);
  se ne possono avere fino a cinque per tipo. Si crea da Certificates, Identifiers & Profiles con
  una richiesta di firma (file `.certSigningRequest`) fatta da Accesso Portachiavi
  ([Create Developer ID certificates](https://developer.apple.com/help/account/certificates/create-developer-id-certificates),
  letta il 2026-11-21).
- Quando il certificato scade serve essere ancora iscritti per averne uno nuovo e firmare
  aggiornamenti (stessa pagina). La durata del certificato non è scritta in quella pagina:
  **non verificato** su fonte primaria.
- Requisiti della notarizzazione
  ([Notarizing macOS software before distribution](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution),
  letta il 2026-11-21):
  - ogni eseguibile distribuito firmato, con firma valida;
  - certificato di tipo Developer ID (non Apple Development, non ad-hoc);
  - **Hardened Runtime** acceso per app e strumenti a riga di comando;
  - **marca temporale sicura** nella firma (`--timestamp`);
  - niente entitlement `com.apple.security.get-task-allow` (quello che permette il debug);
  - entitlement in XML ben formato.
- Lo strumento è `notarytool` (con Xcode); `altool` e Xcode 13 o precedenti non sono più accettati
  dal 1° novembre 2023 (stessa pagina).
- Dal 2019 tutto il software distribuito con Developer ID deve essere notarizzato per girare
  senza blocchi su macOS 10.15 e successivi (stessa pagina).
- Da macOS Sequoia l'utente **non può più aggirare Gatekeeper col Ctrl-clic** su un'app non
  firmata o non notarizzata: deve andare in Impostazioni di Sistema > Privacy e sicurezza
  ([Updates to runtime protection in macOS Sequoia](https://developer.apple.com/news/?id=saqachfa),
  6 agosto 2024, letta il 2026-11-21). Per chi non è esperto, quindi, un'app non notarizzata è di
  fatto inutilizzabile.

### 🤖 Si può fare da GitHub Actions senza esporre i segreti?

**Sì.** Servono due credenziali, tutte e due salvate come segreti del repo:

1. **Il certificato con la sua chiave privata**, esportato da Accesso Portachiavi come file
   `.p12` protetto da password, poi convertito in testo base64. GitHub documenta esattamente
   questo schema: segreti `BUILD_CERTIFICATE_BASE64`, `P12_PASSWORD`, `KEYCHAIN_PASSWORD`; il
   flusso crea un **portachiavi temporaneo** sul runner, ci importa il certificato e firma. I
   runner ospitati da GitHub sono distrutti a fine lavoro, insieme al certificato
   ([Sign Xcode applications](https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms/sign-xcode-applications),
   letta il 2026-11-21). Per un'app Developer ID senza servizi speciali il profilo di
   provisioning di quella guida non serve (**non verificato** per Haul finché non si sa se userà
   servizi che lo richiedono).
2. **La credenziale per la notarizzazione.** Due strade, documentate da Apple:
   - una **chiave API di App Store Connect** di tipo **Team** (file `.p8`, Key ID, Issuer ID),
     passata a `notarytool` con `--key`, `--key-id`, `--issuer`. Le chiavi di tipo Individual
     **non possono usare** `notaryTool`
     ([Creating API keys for App Store Connect API](https://developer.apple.com/documentation/appstoreconnectapi/creating-api-keys-for-app-store-connect-api),
     letta il 2026-11-21; opzioni lette da `xcrun notarytool submit --help` su questo Mac);
   - in alternativa Apple ID più **password specifica per app** e Team ID
     ([Customizing the notarization workflow](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow),
     letta il 2026-11-21). La chiave API è preferibile: non dipende dalla password dell'account
     e si revoca da sola.

Protezioni che GitHub offre e che contano per un **repo pubblico**:

- i segreti **non sono passati** ai flussi avviati da un fork (per esempio una pull request di
  un estraneo), salvo `GITHUB_TOKEN`
  ([Using secrets](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets),
  letta il 2026-11-21);
- i segreti stampati nei log sono oscurati, ma GitHub avverte che l'oscuramento **non è
  garantito** se il valore è trasformato
  ([Secrets](https://docs.github.com/en/actions/concepts/security/secrets), letta il 2026-11-21);
- i segreti di un **environment** possono richiedere l'approvazione di un revisore prima che un
  lavoro li legga (stessa pagina): utile per mettere firma e pubblicazione dietro un clic di Rocco;
- un segreto può pesare al massimo 48 KB (pagina Using secrets): un `.p12` in base64 rientra
  normalmente (dimensione tipica **non verificata**).

## 2. Hardened Runtime ed entitlement per Haul

### 🛡️ Che cosa fa l'Hardened Runtime e che cosa non tocca

- Protegge l'integrità del programma mentre gira (iniezione di codice, librerie sostituite,
  manomissione della memoria). Non influisce sulla maggior parte delle app; blocca alcune capacità
  rare come la compilazione al volo (JIT). Per riaprirne una si aggiunge un entitlement, solo se
  davvero necessario. Gli entitlement si mettono **solo sugli eseguibili**: librerie e plug-in
  caricati dentro l'app ereditano quelli dell'eseguibile che li ospita
  ([Hardened Runtime](https://developer.apple.com/documentation/security/hardened-runtime),
  letta il 2026-11-21).
- La validazione delle librerie impedisce di **caricare dentro il programma** framework, plug-in o
  librerie non firmati da Apple o dallo stesso team. Se la si disattiva
  (`com.apple.security.cs.disable-library-validation`), Gatekeeper fa **controlli in più** sul
  programma
  ([Disable Library Validation](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.security.cs.disable-library-validation),
  letta il 2026-11-21).
- **Punto chiave per Haul**: gallery-dl non è caricato dentro Haul come libreria, è
  **lanciato come processo separato** (un programma a sé). La validazione delle librerie riguarda
  ciò che un processo carica dentro di sé, quindi **non si applica** al lancio di gallery-dl.
  L'Hardened Runtime è un'opzione che si mette sulla firma di ciascun eseguibile (`-o runtime`,
  [Creating distribution-signed code for the Mac](https://developer.apple.com/documentation/xcode/creating-distribution-signed-code-for-the-mac),
  letta il 2026-11-21); gallery-dl ha la sua firma (o nessuna) e gira con le regole sue. Che il
  figlio **non erediti** l'Hardened Runtime del padre è una deduzione da queste due pagine, non
  una frase esplicita di Apple: **non verificato** alla lettera, da provare col primo prototipo.

### ✅ Entitlement che servono a Haul

- **Nessun entitlement di eccezione** per lanciare gallery-dl: niente
  `disable-library-validation`, niente `allow-jit`, niente `allow-unsigned-executable-memory`.
  Questi servirebbero solo se Haul caricasse codice altrui **dentro di sé** (deduzione dalle pagine
  sopra).
- **Niente App Sandbox.** È obbligatoria solo per il Mac App Store
  ([App Sandbox](https://developer.apple.com/documentation/security/app-sandbox), letta il
  2026-11-21), e Haul è fuori dallo Store. Con la sandbox, un eseguibile lanciato dall'app deve
  **ereditarne** la sandbox, con gli entitlement `com.apple.security.app-sandbox` e
  `com.apple.security.inherit` nella sua firma
  ([Embedding a command-line tool in a sandboxed app](https://developer.apple.com/documentation/xcode/embedding-a-helper-tool-in-a-sandboxed-app),
  letta il 2026-11-21): con un gallery-dl scaricato e firmato da altri questo **non si può fare**.
  Un ingegnere Apple scrive inoltre che l'entitlement `files.user-selected.executable` non è nato
  per eseguire codice scaricato dalla rete
  ([forum Apple 683551](https://developer.apple.com/forums/thread/683551), giugno 2021). Quindi la
  sandbox è da escludere.
- **Nessun entitlement per scrivere nelle cartelle scelte dall'utente**: fuori dalla sandbox non
  serve. Per Scrivania, Documenti e Download macOS chiede il consenso la prima volta, **salvo** che
  la cartella o il file sia stato scelto dall'utente in un pannello Apri o Salva, trascinato
  sull'app, o creato dall'app stessa. Si consiglia di aggiungere le frasi di spiegazione
  `NSDownloadsFolderUsageDescription`, `NSDesktopFolderUsageDescription` (e l'analoga per
  Documenti) nell'`Info.plist`
  ([NSDownloadsFolderUsageDescription](https://developer.apple.com/documentation/bundleresources/information-property-list/nsdownloadsfolderusagedescription),
  [NSDesktopFolderUsageDescription](https://developer.apple.com/documentation/bundleresources/information-property-list/nsdesktopfolderusagedescription),
  lette il 2026-11-21). A chi macOS attribuisce la richiesta quando scrive il processo figlio
  gallery-dl (a Haul o a gallery-dl) è **non verificato** su fonte primaria: da provare.
- Gli entitlement di serie, quindi, sono **vuoti** o quasi: firma con `--timestamp -o runtime` e
  nessuna eccezione.

### 📥 Il binario di gallery-dl scaricato: quarantena, Gatekeeper, firma

Che cosa si sa oggi del file da scaricare:

- Il rilascio di oggi di gallery-dl (`v1.32.21`, 2026-11-21) non ha file allegati nel repo
  principale; i binari vivono nel repo `gdl-org/builds`, che nell'ultimo rilascio (`2026.11.21`)
  ha `gallery-dl_macos` (letto con l'API di GitHub il 2026-11-21).
- Il flusso che lo produce (`.github/workflows/executables.yml` di `mikf/gallery-dl`) lo compila
  con **PyInstaller** in un solo file (`--onefile`) sul runner `macOS-latest`, e **non contiene
  alcun passo di firma né di notarizzazione** (letto il 2026-11-21). Quindi il binario **non ha una
  firma Developer ID e non è notarizzato**. Il runner `macos-latest` è oggi arm64
  ([GitHub-hosted runners](https://docs.github.com/en/actions/reference/runners/github-hosted-runners),
  letta il 2026-11-21), e lo script non chiede un'altra architettura: il binario è
  presumibilmente arm64 (**non verificato** sul file).
- Sui Mac Apple silicon ogni eseguibile arm64 **deve avere almeno una firma ad-hoc** (una firma
  senza identità) per poter girare; una firma ad-hoc però **non supera Gatekeeper**
  ([macOS Big Sur 11.0.1 Universal Apps release notes](https://developer.apple.com/documentation/macos-release-notes/macos-big-sur-11_0_1-universal-apps-release-notes),
  letta il 2026-11-21). Che PyInstaller metta da sé questa firma ad-hoc è **non verificato** su
  fonte primaria (non si è scaricato il file per controllarlo).
- Se il binario fosse solo Intel, girerebbe con Rosetta, che Apple chiude: macOS 27 è l'ultima
  versione che lo supporta per le app Intel
  ([Upcoming changes to Rosetta support](https://developer.apple.com/news/?id=w5ngl9k2),
  1° settembre 2026, letta il 2026-11-21).

Come si comporta il sistema:

- La quarantena la mettono le **app per utenti** che scaricano (Safari, per esempio); gli
  strumenti Unix come `curl` non la mettono; l'etichetta va letta o tolta con l'API
  `quarantinePropertiesKey`, non scrivendo l'attributo a mano. **Lanciare un programma in
  quarantena fa sempre scattare Gatekeeper**; il sistema può farlo scattare anche in altri casi
  non documentati, che cambiano nel tempo ([forum Apple 706442](https://developer.apple.com/forums/thread/706442),
  rivisto il 2025-08-06).
- L'opzione `LSFileQuarantineEnabled` dell'`Info.plist` dice se i file creati dall'app sono messi
  in quarantena **di serie**
  ([LSFileQuarantineEnabled](https://developer.apple.com/documentation/bundleresources/information-property-list/lsfilequarantineenabled),
  letta il 2026-11-21). Haul **non** deve attivarla.
- Un'app che scarica con `URLSession` fuori dalla sandbox e senza quella opzione **non** mette in
  quarantena i file: è la deduzione dalle due fonti sopra, **non verificata** alla lettera, da
  provare controllando l'attributo sul file scaricato.
- XProtect (l'antivirus di macOS) controlla le app alla **prima esecuzione** e quando cambiano sul
  disco ([Protecting against malware in macOS](https://support.apple.com/guide/security/protecting-against-malware-sec469d47bd8/web),
  letta il 2026-11-21). Vale anche per gallery-dl, a ogni aggiornamento.

Che cosa deve fare Haul perché gallery-dl giri senza avvisi:

1. Scaricarlo **lui stesso**, con `URLSession`, in `~/Library/Application Support/Haul/` (non nel
   pacchetto dell'app, che è sigillato dalla firma), **senza** quarantena. Se il file arrivasse
   comunque in quarantena, toglierla con `quarantinePropertiesKey`, solo dopo aver verificato
   l'integrità del file (vedi il punto 3).
2. Renderlo eseguibile (permessi) e **sostituirlo in modo atomico** a ogni aggiornamento: scrivere
   in un file temporaneo e poi rinominarlo sopra il vecchio. Apple avverte che sovrascrivere un
   eseguibile firmato sul posto provoca crash da firma non valida difficili da riprodurre
   ([Updating Mac software](https://developer.apple.com/documentation/security/updating-mac-software),
   letta il 2026-11-21).
3. Verificarne l'**integrità** prima di eseguirlo (per esempio un'impronta SHA-256 confrontata con
   una fonte affidabile): togliere la quarantena significa che Gatekeeper non lo controlla, quindi
   il controllo lo fa Haul. Quale impronta o firma pubblichi gallery-dl per `gdl-org/builds` è
   **non verificato**, e appartiene al ticket sulla fonte dell'eseguibile.
4. Se il binario non avesse nemmeno la firma ad-hoc, Haul può dargliela sul Mac dell'utente
   (`codesign -s -`): è il minimo che Apple silicon richiede (release notes di Big Sur sopra).
   Da decidere solo dopo aver controllato il file.

Alternativa più robusta, da valutare nel ticket sulla fonte: una copia di gallery-dl **firmata con
il Developer ID di Haul e notarizzata** dal flusso di Haul (Apple dice che anche i binari a sé si
notarizzano, ma il ticket non si può graffare: Gatekeeper lo cerca in rete
[Customizing the notarization workflow](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow)).
Costa un flusso in più a ogni versione di gallery-dl e rende Rocco il distributore di quel binario.

## 3. L'aggiornamento di Haul stesso

### 🟢 Sparkle 2

- **Stato**: ultima versione stabile `2.10.1` (2026-10-25), beta `2.11.0-beta.1` (2026-11-17);
  repo aggiornato oggi (API di GitHub, letta il 2026-11-21). Manutenzione **attiva**, con rilasci
  circa mensili nel 2026.
- **Licenza**: testo di tipo MIT, con componenti BSD e zlib-like (bsdiff, ed25519)
  ([LICENSE](https://github.com/sparkle-project/Sparkle/blob/2.x/LICENSE), letta il 2026-11-21).
  Compatibile con GPL-3.0 secondo la regola generale che MIT e BSD si possono includere in GPL:
  è un fatto noto di licenze, non letto in una fonte di questa ricerca (**non verificato** qui).
- **Come funziona** ([Documentation](https://sparkle-project.org/documentation/), letta il
  2026-11-21):
  - si aggiunge con Swift Package Manager;
  - `generate_keys` crea una coppia di chiavi **EdDSA** (ed25519): la privata va nel Portachiavi,
    la pubblica nell'`Info.plist` come `SUPublicEDKey`;
  - `SUFeedURL` nell'`Info.plist` indica dove si trova l'appcast, servito in **HTTPS**;
  - `CFBundleVersion` (il numero di build) deve salire a ogni rilascio;
  - `generate_appcast` legge una cartella di archivi, firma ognuno e scrive l'appcast, con gli
    aggiornamenti delta (solo le differenze);
  - formati accettati: DMG, ZIP, tar, Apple Archive, pkg;
  - la validazione delle librerie è compatibile: con la firma Developer ID non c'è niente da
    disattivare;
  - si possono firmare anche il feed e le note di rilascio (`SURequireSignedFeed`, dalla 2.9).
- **Fuori dalla sandbox** i servizi XPC di Sparkle non servono e si possono togliere
  ([Sandboxing](https://sparkle-project.org/documentation/sandboxing/), letta il 2026-11-21).
- **In CI senza esporre la chiave**: `generate_appcast` accetta la chiave privata da standard
  input, con `--ed-key-file -`, e l'aiuto del programma porta proprio l'esempio di un segreto
  passato così; `--download-url-prefix` imposta l'indirizzo da cui scaricare gli archivi
  (sorgente `generate_appcast/main.swift`, ramo `2.x`, letto il 2026-11-21). La chiave privata
  EdDSA diventa quindi un **terzo segreto** del repo.
- **Dove mettere i file**:
  - gli **archivi** come allegati delle **GitHub Releases** (indirizzi stabili, gratuiti);
  - l'**appcast** su **GitHub Pages** (HTTPS) o come file nel repo servito da Pages. Che GitHub
    Releases vada bene per gli archivi segue da `--download-url-prefix` e dall'obbligo di HTTPS;
    la pagina di Sparkle sulla pubblicazione non nomina GitHub (**nessuna raccomandazione
    ufficiale** letta).
- **Due firme diverse**: la firma Developer ID protegge l'app sul Mac; la firma EdDSA protegge
  l'aggiornamento (Sparkle installa solo archivi firmati con la chiave giusta). Si possono cambiare
  una alla volta, mai tutte e due insieme ([Documentation](https://sparkle-project.org/documentation/)).

### 🟡 Alternative

- **Avviso fatto in casa**: Haul interroga l'API delle Releases di GitHub, confronta il numero di
  build e apre la pagina di download. Poco codice, nessuna dipendenza, ma l'utente scarica e
  sostituisce a mano. Nessuna fonte da citare: è una scelta di progetto.
- **Aggiornatore fatto in casa con installazione**: da sconsigliare. Va scritto e mantenuto
  tutto ciò che Sparkle già fa (firma, sostituzione atomica, permessi, riavvio), con i rischi che
  Apple descrive in [Updating Mac software](https://developer.apple.com/documentation/security/updating-mac-software).
- **Homebrew Cask** o altri canali di terzi: possibili in aggiunta, non al posto di Sparkle;
  manutenzione e regole **non verificate** in questa ricerca.

## 4. GitHub Actions su macOS: costo e durata

- **Costo per un repo pubblico: zero.** I runner standard ospitati da GitHub sono gratuiti nei
  repo pubblici; i runner **larger** (per esempio `-xlarge`) si pagano anche nei repo pubblici.
  Per i repo privati la tariffa macOS standard è 0,062 USD al minuto
  ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions),
  letta il 2026-11-21).
- **Limiti**: ogni lavoro dura al massimo 6 ore; al massimo 5 lavori macOS in parallelo col piano
  Free ([Actions limits](https://docs.github.com/en/actions/reference/limits), letta il
  2026-11-21). Nei momenti di carico i lavori macOS possono partire in ritardo
  ([GitHub-hosted runners](https://docs.github.com/en/actions/reference/runners/github-hosted-runners)).
- **Quale runner**: il minimo macOS 27.2 richiede l'SDK di macOS 27.2, che c'è solo in **Xcode
  27.2**. Oggi Xcode 27 è sul runner `xcode-27` (macOS 27.0.1, M1 a 3 core, 7 GB), con Xcode
  27.2 installato ma 27.1 come predefinito: va scelto esplicitamente
  ([scheda dell'immagine xcode-27-arm64](https://github.com/actions/runner-images/blob/main/images/macos/xcode-27-arm64-Readme.md),
  immagine `20261104.0337.1`, letta il 2026-11-21). L'immagine è ancora in **anteprima**
  (preview): l'annuncio è aperto e avverte di possibili instabilità e code
  ([runner-images #14404](https://github.com/actions/runner-images/issues/14404), aggiornato il
  2026-11-21). Le immagini `macos-26` e `macos-latest` hanno solo Xcode 26.
- **Durata tipica**:
  - notarizzazione: Apple scrive di solito **meno di un'ora**, e nella pagina dell'API **pochi
    minuti**
    ([Customizing the notarization workflow](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow);
    [Submitting software for notarization over the web](https://developer.apple.com/documentation/notaryapi/submitting-software-for-notarization-over-the-web),
    lette il 2026-11-21);
  - build, firma, DMG e appcast: **non verificato**, nessuna fonte primaria dà tempi. Stima
    personale per un'app piccola con nucleo Rust su un M1 a 3 core: 10-25 minuti a rilascio,
    notarizzazione compresa. Da misurare al primo rilascio.

## 5. Formato del pacchetto e installazione

- Per la distribuzione diretta i contenitori comuni sono ZIP, DMG e pacchetto `.pkg`
  ([Packaging Mac software for distribution](https://developer.apple.com/documentation/xcode/packaging-mac-software-for-distribution),
  letta il 2026-11-21).
- **DMG** (immagine disco): si firma con il Developer ID Application, si notarizza e **si graffa**
  direttamente (`xcrun stapler staple Haul.dmg`); il formato richiesto è UDIF compresso in sola
  lettura (`UDZO`); si possono disporre le icone, mettere uno sfondo e un collegamento alla
  cartella Applicazioni (stessa pagina). Con contenitori annidati si notarizza solo quello più
  esterno (stessa pagina).
- **ZIP**: non si firma e **non si graffa**; si graffa l'app dentro e poi si rifà lo ZIP (stessa
  pagina e [Customizing the notarization workflow](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow)).
- **Senza graffatura** Gatekeeper potrebbe bloccare l'installazione se il Mac è offline
  ([Packaging Mac software for distribution](https://developer.apple.com/documentation/xcode/packaging-mac-software-for-distribution)).
- **Traslocazione**: se l'utente apre l'app direttamente dal DMG o dalla cartella dello ZIP senza
  spostarla, alla prima apertura Gatekeeper la esegue da un percorso casuale; spostandola prima,
  questo non succede (stessa pagina). Per Haul conta poco, perché gallery-dl vive in
  `Application Support` e non accanto all'app, ma è un motivo in più per guidare l'utente a
  trascinarla in Applicazioni.
- **Come installa chi non è esperto** con un DMG: doppio clic sul file scaricato, si apre una
  finestra con l'icona di Haul e la cartella Applicazioni, si trascina l'una sull'altra, si apre
  Haul da Applicazioni; alla prima apertura macOS mostra un avviso informativo (l'app è stata
  scaricata da Internet) con un pulsante Apri, perché l'app è notarizzata. Il testo esatto
  dell'avviso su macOS 27 è **non verificato**.
- Per Sparkle vanno bene sia DMG sia ZIP; Sparkle consiglia, per i DMG, il formato APFS con
  compressione lzfse
  ([Publishing](https://sparkle-project.org/documentation/publishing/), letta il 2026-11-21).
  Che `UDZO` (richiesto dalla pagina Apple per i DMG di terze parti) e APFS lzfse si possano
  conciliare in un unico file è **non verificato**: la strada sicura è un DMG per il download
  dalla pagina e lo stesso DMG (o uno ZIP) per Sparkle, da provare.

## 6. Raccomandazione

### 🔗 La catena consigliata, dalla build al file che si scarica

Tutto gira in un flusso GitHub Actions avviato da un tag di versione (per esempio `v1.2.0`),
sul runner `xcode-27` con Xcode 27.2 selezionato.

1. **Build** del nucleo Rust (`aarch64-apple-darwin`) e dell'app Swift con `xcodebuild archive`,
   per la sola architettura arm64 (macOS 27 gira solo su Apple silicon).
2. **Firma** con `xcodebuild -exportArchive` (metodo Developer ID) usando il certificato importato
   in un portachiavi temporaneo dai segreti; Hardened Runtime acceso, marca temporale, **nessun
   entitlement di eccezione**, niente sandbox; Sparkle dentro, senza i servizi XPC.
3. **DMG** con `hdiutil` (o uno strumento che produca un DMG firmato), con il collegamento ad
   Applicazioni; firma del DMG con il Developer ID Application.
4. **Notarizzazione** del DMG con `xcrun notarytool submit --wait` e la chiave API Team; lettura
   del log anche se va bene.
5. **Graffatura** con `xcrun stapler staple Haul.dmg`, poi verifica (`stapler validate`,
   `spctl`, `codesign --verify --deep --strict`).
6. **Appcast**: `generate_appcast` con la chiave EdDSA passata da standard input e
   `--download-url-prefix` puntato alla Release; pubblicazione dell'appcast su GitHub Pages.
7. **Release** su GitHub con il DMG allegato. Firma e pubblicazione dietro un **environment** con
   approvazione di Rocco, così nessun rilascio parte senza il suo clic.

A runtime, per gallery-dl: download in `Application Support` senza quarantena, verifica
dell'impronta, sostituzione atomica, lancio come processo figlio. Nessun entitlement extra.

### 👤 Passi che può fare solo Rocco

1. **Iscrizione** all'Apple Developer Program come persona (nome legale, Apple Account con due
   fattori, carta propria), 99 USD all'anno.
2. **Certificato Developer ID Application**: richiesta di firma da Accesso Portachiavi, creazione
   sul sito Apple (solo l'Account Holder può farlo), installazione, esportazione come `.p12` con
   una password robusta. Copia di sicurezza del `.p12` fuori dal repo.
3. **Chiave API di App Store Connect** di tipo **Team** per la notarizzazione: download del file
   `.p8` (si scarica una volta sola, secondo la prassi di App Store Connect: **non verificato** in
   questa ricerca), annotazione di Key ID e Issuer ID.
4. **Chiave EdDSA di Sparkle**: `generate_keys` sul suo Mac, esportazione della privata
   (`generate_keys -x`), copia di sicurezza; la pubblica va nell'`Info.plist` (la può mettere
   l'agente).
5. **Segreti del repo** (o di un environment `release` con lui come revisore): certificato in
   base64, password del `.p12`, password del portachiavi temporaneo, chiave `.p8` (in base64),
   Key ID, Issuer ID, chiave privata EdDSA. Gli agenti scrivono i nomi; i valori li incolla lui.
6. **Attivazione di GitHub Pages** sul repo, se l'appcast vive lì.
7. **Rinnovo** annuale dell'iscrizione e, alla scadenza, del certificato.

### ⚙️ Passi automatici (li scrive e li mantiene l'agente)

- il flusso Actions completo (build, firma, DMG, notarizzazione, graffatura, appcast, Release);
- l'integrazione di Sparkle nell'app e la rimozione dei servizi XPC;
- il codice che scarica, verifica, sostituisce e lancia gallery-dl;
- l'`Info.plist` con `SUFeedURL`, `SUPublicEDKey` e le frasi di consenso per le cartelle;
- i controlli di verifica dopo ogni rilascio (firma, ticket, Gatekeeper).

### 💶 Costi

- Apple Developer Program: **99 USD all'anno** (prezzo in euro **non verificato**).
- GitHub Actions su runner standard, repo pubblico: **0**. Da evitare i runner `-xlarge`, che si
  pagano anche nei repo pubblici.
- GitHub Releases e Pages: **0** (che Pages sia gratuito per un repo pubblico è noto ma **non
  letto** in questa ricerca).
- Sparkle: **0**, licenza di tipo MIT.

### ⚠️ Rischi aperti

- **Immagine `xcode-27` in anteprima**: possibili instabilità e code; Xcode 27.2 non è il
  predefinito. Va scelto esplicitamente e riverificato a ogni rilascio.
- **gallery-dl non firmato da Developer ID**: dipende dal fatto che Haul lo scarichi senza
  quarantena. Se un giorno macOS facesse scattare Gatekeeper anche senza quarantena (Apple dice che
  i casi non sono documentati e cambiano), gallery-dl verrebbe bloccato. Rimedio di riserva: la
  copia firmata e notarizzata dal flusso di Haul.
- **Architettura e firma ad-hoc del binario `gallery-dl_macos`**: non verificate sul file. Se
  fosse solo Intel non girerebbe dopo macOS 27. Da controllare nel ticket sulla fonte.
- **Integrità di gallery-dl**: togliendo la quarantena, il controllo passa a Haul; serve una fonte
  affidabile per l'impronta, oggi **non verificata**.
- **Consenso alle cartelle**: a chi macOS attribuisce le scritture del processo figlio è da
  provare con il primo prototipo.
- **Segreti in un repo pubblico**: protetti dalle regole di GitHub sui fork e dall'environment con
  approvazione; il rischio resta un flusso modificato che stampi un segreto trasformato, che
  l'oscuramento non copre. Rimedio: flusso di rilascio solo sui tag, environment protetto.
- **Perdita della chiave EdDSA o del certificato**: senza la chiave EdDSA gli utenti non possono
  più aggiornarsi da Sparkle; senza copia del `.p12` serve un certificato nuovo. Le copie di
  sicurezza sono un passo di Rocco.
- **Tempi di build**: nessuna fonte; da misurare al primo rilascio.
