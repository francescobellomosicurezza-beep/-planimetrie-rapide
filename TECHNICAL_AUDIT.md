# Technical audit - Planimetrie Rapide

Data audit: 2026-06-23

Workspace: `/Users/francescobellomo/Desktop/planimetrie_rapide_v4`

Branch Git: non disponibile. La cartella analizzata non risulta essere una repository Git.

Stato Git: `git status` restituisce `fatal: not a git repository (or any of the parent directories): .git`.

## 1. Sintesi esecutiva

Planimetrie Rapide V4.1 e una web app statica reale, standalone, composta quasi interamente da un unico `index.html` con HTML, CSS e JavaScript inline, piu `sw.js`, `manifest.webmanifest`, `icon.svg` e documentazione. Non ci sono framework, build, backend, login, analytics o telemetria.

Il prodotto e gia utilizzabile per creare elaborati schematici: muri, muri REI/EI, porte, finestre, scale, ascensori, aree/locali, simboli antincendio e di emergenza, punti di misura, elementi DVR, vie di esodo, testi, foto base, calibrazione, multipiano, layer, esportazione PNG/PDF e salvataggio JSON. L'autosalvataggio usa IndexedDB con fallback localStorage senza foto.

Il principale limite non e la quantita di funzioni, ma la qualita del controllo operativo: molte funzioni sono presenti e utili, pero richiedono precisione manuale, non hanno snap semantico su muri/porte/finestre, non hanno template guidati per planimetrie di emergenza e dipendono da un export rasterizzato. Per un consulente non CAD e in sopralluogo tablet, il prodotto e promettente ma ancora troppo manuale nei passaggi critici.

Criticita massima: il PDF usa ancora branding `Sicuro 360 - Planimetrie` in `drawPdfHeader` e `drawPdfFooter`, mentre l'obiettivo di prodotto richiede standalone e nessuna integrazione attuale con Sicuro360. Questo non e una integrazione tecnica, ma crea ambiguita di posizionamento, responsabilita e futuro disaccoppiamento.

Verifiche eseguite senza modificare file applicativi: `pwd`, `ls -la`, `git status`, `git branch --show-current`, `rg --files`, lettura codice/documentazione, validazione JSON di `manifest.webmanifest` e `demo_officina_v4_1.json`, parsing sintattico dello script inline con Node. Non sono stati installati pacchetti, non sono stati fatti commit, push o pubblicazioni.

## 2. Struttura reale del progetto

Comandi iniziali:

- `pwd`: `/Users/francescobellomo/Desktop/planimetrie_rapide_v4`
- `ls -la`: cartella con 15 file principali, nessuna directory applicativa.
- `git status`: fallito, non e una repository Git.
- `git branch --show-current`: fallito per lo stesso motivo.
- `rg --files`: conferma struttura piatta.

File rilevati:

- `index.html`: applicazione completa, 2151 righe. Contiene HTML, CSS inline, JavaScript inline, catalogo simboli, modello dati, rendering canvas, interazioni, salvataggio, export e PWA registration.
- `sw.js`: service worker, 22 righe.
- `manifest.webmanifest`: manifest PWA, 18 righe.
- `icon.svg`: icona locale.
- `demo_officina_v4_1.json`: progetto demo multipiano.
- `README.md`, `MANUALE_RAPIDO.md`, `COLLAUDO_MANUALE.md`, `TEST_REPORT.md`, `PRODUCT_AUDIT.md`, `CHECKPOINT_V4_1.md`, `CHANGELOG_V4.md`, `CHANGELOG_V4_1.md`, `INSTALLAZIONE.md`: documentazione e audit precedenti.
- `_headers`: header statici.

Dipendenza esterna reale:

- `https://unpkg.com/jspdf@4.2.1/dist/jspdf.umd.min.js`, caricata da CDN e cacheata dal service worker dopo il primo caricamento riuscito.

## 3. Inventario delle funzionalita

Legenda: completa = implementata e coerente nel codice; parziale = utile ma con limiti rilevanti; solo interfaccia = UI presente ma modello/comportamento insufficiente; non funzionante = bug probabile nel codice; assente = non trovata; non verificabile = richiede collaudo browser reale.

| Funzione | Stato | Evidenza e note |
| --- | --- | --- |
| Muri | Completa | `mode="wall"`, snap ortogonale, rendering, selezione, spostamento, cancellazione, quote opzionali, export. |
| Muri REI/EI | Parziale | `reiWalls`, classe selezionabile e render tratteggiato. Non c'e validazione tecnica della classe ne modifica guidata della classe esistente se non da proprieta generiche/selezione. |
| Porte | Parziale | Segmento porta con verso e proprieta. Non fa snap automatico al muro, non crea apertura vincolata a parete reale. |
| Porte tagliafuoco | Parziale | Tipo `rei`, classe EI/REI e simbolo `porta_tagliafuoco`. Non verifica coerenza con compartimentazione o norme. |
| Finestre | Parziale | Segmento finestra renderizzato. Non vincolato a muro e senza proprieta dimensionali avanzate. |
| Scale | Completa/parziale | `stairs` dedicato, dimensioni, direzione, rotazione, target floor. Collegamento logico tra piani presente, navigazione automatica da scala non presente. |
| Ascensori | Completa/parziale | `elevators` dedicato, tipo, piani serviti, note. Non ci sono controlli su uso in esodo. |
| Aree/locali | Parziale | Poligoni e label. Proprieta avanzate `use`, `code`, `risk`, `note` esistono nel pannello proprieta, ma il foglio rapido area salva solo label. |
| Destinazioni d'uso | Parziale | Campo `use` nel modello/proprieta, datalist di label. Non ci sono preset completi, legenda dedicata o controlli. |
| Presidi | Completa/parziale | Catalogo ampio con antincendio, esodo, primo soccorso, impianti. Pittogrammi proprietari stile ISO, non certificati. |
| Vie di esodo | Parziale | Polilinee con frecce, route type nelle proprieta. Non ci sono maniglie vertice, percorsi alternativi graficamente distinti in export, o verifica continuita uscita/raccolta. |
| Testi | Parziale | Testi liberi con dimensione, rotazione, box, colore. Nessun wrapping automatico su canvas/export. |
| Quote | Parziale | Quote permanenti toggle su muri/REI. Non sono entita modificabili, non esportate salvo toggle globale `showQuotes`. |
| Foto base | Parziale | Import immagine, compressione max 2400px, opacita. Mancano rotazione, crop, riposizionamento tramite maniglie. |
| Calibrazione | Parziale | Due punti e scala reale; riscalatura entita intorno alla foto. Non calibra per piano con storico robusto di trasformazione, non gestisce rotazione foto. |
| Multi-piano | Completa/parziale | Modello `floors`, aggiunta, duplicazione, rinomina, eliminazione, export multipiano. Header ha solo un campo piano globale aggiornato al piano attivo. |
| Layer | Parziale | Visibilita per macro-layer e quote. Non persistita nel JSON e non distinta da layer di export. |
| Orientamento nord | Parziale | Step 45 gradi, render canvas/export. Non e posizionabile liberamente. |
| Intestazione | Parziale | Azienda, indirizzo, piano, revisione, redattore, qualifica. Mancano data/revisione strutturata separata dalla data generata. |
| Istruzioni | Parziale | Testo personalizzabile e pagina PDF. Il default usa 115, non il numero unico 112 richiesto nel caso d'uso. |
| Controlli pre-export | Parziale | Verifica presenza minima di uscita, sei qui, vie, presidi, punto raccolta, misure, aree, calibrazione, azienda, redattore. Non verifica completezza normativa o grafica. |
| PDF A3/A4 | Parziale | jsPDF con A3/A4 automatico portrait/landscape. Export rasterizza il piano e aggiunge header/legenda/footer. |
| Export emergenza | Parziale | Filtra marker `emergency`, routes e REI; include istruzioni. Aree restano sempre visibili per `areasVisible()`. |
| Export DVR | Parziale | Include tutto; non ha layout DVR dedicato o legenda specifica dei rischi. |
| Export misurazioni | Parziale | Include measure + dvr e tabella misure. Puo includere DVR non richiesto per misure pure. |
| Export completo | Completa/parziale | Include tutto e tabelle/pagine aggiuntive. Qualita dipende da densita e legenda. |
| PNG | Parziale | Export PNG del piano attivo tramite stesso renderer. Non include cartiglio PDF. |
| Salvataggio JSON | Completa/parziale | JSON completo con foto inclusa; versione 4.1. Rischio dimensione per foto grandi. |
| Riapertura JSON | Completa/parziale | Normalizzazione V4/floors e legacy, migrazione marker scala/ascensore. Validazione schema limitata. |
| IndexedDB | Completa/parziale | Store `projects`, chiave `latest`, autosave completo. Non ha gestione multi-progetto. |
| Fallback localStorage | Parziale | Salva senza foto se IndexedDB fallisce. Avvisa l'utente. |
| Compatibilita tablet | Parziale | UI touch-first, pointer events, pinch zoom, bottom sheets. Pulsanti e palette possono essere affollati; precisione richiesta alta. |
| Offline | Parziale | PWA e service worker. Offline PDF dipende dal primo caricamento/cache di jsPDF CDN. |

## 4. Problemi critici

1. Branding/export non standalone: `drawPdfHeader` e `drawPdfFooter` usano `Sicuro 360 - Planimetrie`. Questo contraddice il requisito di prodotto standalone e crea accoppiamento concettuale prima di qualsiasi integrazione futura.

2. Monolite applicativo: `index.html` contiene struttura, stile, stato, dominio, storage, rendering, interazioni ed export. Ogni modifica rischia regressioni trasversali.

3. Export PDF dipendente da CDN: se jsPDF non e gia in cache, offline o rete bloccata impediscono la generazione PDF. Per un prodotto da sopralluogo e un rischio operativo alto.

4. Assenza di test automatizzati reali: ci sono checklist e report manuali, ma non test browser, export, storage, migrazioni o regressioni visive.

5. Modellazione grafica troppo libera: porte, finestre, vie di esodo e simboli non sono vincolati a muri/locali/uscite. Questo permette risultati incoerenti senza avviso.

## 5. Problemi importanti

- Il numero di emergenza default nelle istruzioni e 115; il caso d'uso richiede 112.
- Header e cartiglio non sono un vero cartiglio professionale: non separano chiaramente titolo, indirizzo, piano, data, revisione e redattore come campi sempre visibili.
- I layer sono solo visibilita UI, non entita persistenti o livelli esportabili in modo configurabile.
- La legenda e generata da simboli usati, ma puo saturarsi e non gestisce pagine legenda aggiuntive.
- `areasVisible()` ritorna sempre true: gli export di emergenza includono comunque le aree, anche quando il filtro dichiara focus emergenza.
- Le quote sono globali e temporanee, non elementi con posizione, stile e persistenza dedicata.
- La calibrazione riscalando tutto intorno alla foto puo sorprendere l'utente se elementi sono gia stati disegnati con scala diversa.
- Il sistema di undo e snapshot JSON con foto: semplice ma potenzialmente pesante.
- Il service worker cachea anche richieste runtime, ma senza strategia di versioning degli asset esterni oltre alla cache name.
- Il demo JSON usa `ts` futuro rispetto alla data file/prodotto (`1781769600000`, circa giugno 2026), da verificare se intenzionale.

## 6. Problemi minori

- Alcuni testi usano ASCII approssimato o apostrofi incoerenti (`Unita'`, `l esodo`).
- Le icone dei pulsanti sono SVG inline, non una libreria iconografica centralizzata.
- Pannelli con molte proprieta usano campi numerici grezzi, poco adatti a utenti non CAD.
- `clearAutosave()` e definita ma non risulta esposta in UI.
- La ricerca palette e utile ma non ha preferiti o ultimi usati.
- `includeInstructions` e `includeDisclaimer` sono nel modello, ma non emergono come opzioni UI complete.
- Nessuna gestione esplicita di errore quota IndexedDB prima del fallback.
- Nessuna importazione PDF/DXF/DWG, correttamente assente rispetto allo scope attuale ma rilevante per velocita di rilievo.

## 7. Gap rispetto al caso d'uso

Caso d'uso: planimetria di evacuazione con pianta locali, nomi ambienti, estintori, pronto soccorso, uscite, percorsi, quadro elettrico, gas, coperta antifiamma, raccolta, "Voi siete qui", legenda, norme generali/incendio, 112, titolo, indirizzo, piano, data, revisione, redattore.

Cosa e gia possibile:

- Disegnare la pianta dei locali con muri/aree.
- Inserire nomi ambienti tramite aree o testi.
- Inserire estintori, primo soccorso, uscite, percorsi di fuga, quadro elettrico, gas, coperta antincendio, punto raccolta e `Sei qui`.
- Generare legenda basata sui simboli usati.
- Inserire istruzioni generali e incendio in pagina PDF.
- Inserire azienda, indirizzo, piano/revisione, redattore e qualifica.
- Esportare A3/A4 in PDF e PNG.

Cosa manca:

- Template "planimetria evacuazione standard" con checklist guidata.
- Campo titolo esplicito e data/revisione separata dalla data automatica.
- Numero unico 112 nel default.
- Cartiglio strutturato e professionale.
- Posizionamento controllato di legenda e istruzioni nella stessa tavola, quando richiesto.
- Verifica grafica di sovrapposizioni tra simboli, testi, legenda e percorsi.

Cosa richiede troppi passaggi:

- Tracciare una pianta da zero su tablet.
- Allineare porte/finestre ai muri.
- Compilare una planimetria completa con tutti i presidi cercandoli nella palette.
- Correggere vertici di aree e vie di esodo: si sposta l'intero elemento, non il singolo punto.
- Preparare cartiglio/intestazione conforme allo standard interno del consulente.

Cosa rischia risultato graficamente debole:

- Simboli troppo vicini e label sovrapposte.
- Vie di esodo che attraversano muri o locali non pertinenti.
- Testi senza wrapping.
- Legenda troppo compressa.
- Foto base opaca o non ritagliata.
- Aree sempre visibili anche in export emergenza.

Modifiche che ridurrebbero maggiormente i tempi:

1. Wizard "Planimetria evacuazione" con checklist e inserimento rapido dei campi obbligatori.
2. Snap porte/finestre al muro piu vicino e snap percorsi a uscite/raccolta.
3. Preset simboli preferiti per emergenza: estintore, primo soccorso, uscita, raccolta, sei qui, quadro, gas, coperta, allarme.
4. Cartiglio professionale automatico con titolo, indirizzo, piano, data, revisione, redattore.
5. Maniglie vertice per vie di esodo e aree.

## 8. Analisi dell'usabilita

Per un consulente non AutoCAD, l'app e piu accessibile di un CAD: usa contesti, pulsanti grandi, bottom sheet e simboli predefiniti. Tuttavia resta un editor geometrico: richiede precisione, comprensione di modalita e gestione manuale di scala, griglia e snap.

Numero di passaggi:

- Buono per inserire marker singoli.
- Medio/alto per una planimetria completa.
- Alto per correzioni geometriche fini.

Chiarezza icone e linguaggio:

- Le label testuali aiutano.
- Alcune categorie sono tecniche ma appropriate per consulenti sicurezza.
- "Muri", "Porte", "Finestre", "Vie di esodo" sono chiari.
- "Completo", "DVR", "Misurazioni" sono coerenti, ma un utente inesperto puo non capire cosa cambia realmente nell'export.

Grandezza pulsanti:

- Topbar e toolbar sono utilizzabili su tablet.
- Palette simboli da 60/66px e leggibile, ma affollata su schermi piccoli.
- I mini button nei piani sono piccoli per touch.

Strumenti nascosti:

- Intestazione, istruzioni, nord, griglia, layer, calibrazione e salvataggio JSON sono nel menu, non nel flusso di export.
- Proprietari avanzati appaiono solo dopo selezione.

Annullamento/correzione:

- Undo/redo esiste e copre molte operazioni.
- Limite storico 12 snapshot: sufficiente per errori recenti, poco per sessioni lunghe.
- Mancano maniglie per editare vertici, quindi la correzione spesso richiede cancellare e ridisegnare.

Touch, zoom e trascinamento:

- Pointer events e pinch zoom presenti.
- `touchmove` globale prevent default evita scroll, utile per app fullscreen ma puo interferire con comportamenti browser/accessibilita.
- Precisione su tablet resta critica per linee, porte e percorsi.

Salvataggio e prevenzione perdita dati:

- Autosave frequente su IndexedDB.
- Fallback localStorage senza foto e avviso.
- Salvataggio JSON manuale completo.
- Non c'e gestione multi-progetto locale o schermata recupero versioni.

## 9. Analisi tablet e accessibilita

Tablet:

- Layout mobile-first, app fullscreen, bottom sheets.
- `maximum-scale=1.0, user-scalable=no` disabilita lo zoom browser: utile per canvas, ma peggiora accessibilita.
- I controlli orizzontali scrollabili possono essere difficili se l'utente non nota che ci sono strumenti fuori vista.
- Alcune azioni distruttive usano `confirm`, semplice ma poco raffinato su mobile.

Accessibilita base:

- Molti controlli sono `div` cliccabili, non `button`, quindi tastiera e screen reader sono deboli.
- Icon button hanno `title`, ma non `aria-label`.
- Canvas non ha alternativa testuale del contenuto.
- Contrasti generalmente buoni per UI scura, ma alcuni testi piccoli `10-11px` sono borderline.
- Non c'e focus management nei bottom sheet.
- Le scorciatoie tastiera sono utili desktop, ma non documentate in UI e non sostituiscono accessibilita semantica.

## 10. Analisi dell'export

PDF:

- Rendering del piano su canvas bianco, poi inserimento raster PNG nel PDF.
- A3/A4 con orientamento automatico in base al rapporto del piano.
- Header, legenda, scala e footer sono disegnati con jsPDF.
- Pagine aggiuntive per tabella misure e istruzioni.

Leggibilita:

- Buona per elaborati piccoli/medi.
- Per planimetrie grandi `pxPerMeterExport` si riduce fino a 35 px/m: testi e simboli possono diventare meno leggibili.
- Simboli hanno dimensione minima, ma label e testi possono sovrapporsi.

Proporzioni e margini:

- Margine fisso 12 mm.
- Header fisso 28 mm.
- Legenda stimata fino a 80 mm: se molti simboli sono presenti, la tavola si comprime.

Cartiglio:

- Presente solo come header/footer, non come cartiglio tecnico.
- Branding attuale non standalone.
- Data generata al momento export, non campo progetto controllato.

Legenda:

- Generata per simboli usati e elementi extra.
- Include codici ISO quando presenti nel catalogo.
- Non gestisce overflow con pagina legenda dedicata.

A3/A4 e orientamento:

- La scelta formato e presente.
- Orientamento e automatico, non forzabile.
- Per planimetrie molto allungate, l'immagine puo diventare troppo bassa/stretta.

PNG:

- Coerente con il renderer PDF del piano, ma non include header, cartiglio, legenda PDF o istruzioni.
- Utile come immagine tecnica, non come elaborato completo.

Coerenza canvas/PDF/PNG:

- Canvas UI ha tema scuro e griglia scura; export usa bianco e griglia chiara.
- Alcuni colori/testi cambiano per export, scelta corretta per stampa ma da testare visivamente.
- Aree e layer export non sono completamente allineati ai layer UI.

## 11. Analisi tecnica e normativa

Fonti consultate per perimetro, non per dichiarare conformita:

- Gazzetta Ufficiale, DM 2 settembre 2021, GU Serie Generale n.237 del 04-10-2021: criteri gestione luoghi di lavoro in esercizio ed emergenza e servizio antincendio.
- UNI store, UNI ISO 23601:2024: norma in vigore dal 03 maggio 2024; stabilisce principi di progettazione per planimetrie dell'emergenza da esporre con informazioni per sicurezza antincendio, esodo, evacuazione e salvataggio; non riguarda disegni tecnico-professionali dettagliati per specialisti.
- ISO, ISO 7010:2019: standard pubblicato, edizione 3, con emendamenti; prescrive segnali di sicurezza per prevenzione incidenti, antincendio, salute ed evacuazione; forma e colore secondo ISO 3864-1 e simboli secondo ISO 3864-3.
- D.Lgs. 81/2008: riferimento di contesto per DVR, segnaletica, emergenza e responsabilita. Normattiva non ha restituito testo leggibile via tool in questa sessione, quindi non estraggo prescrizioni puntuali non verificate dal codice.

Controlli automatizzabili dal software:

- Presenza di intestazione minima: azienda, indirizzo, piano, revisione, data, redattore.
- Presenza di uscita, via di esodo, punto raccolta, "sei qui" e istruzioni.
- Presenza di almeno un presidio quando si esporta una planimetria di emergenza.
- Presenza di legenda coerente con simboli usati.
- Evidenza "scala indicativa" quando non calibrata.
- Avviso se foto base non calibrata.
- Avviso su codici misura duplicati.
- Avviso su marker senza label/codice, testi troppo lunghi, sovrapposizioni probabili.
- Verifica che i percorsi di esodo abbiano almeno due punti e terminino vicino a uscita/raccolta.
- Verifica che porte/finestre siano vicine a muri.
- Verifica che i simboli usino categorie colore coerenti con catalogo.

Informazioni da inserire dal consulente:

- Dati azienda/sito, indirizzo, piano, revisione, data, redattore.
- Nomi locali e destinazioni d'uso.
- Posizione reale di presidi, uscite, interruttori, intercettazioni, raccolta.
- Istruzioni specifiche del luogo.
- Valori e dettagli dei punti di misura.
- Classi REI/EI e classificazioni tecniche.
- Eventuali note DVR e rischi.

Valutazioni che restano responsabilita professionale:

- Adeguatezza delle vie di esodo.
- Correttezza dei presidi rispetto al rischio.
- Validita di classi REI/EI e compartimentazioni.
- Coerenza con piano di emergenza aziendale e DVR.
- Conformita delle planimetrie esposte ai requisiti applicabili.
- Correttezza e aggiornamento della segnaletica ISO 7010 realmente installata.
- Idoneita della scala, misure e rilievo.

Prescrizioni non dichiarate:

- Il codice non puo dimostrare conformita a D.Lgs. 81/2008, DM 2 settembre 2021, UNI ISO 23601:2024 o ISO 7010.
- I pittogrammi sono proprietari "in stile" e associati a codici ISO nel catalogo: non basta per dichiarare segnaletica certificata.
- Senza testo completo licenziato delle norme UNI/ISO non e corretto implementare o dichiarare tutte le prescrizioni di dettaglio.

## 12. Rischi architetturali

- Identificativi: marker, floor, scale, ascensori e testi hanno id; muri, REI, finestre, aree e routes spesso non hanno id stabile. Questo limita sync, diff, migrazioni e modifica fine.
- Schema JSON: `version: 4.1` esiste, ma non c'e schema formale, validatore o migrazioni versionate per ogni release.
- Migrazioni: `normalizeProjectData`, `projectFromLegacy`, `migrateMarker`, `migrateStructuralMarkers` sono utili ma accoppiate al runtime UI.
- Storage: IndexedDB/localStorage sono dentro `index.html`, non separati da UI e modello.
- Export: PDF/PNG dipendono da stato globale `exportType`, `showQuotes`, DOM e funzioni canvas; riproducibilita non garantita se cambiano font/browser/DPR.
- Sincronizzazione futura: possibile solo dopo stabilizzazione id, schema, storage adapter e separazione domain/export.
- Accoppiamento Sicuro360: branding nel PDF e changelog crea vincolo nominale non necessario.
- Service worker: caching di CDN e runtime request puo rendere diagnosi offline difficile.

## 13. Interventi raccomandati ordinati per impatto

1. Rimuovere branding Sicuro360 dall'export e centralizzare brand/prodotto in configurazione standalone.
2. Rendere jsPDF locale o fornire fallback offline controllato, cosi il PDF funziona dopo installazione senza CDN.
3. Separare almeno in moduli logici: modello/schema, storage, rendering canvas, interazioni, export, UI sheets.
4. Definire `PROJECT_SCHEMA_VERSION` e validatore/migrazioni versionate.
5. Aggiungere id stabili a tutte le entita: walls, reiWalls, windows, doors, areas, routes e route points se necessario.
6. Creare wizard/checklist per planimetria di evacuazione con 112, campi cartiglio e simboli richiesti.
7. Implementare snap semantico: porte/finestre su muri, route verso uscite/raccolta, marker su griglia/locali.
8. Aggiungere edit vertici per aree e vie di esodo.
9. Migliorare export: cartiglio vero, legenda paginabile, controlli overflow/sovrapposizione, orientamento forzabile.
10. Introdurre test automatici browser per salvataggio/apertura, export e funzioni core.

## 14. Roadmap a checkpoint

Checkpoint 1 - Stabilizzazione standalone:

- Rimuovere riferimenti Sicuro360 da PDF/UI/documentazione dove non voluti.
- Rendere jsPDF locale.
- Aggiungere smoke test statici e checklist export manuale aggiornata.
- Correggere default 112 nelle istruzioni, se confermato dal prodotto.

Checkpoint 2 - Schema e storage:

- Estrarre schema JSON e migrazioni.
- Aggiungere id a tutte le entita.
- Separare storage adapter da UI.
- Aggiungere test su demo JSON, legacy e autosave.

Checkpoint 3 - Velocita sopralluogo:

- Wizard planimetria evacuazione.
- Preset simboli rapidi.
- Snap porte/finestre/percorsi.
- Maniglie vertici.

Checkpoint 4 - Export professionale:

- Cartiglio completo.
- Legenda paginabile.
- Controlli sovrapposizione.
- Impostazioni export persistenti.
- Test PDF A3/A4 con casi grandi.

Checkpoint 5 - Accessibilita e tablet:

- Sostituire div cliccabili principali con button semantici.
- Aria label, focus trap nei bottom sheet, target touch coerenti.
- Revisione contrasto e font piccoli.
- Test tablet reale.

## 15. Test da eseguire

Test gia eseguiti in questo audit:

- `pwd`: OK.
- `ls -la`: OK.
- `git status`: fallito per assenza repo Git.
- `git branch --show-current`: fallito per assenza repo Git.
- `rg --files`: OK.
- Validazione JSON di `manifest.webmanifest` e `demo_officina_v4_1.json`: OK.
- Parsing sintattico dello script inline estratto da `index.html`: OK.

Test manuali/browser raccomandati:

- Aprire app da server locale e verificare assenza errori console.
- Creare da zero un piano con muri, porte, finestre, aree, presidi, vie, testi.
- Caricare foto, calibrare, esportare PDF/PNG.
- Salvare JSON, ricaricare pagina, ripristinare autosave, aprire JSON.
- Testare fallback localStorage simulando IndexedDB non disponibile.
- Esportare A3/A4 in tutti i tipi: emergenza, DVR, misurazioni, completo.
- Testare multipiano con scale collegate.
- Testare offline dopo primo caricamento e dopo cache vuota.
- Testare tablet: iPad/Android, orientamento verticale/orizzontale, pinch, drag, bottom sheet.
- Testare casi grandi: planimetria larga, molti simboli, legenda lunga, istruzioni lunghe.
- Testare accessibilita base: tastiera, screen reader, focus, contrasto.

Parti non verificate operativamente:

- Rendering reale PDF in browser con jsPDF.
- Download effettivo PDF/PNG.
- Registrazione service worker in browser.
- IndexedDB reale e fallback localStorage in browser.
- Interazione touch su tablet fisico.
- Qualita stampa su stampante/PDF viewer.

## 16. File che probabilmente richiederanno modifiche nelle fasi successive

- `index.html`: oggi contiene quasi tutta l'app. Richiedera modifiche per branding, schema, UI, rendering, export, accessibilita e testabilita.
- `sw.js`: per strategia cache, eventuale jsPDF locale e versionamento asset.
- `manifest.webmanifest`: se cambiano nome, scope, icone o installabilita.
- `_headers`: se servono policy cache/CSP piu controllate per CDN o asset locali.
- `README.md`, `MANUALE_RAPIDO.md`, `COLLAUDO_MANUALE.md`, `TEST_REPORT.md`: da aggiornare con flussi reali, limiti e nuova roadmap.
- `demo_officina_v4_1.json`: da riallineare allo schema futuro e usare come fixture di test.

## Riepilogo finale

Percorso workspace: `/Users/francescobellomo/Desktop/planimetrie_rapide_v4`

Branch: non disponibile, cartella non Git.

Stato Git: non disponibile, `git status` fallisce con `fatal: not a git repository (or any of the parent directories): .git`.

File analizzati:

- `index.html`
- `sw.js`
- `manifest.webmanifest`
- `demo_officina_v4_1.json`
- `README.md`
- `PRODUCT_AUDIT.md`
- `MANUALE_RAPIDO.md`
- `COLLAUDO_MANUALE.md`
- `TEST_REPORT.md`
- `CHECKPOINT_V4_1.md`
- `CHANGELOG_V4.md`
- `CHANGELOG_V4_1.md`
- `INSTALLAZIONE.md`
- `_headers`
- `icon.svg` come asset rilevato, non ispezionato semanticamente oltre alla presenza.

File creato:

- `TECHNICAL_AUDIT.md`

Principali cinque criticita:

1. Branding Sicuro360 nel PDF non coerente con standalone.
2. Monolite `index.html` con alto rischio regressione.
3. PDF offline dipendente da jsPDF CDN/cache.
4. Assenza test automatici browser/export/storage.
5. Geometria non vincolata: snap e controlli insufficienti per elaborati robusti.

Principali cinque opportunita:

1. Wizard evacuazione con checklist e simboli rapidi.
2. Cartiglio professionale automatico.
3. Snap porte/finestre/percorsi per ridurre errori e tempi.
4. Schema JSON versionato con id stabili e migrazioni.
5. Export piu controllato con legenda paginabile e controlli grafici.

Parti non verificate:

- Esecuzione completa in browser reale.
- Export PDF/PNG effettivo.
- Offline/PWA reale.
- Storage IndexedDB/localStorage reale.
- Touch/tablet fisico.
- Conformita normativa di dettaglio, che resta da valutare con fonti complete e responsabilita professionale.
