# Fase 2 - Piano implementativo export PEE professionale

Data analisi: 2026-06-23  
Workspace: `/Users/francescobellomo/Desktop/planimetrie_rapide_v4`  
Branch: `main`  
Ultimo commit rilevato: `b8581c1 Checkpoint 1 stabilizzazione standalone e PDF offline`

## 1. Sintesi

La Fase 1 risulta conclusa e validata: l'app e standalone, usa `vendor/jspdf.umd.min.js`, funziona offline dopo il primo caricamento, non effettua richieste verso `unpkg.com`, usa il branding "Planimetrie Rapide", usa il numero 112 nei contenuti PEE e rispetta `state.includeInstructions !== false`.

La Fase 2 deve migliorare l'export di emergenza rendendolo piu professionale, leggibile e adatto all'esposizione, senza refactor strutturali. Il codice attuale consente l'intervento in modo incrementale perche l'export PDF e gia concentrato in funzioni specifiche dentro `index.html`: `buildPDF`, `drawPdfHeader`, `drawPdfFooter`, `drawLegendPDF`, `estimateLegendH`, `drawInstructionsPage`, `renderFloorToCanvas` e `preflightCheck`.

La priorita tecnica non e cambiare il modello dati, ma rendere piu robusto il layout PDF:

- cartiglio professionale e retrocompatibile;
- legenda filtrata, ordinata e paginabile;
- istruzioni comportamentali impaginate in modo leggibile, con box 112;
- gestione coerente di "Voi siete qui";
- controlli pre-export non bloccanti;
- criteri minimi di leggibilita e stampa.

Non sono richiesti nuovi backend, login, sincronizzazioni, wizard, snap o maniglie dei vertici.

## 2. Inventario degli elementi gia presenti

Inventario verificato nel codice, in particolare nel catalogo `SYMBOLS`, nelle funzioni di disegno ed export e nella validazione pre-export.

| Elemento richiesto | Stato reale | Evidenza codice | Nota operativa |
| --- | --- | --- | --- |
| Pianta/locali | Implementato | muri, porte, finestre, aree, testi | I nomi locali si ottengono con aree/testi; non esiste un workflow guidato dedicato. |
| Vie di esodo | Implementato | `floor.routes`, `drawRoutePath`, `routesVisible` | Sono linee verdi con frecce, filtrate per PEE/completo. |
| Uscite | Implementato | `uscita`, `uscita_emerg` in `SYMBOLS` | Entrambe presenti; la legenda deve evitare duplicati impropri. |
| Estintori | Implementato | `estintore`, `estintore_carrellato` | Presente anche variante carrellata. |
| Primo soccorso/cassetta | Presente con nome diverso | `primo_socc`, `pacchetto_med` | Non creare duplicati; usare alias/label export "Cassetta primo soccorso" se necessario. |
| Quadro elettrico generale | Presente con nome diverso | `quadro` | Label attuale "Quadro elett."; puo diventare label export "Quadro elettrico generale". |
| Intercettazione gas | Presente con nome diverso | `gas` | Label attuale "Valvola gas"; coerente con intercettazione gas. |
| Coperta antifiamma | Presente con nome abbreviato | `coperta_antincendio` | Label attuale "Coperta antinc."; usare label completa in legenda/export. |
| Punto di raccolta | Implementato | `raccolta` | Gia controllato da `preflightCheck`. |
| "Voi siete qui" | Presente con nome diverso | `sei_qui`, glyph `youarehere` | Label attuale "Sei qui"; per PEE usare "Voi siete qui" senza cambiare `symId`. |
| Legenda | Implementata ma incompleta | `estimateLegendH`, `drawLegendPDF` | Filtra i marker esportati ma non pagina e puo comprimere troppo la planimetria. |
| Norme comportamentali | Implementate ma semplici | `state.instructions`, `drawInstructionsPage` | Pagina testuale monocolonna; manca layout professionale a sezioni e box 112. |
| Numero 112 | Implementato | `DEFAULT_INSTRUCTIONS` | Presente nelle istruzioni default dopo Fase 1. |
| Azienda | Implementato | `state.header.azienda` | Campo gia disponibile. |
| Indirizzo | Implementato | `state.header.indirizzo` | Campo gia disponibile. |
| Titolo elaborato | Incompleto | derivato da export type/meta | Non esiste campo esplicito; si puo derivare "Planimetria di emergenza". |
| Piano/area | Implementato | `state.header.piano`, `floor.name` | Disponibile. |
| Data | Parziale | data export in header PDF | Non esiste campo modificabile dedicato; default retrocompatibile: data generazione PDF. |
| Revisione | Implementato | `state.header.revisione` | Campo gia disponibile. |
| Redattore | Implementato | `state.header.redattore` | Campo gia disponibile. |
| Qualifica | Implementato | `state.header.qualifica` | Campo gia disponibile. |

Conclusione inventario: gli elementi PEE principali sono gia presenti. La Fase 2 non deve introdurre duplicati di simboli, ma normalizzare label, legenda, controlli e impaginazione.

## 3. Gap reali

I gap principali non riguardano la disponibilita degli elementi, ma la qualita dell'elaborato esportato.

1. Cartiglio non professionale: l'header attuale contiene alcune informazioni, ma non e un vero cartiglio tecnico con gerarchia, formato, scala, revisione, redattore e qualifica organizzati in tabella.

2. Legenda non paginabile: `drawLegendPDF` usa due colonne fisse e non gestisce bene overflow, molte voci o planimetrie che richiedono spazio.

3. Legenda non completamente governata da criteri misurabili: usa simboli effettivamente presenti, ma gli extra vengono aggiunti con logica separata e non esiste un modello unico di item esportati.

4. Istruzioni comportamentali poco adatte all'esposizione: `drawInstructionsPage` produce testo monocolonna, senza box 112, senza colonne "Norme generali" / "In caso di incendio" e senza layout di continuita controllato.

5. "Voi siete qui" esiste come `sei_qui`: funziona, ma la label non coincide con il caso d'uso richiesto e non c'e gestione specifica di marcatori multipli.

6. Controlli pre-export presenti ma incompleti: mancano controlli su cartiglio completo, legenda vuota, simboli fuori area esportabile, elementi troppo piccoli, overlap probabili e route con meno di due punti.

7. Qualita di stampa non formalizzata: margini, dimensioni minime, spessori e soglie di leggibilita non sono espressi come regole tecniche verificabili.

## 4. Proposta cartiglio

### Obiettivo

Sostituire l'header/footer semplice dell'export PEE con un cartiglio professionale, mantenendo compatibilita con `state.header` esistente e senza cambiare la struttura JSON.

### Campi

Campi obbligatori per controllo pre-export:

- titolo elaborato;
- ragione sociale;
- indirizzo sede;
- piano o area;
- data elaborato;
- revisione;
- redattore.

Campi facoltativi:

- qualifica;
- scala;
- formato;
- note/disclaimer.

Compatibilita dati:

- `ragione sociale`: `state.header.azienda`, fallback `state.meta.name`;
- `indirizzo`: `state.header.indirizzo`;
- `piano/area`: `state.header.piano`, fallback `floor.name`;
- `revisione`: `state.header.revisione`, fallback `Rev. 00`;
- `redattore`: `state.header.redattore`;
- `qualifica`: `state.header.qualifica`;
- `titolo`: default derivato dall'export, per PEE "Planimetria di emergenza";
- `data`: default data di generazione PDF;
- `formato`: `A3`/`A4` piu orientamento;
- `scala`: se calibrata e calcolabile, valore indicativo; altrimenti dicitura "Scala indicativa".

Se in una fase successiva servisse un campo dedicato, deve essere opzionale e retrocompatibile, per esempio `state.header.titolo` o `state.header.data`, senza migrazione obbligatoria.

### Layout proposto

Per evitare regressioni, il layout deve essere implementato come helper PDF, non come modifica del canvas engine.

Pagina principale:

- margine esterno minimo: 10 mm;
- fascia titolo superiore: 16-18 mm;
- area planimetria centrale: massimizzata;
- cartiglio inferiore: 28-34 mm;
- legenda su lato o pagina separata in base alle soglie della sezione 5.

Gerarchia testi:

- titolo elaborato: 12-14 pt su A4, 14-16 pt su A3;
- azienda/sede: 9-10 pt;
- campi tecnici cartiglio: 7-8 pt;
- footer/disclaimer: 6.5-7 pt.

Comportamento A3/A4 e orientamento:

- A3 landscape: cartiglio inferiore 28 mm, legenda preferibilmente in sidebar destra se non riduce troppo la planimetria;
- A3 portrait: cartiglio inferiore 30 mm, legenda in basso solo se breve, altrimenti pagina legenda;
- A4 landscape: cartiglio inferiore 30 mm, legenda in sidebar solo se poche voci;
- A4 portrait: cartiglio inferiore 34 mm, legenda quasi sempre in pagina separata se supera 6 voci.

Gestione campi lunghi:

- massimo 2 righe per azienda e indirizzo nel cartiglio;
- riduzione font fino a 7 pt;
- ellissi solo nei campi del cartiglio se il testo resta eccessivo;
- nessun taglio nei contenuti normativi o nelle istruzioni: quelli devono proseguire su pagina successiva.

Funzioni previste:

- `resolveCartoucheData(floor, exportType, fmt, orientation)`;
- `drawProfessionalCartouche(doc, data, box)`;
- `fitPdfText(doc, text, box, options)`;
- aggiornamento mirato di `drawPdfHeader`, `drawPdfFooter` e `buildPDF`.

## 5. Proposta legenda

### Problema attuale

La legenda e generata da `legendMarkers` piu alcuni extra derivati dalla planimetria. Usa due colonne fisse, non pagina, puo comprimere la planimetria e non espone criteri chiari per decidere quando spostarla.

### Regole funzionali

La legenda deve:

- mostrare solo simboli effettivamente esportati;
- eliminare duplicati per `symId` e per extra equivalenti;
- rispettare `exportType`, layer e filtri gia esistenti;
- includere vie di esodo solo se `routesVisible()` e almeno una route esportata;
- includere REI/EI solo se `reiVisible()` e almeno un muro REI/EI esportato;
- includere porte, finestre, scale e ascensori solo se effettivamente visibili nell'export;
- ordinare per categoria: esodo, antincendio, primo soccorso, impianti/intercettazioni, strutture, misure/DVR, testi;
- usare icona e descrizione leggibili;
- non comprimere la planimetria sotto soglie minime;
- usare pagina aggiuntiva se lo spazio non basta.

### Soglie tecniche proposte

Parametri minimi:

- altezza riga legenda: 6.5 mm;
- icona legenda: minimo 5 mm;
- font legenda: minimo 8 pt;
- larghezza minima colonna: 55 mm;
- spazio minimo tra colonne: 6 mm;
- quota massima occupabile dalla legenda nella pagina principale: 28% dell'area utile;
- quota minima riservata alla planimetria: 60% dell'area utile.

Decisione layout:

1. Legenda nella pagina principale se:
   - A4 portrait: massimo 6 voci;
   - A4 landscape: massimo 8 voci;
   - A3 portrait: massimo 8 voci;
   - A3 landscape: massimo 10 voci;
   - e la planimetria mantiene almeno il 60% dell'area utile.

2. Legenda su seconda pagina se:
   - le soglie sopra sono superate;
   - oppure la legenda occuperebbe piu del 28% dell'area utile;
   - oppure la planimetria scenderebbe sotto il 60% dell'area utile;
   - oppure le voci non entrano con font minimo 8 pt.

3. Legenda su piu colonne se:
   - sono presenti piu di 8 voci;
   - la larghezza disponibile consente colonne da almeno 55 mm;
   - A4: massimo 2 colonne;
   - A3: massimo 3 colonne in pagina legenda;
   - nessuna colonna deve superare l'altezza utile.

4. Legenda multipagina se:
   - il numero di voci supera la capacita della pagina legenda;
   - oppure l'altezza residua non consente font e icone minimi.

Funzioni previste:

- `collectLegendItems(floor, renderResult, exportType)`;
- `measureLegendLayout(doc, items, availableBox, format)`;
- `drawLegendPDF(doc, items, box, options)`;
- `drawLegendPage(doc, items, context)`;
- sostituzione progressiva di `estimateLegendH` con una misura basata su item reali.

## 6. Proposta norme comportamentali

### Stato attuale

Le istruzioni sono salvate come stringa unica in `state.instructions`; il default contiene il 112. La pagina viene disegnata da `drawInstructionsPage` con testo monocolonna. La sezione viene esclusa correttamente se `state.includeInstructions === false`.

### Layout proposto

Per PEE/completo, se le istruzioni sono abilitate:

- titolo pagina: "Norme comportamentali";
- box evidente: "Numero unico di emergenza 112";
- colonna sinistra: "Norme generali";
- colonna destra: "In caso di incendio";
- sezione inferiore opzionale: "Istruzioni personalizzate";
- footer con progetto, revisione e pagina.

### Modello dati

Non introdurre nuovi campi obbligatori. Usare ancora `state.instructions`.

Comportamento retrocompatibile:

- se il testo contiene intestazioni riconoscibili, dividerlo in sezioni;
- se non contiene intestazioni riconoscibili, trattarlo come istruzioni personalizzate;
- se il consulente ha modificato il testo, mantenerlo integralmente;
- se `state.includeInstructions === false`, non generare alcuna pagina istruzioni.

Contenuti:

- testo predefinito: deriva da `DEFAULT_INSTRUCTIONS`;
- testo personalizzato: deriva da `state.instructions` quando diverso dal default o non parsabile;
- numero 112: esposto come box separato anche se presente nel testo, evitando duplicazioni visive e mantenendo il testo originale.

Gestione testi lunghi:

- font minimo corpo: 8 pt;
- interlinea minima: 4.2 mm;
- nessun taglio;
- se una colonna eccede l'altezza utile, continuare su pagina successiva;
- se il testo personalizzato e molto lungo, impaginarlo dopo le colonne standard;
- segnalare in pre-export quando le istruzioni superano due pagine, senza bloccare.

Funzioni previste:

- `parseInstructionSections(text)`;
- `drawInstruction112Box(doc, box)`;
- `drawInstructionColumns(doc, sections, box)`;
- aggiornamento mirato di `drawInstructionsPage`.

## 7. Proposta "Voi siete qui"

### Stato attuale

L'elemento esiste come `symId: "sei_qui"` con label "Sei qui" e glyph `youarehere`. Viene disegnato come marker e puo apparire nella legenda.

### Proposta retrocompatibile

Non introdurre un nuovo simbolo e non cambiare `symId`.

Regole:

- mantenere `sei_qui` come identificativo stabile;
- usare label export "Voi siete qui" per PEE e PDF professionale;
- mantenere label UI esistente o aggiornarla solo come testo, senza duplicare il simbolo;
- includere il simbolo in legenda se presente nell'elaborato esportato;
- preflight: avviso se assente, avviso se presente piu di una volta;
- export consentito comunque.

Aspetto grafico:

- marker ben contrastato;
- dimensione minima in stampa: 6 mm, preferibile 7 mm;
- label leggibile solo se non sovrapposta; in alternativa label nella legenda;
- rotazione: mantenere orientamento stabile rispetto alla pagina, non dipendente dalla rotazione della pianta;
- se in futuro serve indicare direzione dello sguardo, aggiungere campo opzionale di rotazione solo in modo retrocompatibile.

Marcatori multipli:

- comportamento attuale: esportare tutti i marker presenti;
- preflight: "Sono presenti piu marcatori Voi siete qui. Verificare che l'elaborato sia destinato a un solo punto di esposizione.";
- futura evoluzione semplice: generare piu PDF selezionando un marker attivo, senza introdurre ora varianti di progetto.

## 8. Controlli pre-export

I controlli devono essere non bloccanti. Il consulente deve poter esportare comunque. Il software supporta il professionista, ma non certifica automaticamente la conformita dell'elaborato.

Livelli:

- informazione: dato utile o scelta da verificare;
- avviso: potenziale problema di qualita o completezza;
- criticita: elemento essenziale mancante o forte rischio di elaborato non utilizzabile.

| Controllo | Condizione | Livello | Testo mostrato | Export |
| --- | --- | --- | --- | --- |
| Titolo mancante | Titolo derivato vuoto o non risolto | Avviso | "Titolo elaborato non compilato: verra usato un titolo automatico." | Consentito |
| Azienda mancante | `state.header.azienda` vuoto e nessun fallback | Criticita | "Ragione sociale mancante nel cartiglio." | Consentito |
| Indirizzo mancante | `state.header.indirizzo` vuoto | Avviso | "Indirizzo sede mancante nel cartiglio." | Consentito |
| Piano/area mancante | `state.header.piano` e `floor.name` vuoti | Avviso | "Piano o area non indicati." | Consentito |
| Revisione mancante | `state.header.revisione` vuoto | Informazione | "Revisione non indicata: verra usato il default Rev. 00." | Consentito |
| Redattore mancante | `state.header.redattore` vuoto | Criticita | "Redattore mancante nel cartiglio." | Consentito |
| Qualifica mancante | `state.header.qualifica` vuoto | Avviso | "Qualifica del redattore non indicata." | Consentito |
| Marcatore assente | nessun `sei_qui` esportato | Avviso | "Manca il marcatore Voi siete qui." | Consentito |
| Marcatori multipli | piu di un `sei_qui` esportato | Avviso | "Sono presenti piu marcatori Voi siete qui." | Consentito |
| Uscita assente | nessun `uscita`/`uscita_emerg` esportato | Criticita | "Manca almeno una uscita." | Consentito |
| Via di esodo assente | nessuna route esportata | Criticita | "Manca almeno una via di esodo." | Consentito |
| Percorso troppo corto | route con meno di due punti validi | Criticita | "Una via di esodo ha meno di due punti." | Consentito |
| Punto raccolta assente | nessun `raccolta` esportato, se richiesto dal tipo PEE | Avviso | "Punto di raccolta non presente." | Consentito |
| Legenda vuota | nessun item legenda dopo filtri | Avviso | "La legenda risulta vuota." | Consentito |
| Sovrapposizioni probabili | bounding box marker/testi si intersecano oltre soglia | Avviso | "Alcuni simboli o testi potrebbero sovrapporsi in stampa." | Consentito |
| Simboli fuori area esportabile | marker/testi fuori bounding box calcolato o tagliati dal render | Criticita | "Alcuni elementi potrebbero non comparire nell'area esportata." | Consentito |
| Planimetria non calibrata | `state.calibration` assente o non valida | Informazione | "Planimetria non calibrata: la scala sara indicativa." | Consentito |
| Elementi troppo piccoli | simbolo sotto 5 mm o testo sotto 7 pt nel layout calcolato | Avviso | "Alcuni elementi potrebbero essere troppo piccoli per la stampa." | Consentito |
| Istruzioni disattivate | `includeInstructions === false` | Informazione | "Le norme comportamentali sono disattivate per questo export." | Consentito |
| Istruzioni molto lunghe | stima oltre due pagine | Informazione | "Le norme comportamentali occuperanno piu pagine." | Consentito |

Funzioni previste:

- estensione di `preflightCheck(kind)`;
- eventuali helper `collectExportedElements`, `estimatePrintSizes`, `detectLikelyOverlaps`;
- aggiornamento di `showPreflight` solo per testi/livelli, senza bloccare il flusso.

## 9. Standard grafici e di stampa

Questa sezione distingue requisiti normativi verificati, buone pratiche grafiche e decisioni interne di prodotto. Non viene dichiarata conformita automatica.

### Requisiti normativi verificati a livello di contesto

Dal Product Spec e dall'audit risultano rilevanti:

- D.Lgs. 81/2008 come contesto generale di sicurezza sul lavoro;
- D.M. 2 settembre 2021 come riferimento per gestione emergenze antincendio;
- UNI ISO 23601:2024 per planimetrie di emergenza;
- ISO 7010 per segnaletica di sicurezza applicabile.

Il codice non contiene una matrice normativa completa e non puo verificare automaticamente la conformita del documento. La Fase 2 deve quindi implementare controlli di completezza e leggibilita, non certificazioni.

### Buone pratiche grafiche

- margine pagina minimo: 10 mm;
- distanza minima tra contenuti non correlati: 2 mm;
- titolo principale: almeno 12 pt su A4, 14 pt su A3;
- testi legenda/istruzioni: almeno 8 pt;
- testi cartiglio secondari: almeno 7 pt;
- icone legenda: almeno 5 mm;
- simboli in planimetria: almeno 5 mm, preferibile 6-7 mm;
- linee vie di esodo: almeno 0.8 mm in stampa;
- linee muri principali: almeno 0.35-0.4 mm in stampa;
- contrasto testo/sfondo: preferibilmente almeno 4.5:1;
- evitare rosso/verde come unico codice informativo senza simbolo o label;
- non sovrapporre legenda, cartiglio e planimetria.

### Decisioni interne di prodotto

- A4 e A3 devono usare dimensioni reali jsPDF: A4 210 x 297 mm, A3 297 x 420 mm;
- orientamento scelto automaticamente in base alla planimetria, salvo selezione formato;
- legenda in pagina principale solo se non riduce la planimetria sotto il 60% dell'area utile;
- se planimetria molto larga, preferire landscape e legenda separata;
- se planimetria molto alta, preferire portrait e legenda separata;
- se molti simboli, usare legenda su pagina aggiuntiva;
- se testi lunghi, continuare su pagine successive invece di ridurre sotto font minimo;
- raster export: mantenere limite massimo esistente compatibile con prestazioni, ma garantire almeno una qualita adeguata per stampa quando possibile.

## 10. Retrocompatibilita

La Fase 2 deve mantenere:

- apertura progetti V4.1;
- autosave IndexedDB/localStorage;
- demo `demo_officina_v4_1.json`;
- export PDF emergenza, DVR, misurazioni e completo;
- export PNG;
- funzionamento offline;
- `includeInstructions`;
- vecchi dati privi di eventuali campi futuri.

Regole operative:

- non modificare la struttura globale di `state`;
- non introdurre `objects[]` unico;
- non cambiare il canvas engine;
- non modificare serializzazione JSON se non con campi opzionali in `state.header`, e solo se indispensabile;
- preferire valori derivati in fase di export;
- non aggiungere asset remoti;
- se si modifica `index.html`, aggiornare `sw.js` solo per cache busting della PWA;
- non cambiare il comportamento PNG salvo bug direttamente collegati al layout export.

## 11. Sotto-checkpoint 2A-2F

### 2A - Inventario e simboli

Obiettivo:

- completare la coerenza semantica degli elementi PEE senza duplicare simboli gia presenti.

Funzioni da modificare:

- `SYMBOLS` solo per label/alias se necessario;
- `symbolGroup`;
- `preflightCheck`;
- funzioni legenda che leggono label dei simboli.

File interessati:

- `index.html`;
- `sw.js` solo per aggiornamento cache se `index.html` cambia.

Rischi:

- duplicare simboli gia presenti;
- rompere progetti esistenti se si cambia `symId`;
- cambiare la UI piu del necessario.

Dipendenze:

- nessuna, ma e propedeutico alla legenda.

Criteri di accettazione:

- `estintore`, `primo_socc`, `quadro`, `gas`, `coperta_antincendio`, `sei_qui`, `raccolta`, `uscita`, `uscita_emerg` restano apribili nei vecchi progetti;
- nessun `symId` cambia;
- legenda/export usano label professionali;
- nessun duplicato nella toolbar.

Test richiesti:

- aprire demo V4.1;
- inserire ogni simbolo PEE;
- esportare PEE e verificare legenda;
- riaprire JSON salvato con simboli esistenti.

Conclusione checkpoint:

- tutti gli elementi PEE risultano disponibili con nome export coerente e retrocompatibile.

### 2B - Cartiglio

Obiettivo:

- introdurre layout cartiglio professionale nel PDF PEE, senza cambiare modello dati.

Funzioni da modificare:

- `buildPDF`;
- `drawPdfHeader`;
- `drawPdfFooter`;
- nuove helper `resolveCartoucheData`, `drawProfessionalCartouche`, `fitPdfText`.

File interessati:

- `index.html`;
- `sw.js` per cache busting.

Rischi:

- ridurre troppo la planimetria;
- tagliare campi lunghi;
- alterare export DVR/misure/completo.

Dipendenze:

- 2A utile per label coerenti, ma non bloccante.

Criteri di accettazione:

- PDF PEE mostra titolo, azienda, indirizzo, piano, data, revisione, redattore, qualifica, formato e scala indicativa;
- campi lunghi non escono dal cartiglio;
- A3/A4 portrait/landscape restano leggibili;
- export DVR/misure/completo non regressiscono.

Test richiesti:

- PEE A4 portrait;
- PEE A4 landscape;
- PEE A3 portrait;
- PEE A3 landscape;
- valori lunghi per azienda/indirizzo/redattore;
- campi vuoti con fallback.

Conclusione checkpoint:

- il cartiglio e stabile, leggibile e non richiede modifiche JSON.

### 2C - Legenda

Obiettivo:

- generare legenda automatica solo con elementi esportati, ordinata, senza duplicati e con overflow gestito.

Funzioni da modificare:

- `estimateLegendH`;
- `drawLegendPDF`;
- `buildPDF`;
- nuove helper `collectLegendItems`, `measureLegendLayout`, `drawLegendPage`.

File interessati:

- `index.html`;
- `sw.js` per cache busting.

Rischi:

- legenda non coerente con filtri layer;
- pagina aggiuntiva non numerata correttamente;
- compressione eccessiva della planimetria;
- regressioni su export completo.

Dipendenze:

- 2A per label;
- 2B per layout e spazi.

Criteri di accettazione:

- legenda mostra solo item esportati;
- nessun duplicato;
- vie di esodo, REI/EI, porte, finestre e scale compaiono solo se presenti;
- se lo spazio non basta viene creata pagina legenda;
- font e icone rispettano minimi definiti.

Test richiesti:

- PEE con pochi simboli;
- PEE con molti simboli;
- PEE senza marker;
- PEE con layer non pertinenti;
- completo con elementi PEE+DVR+misure;
- A4 portrait con legenda lunga;
- A3 landscape con legenda breve.

Conclusione checkpoint:

- la legenda non compromette la planimetria e resta leggibile.

### 2D - Norme e box 112

Obiettivo:

- trasformare le istruzioni in una pagina professionale con due colonne e box 112, mantenendo testo personalizzato.

Funzioni da modificare:

- `drawInstructionsPage`;
- nuove helper `parseInstructionSections`, `drawInstruction112Box`, `drawInstructionColumns`.

File interessati:

- `index.html`;
- `sw.js` per cache busting.

Rischi:

- perdere istruzioni personalizzate;
- duplicare il 112 in modo confuso;
- tagliare testi lunghi;
- non rispettare `includeInstructions === false`.

Dipendenze:

- 2B per stile cartiglio/pagine;

Criteri di accettazione:

- se istruzioni abilitate, PDF include norme generali, incendio e box 112;
- se istruzioni disattivate, nessuna pagina istruzioni;
- testo personalizzato viene mantenuto;
- testi lunghi continuano su pagine successive;
- nessuna sovrapposizione.

Test richiesti:

- default instructions;
- istruzioni custom brevi;
- istruzioni custom lunghe;
- `includeInstructions=false`;
- export PEE e completo.

Conclusione checkpoint:

- le norme sono leggibili, modificabili e compatibili con la Fase 1.

### 2E - Controlli pre-export

Obiettivo:

- aggiungere avvisi non bloccanti per completezza, leggibilita e rischi grafici.

Funzioni da modificare:

- `preflightCheck`;
- `showPreflight`;
- eventuali helper `collectExportedElements`, `detectLikelyOverlaps`, `estimatePrintSizes`.

File interessati:

- `index.html`;

Rischi:

- falsi positivi eccessivi;
- percezione di blocco da parte dell'utente;
- controlli troppo costosi su progetti grandi.

Dipendenze:

- 2B e 2C per conoscere layout e legenda;
- 2D per istruzioni.

Criteri di accettazione:

- tutti i controlli della sezione 8 sono presenti;
- nessun controllo blocca l'export;
- messaggi sono chiari e non dichiarano conformita;
- livelli informazione/avviso/criticita sono distinguibili.

Test richiesti:

- progetto completo;
- progetto senza uscita;
- progetto senza via di esodo;
- progetto senza `sei_qui`;
- progetto con due `sei_qui`;
- route con un solo punto;
- testi/simboli sovrapposti;
- planimetria non calibrata.

Conclusione checkpoint:

- il consulente riceve avvisi utili prima dell'export e puo procedere consapevolmente.

### 2F - Collaudo

Obiettivo:

- verificare regressioni e qualita dell'export su casi semplici e complessi.

Funzioni da verificare:

- export PEE;
- export completo;
- export DVR;
- export misurazioni;
- export PNG;
- salvataggio/apertura JSON;
- autosave;
- service worker/offline.

File interessati:

- nessuna modifica applicativa prevista nel checkpoint, salvo correzioni mirate di bug emersi;
- eventuale aggiornamento documentale di collaudo in una fase dedicata.

Rischi:

- regressioni non visibili senza browser reale;
- differenze tra PDF e canvas;
- service worker che serve cache vecchia;
- PDF grandi con immagini raster pesanti.

Dipendenze:

- 2A-2E completati.

Criteri di accettazione:

- PDF PEE leggibile in A3/A4 portrait/landscape;
- nessun riferimento Sicuro360;
- jsPDF locale;
- offline funzionante;
- demo V4.1 apribile;
- JSON vecchi apribili;
- PNG invariato o migliorato;
- console senza errori bloccanti.

Test richiesti:

- browser reale;
- export PDF su progetto minimo;
- export PDF su demo officina;
- export con molte voci legenda;
- export con istruzioni lunghe;
- offline dopo primo caricamento;
- ricarica e autosave.

Conclusione checkpoint:

- Fase 2 chiudibile solo con collaudo operativo PASS e nessuna regressione critica.

## 12. Criteri di accettazione complessivi

La Fase 2 puo essere considerata completata quando:

- il PDF PEE include cartiglio professionale completo;
- legenda e istruzioni restano leggibili in A3/A4 e portrait/landscape;
- il numero 112 e visibile in box dedicato quando le istruzioni sono abilitate;
- `includeInstructions=false` esclude davvero la sezione istruzioni;
- "Voi siete qui" e disponibile in export con label professionale;
- preflight segnala problemi senza bloccare;
- i progetti V4.1 e la demo si aprono;
- autosave e JSON restano invariati;
- PNG continua a funzionare;
- app continua a funzionare offline;
- non sono stati introdotti refactor strutturali del modello dati o del canvas engine.

## 13. Test

Test minimi manuali in browser reale:

1. Aprire app senza errori console bloccanti.
2. Aprire `demo_officina_v4_1.json`.
3. Creare progetto minimo PEE con muro, porta, uscita, estintore, via di esodo, punto raccolta, `sei_qui`.
4. Compilare cartiglio con dati brevi.
5. Esportare PEE A4 portrait.
6. Esportare PEE A4 landscape.
7. Esportare PEE A3 portrait.
8. Esportare PEE A3 landscape.
9. Verificare cartiglio, legenda, planimetria, norme, 112.
10. Disattivare istruzioni e riesportare.
11. Inserire molti simboli e verificare legenda su pagina aggiuntiva.
12. Inserire campi cartiglio lunghi e verificare wrapping.
13. Inserire istruzioni lunghe e verificare paginazione.
14. Verificare preflight su progetto incompleto.
15. Esportare PDF completo, DVR e misurazioni.
16. Esportare PNG.
17. Salvare JSON e riaprire.
18. Ricaricare e verificare autosave.
19. Spegnere server dopo primo caricamento e verificare offline.
20. Controllare che non partano richieste remote.

Test tecnici consigliati:

- ricerca testuale nel PDF per "Planimetrie Rapide", "112", assenza "Sicuro360";
- controllo dimensione PDF e presenza pagine aggiuntive;
- screenshot o rendering PDF per verificare sovrapposizioni;
- prova su viewport tablet;
- prova con service worker aggiornato e cache precedente.

## 14. Rischi

Rischi principali:

1. Compressione eccessiva della planimetria: mitigare con soglie legenda e pagina separata.
2. Perdita di retrocompatibilita: non cambiare `symId`, struttura JSON o serializzazione.
3. Regressioni su export non PEE: limitare nuove logiche al tipo PEE o renderle compatibili con completo/DVR/misure.
4. Testi tagliati: usare helper di wrapping e paginazione, mai font sotto soglia.
5. Service worker con cache vecchia: aggiornare `CACHE` quando cambia `index.html`.
6. Falsi positivi preflight: mantenere messaggi non bloccanti e chiari.
7. PDF pesanti: non aumentare indiscriminatamente la risoluzione raster.
8. Ambiguita normativa: non dichiarare conformita automatica, distinguere controlli software e responsabilita professionale.

## 15. Ordine raccomandato delle modifiche

1. 2A - Normalizzare label e inventario simboli senza cambiare identificativi.
2. 2B - Introdurre cartiglio e layout base PDF.
3. 2C - Rifare la logica legenda come raccolta item + misura + disegno/paginazione.
4. 2D - Migliorare pagina norme con colonne e box 112.
5. 2E - Estendere preflight con controlli non bloccanti.
6. 2F - Collaudo operativo completo, incluso offline e regressioni export.

Motivo dell'ordine: cartiglio e legenda dipendono dagli stessi spazi pagina; la legenda deve conoscere label e filtri; i controlli pre-export devono usare le regole definitive di layout.

## 16. File e funzioni probabilmente modificati

File applicativi:

- `index.html`
- `sw.js` solo per aggiornare il nome cache dopo modifiche a `index.html`

File da non modificare salvo necessita esplicita futura:

- `demo_officina_v4_1.json`
- progetti JSON utente
- `vendor/jspdf.umd.min.js`
- modello IndexedDB/localStorage

Funzioni o blocchi in `index.html` probabilmente interessati:

- `SYMBOLS` per label/alias non distruttivi;
- `symbolGroup`;
- `preflightCheck`;
- `showPreflight`;
- `renderFloorToCanvas` solo se serve restituire metriche/exported items, senza cambiare canvas engine;
- `buildPDF`;
- `drawPdfHeader`;
- `drawPdfFooter`;
- `estimateLegendH`;
- `drawLegendPDF`;
- `drawInstructionsPage`;
- `drawScaleBarMM`;
- `drawIndicativeScale`;
- `layerVisible`;
- `routesVisible`;
- `reiVisible`;
- eventuale nuova helper `resolveCartoucheData`;
- eventuale nuova helper `drawProfessionalCartouche`;
- eventuale nuova helper `fitPdfText`;
- eventuale nuova helper `collectLegendItems`;
- eventuale nuova helper `measureLegendLayout`;
- eventuale nuova helper `drawLegendPage`;
- eventuale nuova helper `parseInstructionSections`;
- eventuale nuova helper `drawInstruction112Box`;
- eventuale nuova helper `drawInstructionColumns`;
- eventuale nuova helper `collectExportedElements`;
- eventuale nuova helper `detectLikelyOverlaps`;
- eventuale nuova helper `estimatePrintSizes`.

Funzioni da non rifare:

- canvas engine interattivo;
- modello dati globale;
- storage IndexedDB/localStorage;
- serializzazione JSON, salvo campi opzionali retrocompatibili se approvati;
- gestione generale dei piani/layer.

